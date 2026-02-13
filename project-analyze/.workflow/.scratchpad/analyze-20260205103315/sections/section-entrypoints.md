# 系統入口與調用鏈

## HTTP 端點設計哲學

Mtr.StravaConnect系統採用最小化入口設計策略,通過單一的Web AP專案暴露HTTP服務,避免了多入口帶來的部署與維護複雜度。從 [Program.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.WebApi/Program.cs) 的啟動配置可以看出,系統採用了ASP.NET Core 8.0的Minimal Hosting Model,通過`WebApplication.CreateBuilder`快速構建應用實例,配合頂層語句(Top-Level Statements)實現了啟動代碼的極致簡化。

系統的入口設計遵循單一職責原則,WebApi層僅承擔HTTP協議適配職責。[StravaController.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.WebApi/Controllers/StravaController.cs) 的三個端點方法均採用薄控制器模式,方法體僅包含服務調用與響應封裝,不含任何業務邏輯。這種設計使得控制器成為純粹的協議轉換層,真正的業務邏輯被下沉至Service層,確保了表現層的穩定性與可測試性。

## 請求處理管道

HTTP請求在進入控制器前經過一系列精心設計的中介軟體管道處理。從 [Program.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.WebApi/Program.cs) 的管道配置順序可以看出系統的處理策略。首先是Swagger中介軟體(僅在開發環境啟用),提供API文檔的即時預覽能力。隨後是CORS中介軟體,執行跨域請求的來源驗證與標頭注入。接著是自定義的 [TraceIdMiddleware](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.MiddleWare/TraceIdMiddleware.cs),負責生成或傳遞分散式追蹤標識,並將其推送至日誌上下文。

TraceIdMiddleware的實現展現了對ASP.NET Core管道機制的深度運用。該中介軟體通過檢查請求標頭中的`X-Trace-Id`,若不存在則基於Guid生成唯一標識,隨後將其存儲至`HttpContext.Items`字典供後續處理使用,並通過`Response.OnStarting`回調確保TraceId在響應返回前被添加至響應標頭。通過`LogContext.PushProperty`,TraceId被推送至Serilog的日誌上下文,使得該請求生命週期內的所有日誌記錄都自動包含該標識。這種設計實現了全鏈路的請求追蹤能力,為分散式環境下的問題定位提供了關鍵支撐。

路由中介軟體通過`MapControllers`方法自動發現並註冊所有標註了`[ApiController]`與`[Route]`特性的控制器。ASP.NET Core的Routing系統根據請求的HTTP方法與URL路徑,精確匹配到對應的控制器方法並執行參數綁定。這種約定優於配置的設計使得開發者無需編寫繁瑣的路由映射代碼,僅通過聲明式的特性標註即可完成路由配置。

## 關鍵業務流程調用鏈

以訓練記錄同步流程為例,完整的調用鏈展現了系統各層級的協作機制。前端發起`POST /api/strava/training-records`請求,請求體包含用戶名稱、目標年份、目標代碼等參數。請求首先經過CORS中介軟體驗證來源合法性,隨後通過TraceIdMiddleware注入追蹤標識。路由中介軟體將請求分發至`StravaController.InsertTrainingRecords`方法,ASP.NET Core的模型綁定器自動將JSON請求體反序列化為`RequestInsertTrainingRecords`物件。

控制器方法調用`IStravaService.InsertTrainingRecordsAsync`,請求進入業務層。[StravaService.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.Service/StravaService.cs) 的實現首先通過`StravaTokenStore.AccessTokens.TryGetValue`嘗試獲取該用戶的有效Token,若不存在則拋出`UnauthorizedAccessException`。隨後通過`ITrainingRecordRepository.GetLatestStartDateAsync`查詢最後一筆訓練記錄的日期,轉換為Unix時間戳作為Strava API的查詢起始點。

服務層通過注入的`IStravaApiClient.GetActivitiesAsync`發起HTTP請求至Strava API,Refit框架自動構建GET請求並附帶Bearer Token。Strava返回的活動列表經過業務規則過濾(僅保留指定運動類型)與轉換(配速計算、跑步類型識別、週數計算),轉化為`TrainingRecordEntity`集合。最終通過`ITrainingRecordRepository.BatchInsertDuplicateKeyUpdateAsync`批次寫入資料庫,Repository層通過Dapper執行動態構建的SQL語句完成持久化。

整個調用鏈的執行過程中,所有日誌記錄都自動包含TraceId,使得開發團隊能夠通過日誌系統快速重建請求的完整執行軌跡。當任一環節發生異常,異常會沿調用鏈向上傳播,最終由ASP.NET Core的全域異常處理器捕獲並轉化為適當的HTTP錯誤響應。

## 背景服務入口

除了HTTP端點,系統還包含一個重要的背景服務入口—— [StravaTokenBackgroundService](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.Service/Background/StravaTokenBackgroundService.cs)。該服務通過實現`IHostedService`介面註冊至ASP.NET Core的Hosting系統,在應用啟動時自動啟動並在應用關閉時優雅終止。背景服務的設計目的在於將Token刷新的週期性任務從請求處理流程中剝離,避免了首次請求因Token過期而需要等待刷新的性能損耗。

背景服務的執行策略採用了定時循環模式。`ExecuteAsync`方法在應用啟動時立即執行一次Token刷新,確保系統在對外提供服務前已具備有效的認證憑證。隨後通過`await Task.Delay(_refreshInterval, stoppingToken)`實現五小時的週期性等待,每當等待結束即觸發新一輪的Token刷新。這種策略確保了Token在過期前總是被提前更新,避免了業務請求因Token失效而失敗的情況。

值得注意的是,背景服務通過`IServiceProvider.CreateScope()`為每次Token刷新創建獨立的依賴注入作用域。這一設計源於ASP.NET Core的Scop生命週期約束:IStravaService被註冊為Scoped服務,意味著其僅能在請求作用域內被解析。背景服務作為Singleton生命週期的長期運行任務,若直接注入Scoped服務會導致生命週期衝突。通過手動創建作用域並在使用後釋放,系統確保了每次Token刷新都使用獨立的服務實例,避免了狀態污染與資源泄漏問題。

## 依賴注入作用域管理

系統的入口設計中蘊含了對依賴注入作用域的精細化管理。HTTP請求處理流程中, ASP.NET Core為每個請求自動創建獨立的作用域,控制器與服務層的Scoped服務在該作用域內被實例化,請求結束後作用域被釋放,所有Scoped服務實例被銷毀。這種自動化的作用域管理確保了多請求間的隔離性,避免了狀態共享可能導致的併發問題。

Singleton服務如StravaTokenStore則在應用生命週期內全局唯一,所有請求與背景服務共享同一實例。這種設計策略使得Token快取能夠被高效復用,但同時也引入了線程安全的考量。系統通過背景服務的全量更新策略緩解了這一風險:Token刷新時先清空字典再重新填充,避免了細粒度的增刪改操作可能導致的競爭條件。雖然理論上讀寫併發仍可能發生,但Token刷新的低頻率(每五小時一次)與業務請求的高頻率特性使得實際衝突概率極低。

從整體視角審視,Mtr.StravaConnect的入口設計體現了對關注點分離原則的深刻理解。HTTP端點作為唯一的對外服務入口,通過精心設計的中介軟體管道實現橫切關注點的統一處理;背景服務作為內部任務入口,通過週期性執行機制實現運維自動化。各入口類型通過依賴注入容器實現協作,在保持鬆耦合的同時,實現了高效的資源共享與任務編排。這種入口設計策略為系統的可維護性、可擴展性以及運維友好度奠定了堅實基礎。
