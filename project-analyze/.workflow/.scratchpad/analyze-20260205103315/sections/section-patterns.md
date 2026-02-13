# 設計模式與工程規範

## 架構級模式應用

Mtr.StravaConnect系統在架構設計中廣泛運用了多種經典設計模式,這些模式的選擇均源於對特定技術挑戰的精準應對。Repository模式在系統中得到了完整實現,通過 [ITrainingRecordRepository](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.Repository/Interface/ITrainingRecordRepository.cs) 介面抽象了資料存取邏輯,使得業務層能夠以領域對象的方式操作數據,而無需關注底層SQL的構建細節。這種模式的價值在於將資料存取邏輯集中管理,避免了SQL語句在業務代碼中的散亂分佈,同時為單元測試提供了清晰的Mock邊界。

依賴注入(Dependency Injection)模式貫穿整個系統架構,從 [Program.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.WebApi/Program.cs) 的服務註冊可以看出,系統通過ASP.NET Core的內建容器管理所有組件的生命週期。控制器、服務、Repository均通過構造函數注入獲取依賴,這種設計不僅實現了鬆耦合,更使得組件的依賴關係在類別定義處即可一目了然。特別值得關注的是Service層對`IOptionsSnapshot<T>`的注入方式,這種配置注入模式允許系統在運行時動態讀取配置變更,為生產環境的靈活配置管理提供了技術基礎。

策略模式(Strategy Pattern)在訓練類型識別邏輯中得到了巧妙應用。[RunType.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.Model/Shared/RunType.cs) 通過靜態只讀集合定義了多種訓練類型,每種類型封裝了自身的匹配邏輯。`Classify`方法通過遍歷所有類型並調用其`Matches`方法,實現了運動名稱到訓練類型的映射。這種設計避免了龐大的if-else分支結構,新增訓練類型時僅需添加新的RunType實例,無需修改分類邏輯,體現了開閉原則(Open-Closed Principle)的應用。

工廠模式(Factory Pattern)在資料庫連線管理中發揮了關鍵作用。[SqliteConnectionFactory.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.Repository/Infrastructure/SqliteConnectionFactory.cs) 通過`Create`方法封裝了SQLite連線的創建邏輯,並在首次連線時自動執行資料庫初始化。這種設計將連線創建的複雜性(如連接字串解析、資料庫檔案檢查、建表腳本執行)隔離在工廠內部,使用方僅需調用`Create`方法即可獲得可用的連線物件。特別值得注意的是,工廠內部實現了雙重檢查鎖定(Double-Check Locking)模式,確保多線程環境下資料庫僅被初始化一次。

## 通信與並發模式

系統在並發處理方面採用了多種模式以確保線程安全與性能平衡。Singleton模式在 [StravaTokenStore.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.Service/Stores/StravaTokenStore.cs) 中被用於實現全局唯一的Token快取,通過依賴注入容器的Singleton生命週期配置,確保整個應用程序僅存在一個TokenStore實例。這種設計使得所有請求能夠共享同一組Access Token,避免了重複的認證開銷。然而,字典結構的讀寫操作在多線程環境下可能存在競爭條件,系統通過背景服務的定期全量更新策略緩解了這一風險:Token刷新時會先清空字典再重新填充,避免了細粒度的增刪改操作可能導致的併發問題。

背景服務模式(Background Service Pattern)在Token管理中得到了充分應用。[StravaTokenBackgroundService.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.Service/Background/StravaTokenBackgroundService.cs) 通過繼承`BackgroundService`基類,實現了一個長期運行的背景任務。該服務在應用啟動時立即執行一次Token刷新,隨後每五小時自動重複執行。這種設計將Token的生命週期管理從業務請求中剝離,避免了業務邏輯中夾雜Token過期檢查與刷新的代碼,體現了關注點分離(Separation of Concerns)原則。值得關注的是,背景服務通過`IServiceProvider.CreateScope()`創建獨立的依賴注入作用域,確保每次Token刷新都使用新的服務實例,避免了長生命週期服務可能導致的狀態污染。

異步編程模式(Async/Await Pattern)在系統中被廣泛採用,幾乎所有的I/O密集型操作(如HTTP請求、資料庫查詢)均使用async/await語法實現。從 [StravaService.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.Service/StravaService.cs) 的`InsertTrainingRecordsAsync`方法可以看出,系統通過`Task.WhenAll`並行執行多個用戶的Token刷新請求,這種並發策略顯著提升了多用戶場景下的響應效率。異步方法鏈的設計確保了線程不會在等待I/O操作時被阻塞,從而提升了系統的並發吞吐能力。

## 橫切關注點實現

中介軟體管道(Middleware Pipeline)模式是ASP.NET Core處理橫切關注點的核心機制。[TraceIdMiddleware.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.MiddleWare/TraceIdMiddleware.cs) 通過實現中介軟體介面,在HTTP請求處理管道中注入了分散式追蹤邏輯。該中介軟體在請求到達時檢查是否包含`X-Trace-Id`標頭,若不存在則自動生成唯一標識,並在響應返回前將其添加至響應標頭。通過`LogContext.PushProperty`,TraceId被推送至Serilog的日誌上下文,使得該請求生命週期內的所有日誌記錄都自動包含該標識。這種設計實現了全鏈路的請求追蹤能力,為分散式環境下的問題定位提供了關鍵支撐。

擴展方法模式(Extension Method Pattern)在Common層被廣泛應用,用於增強基礎類型的功能。[DateTimeExtensions.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.Common/Extensions/DateTimeExtensions.cs) 提供了一系列DateTime的擴展方法,如`ToUnixTimeSecondsSafe`、`ToTaiwanTime`、`GetWeekRange`等,這些方法將複雜的日期時間處理邏輯封裝為語義化的擴展方法,使得業務代碼能夠以鏈式調用的優雅方式完成複雜計算。擴展方法的優勢在於無需修改原始類型即可添加新功能,同時避免了工具類方法的靜態依賴問題。

裝飾器模式(Decorator Pattern)的思想在日誌處理中得到體現。Serilog通過`LogContext.PushProperty`方法實現了日誌上下文的動態裝飾,中介軟體無需直接操作日誌記錄器,僅需向上下文中推送屬性,即可讓該作用域內的所有日誌自動包含這些屬性。這種設計使得橫切關注點的添加不需要侵入式修改業務代碼,保持了代碼的整潔性。

## 抽象與復用策略

系統在抽象設計上展現出對單一職責原則(Single Responsibility Principle)的堅持。[BaseRepository.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.Repository/BaseRepository.cs) 作為所有Repository的基類,封裝了資料庫連線的獲取邏輯,子類別僅需關注具體的SQL查詢操作,無需重複編寫連線管理代碼。這種基類抽象策略不僅減少了代碼重複,更提供了統一的連線管理點,未來若需添加連線池或連線攔截邏輯,僅需在基類中修改即可影響所有Repository。

泛型約束在模型設計中被用於實現類型安全的復用。[ResponseBase<T>.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.Model/Response/ResponseBase.cs) 通過泛型定義了統一的響應結構,包含成功標誌、訊息、數據載荷等欄位。所有API響應均通過該泛型類別包裝,確保了響應格式的一致性。這種泛型設計避免了為每種數據類型定義重複的響應類別,同時通過編譯期的類型檢查確保了數據載荷的類型安全。

值對象(Value Object)模式在領域模型中得到應用。[RunType.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.Model/Shared/RunType.cs) 通過密封類(sealed class)與私有構造函數實現了不可變的訓練類型定義,所有實例均為靜態只讀欄位,這確保了訓練類型在系統運行期間的唯一性與不可變性。值對象的樹狀結構(通過Parent屬性實現)允許訓練類型具有層級關係,如"短間歇"與"長間歇"均屬於"強度訓練"類別,這種設計使得系統能夠靈活處理不同粒度的訓練類型查詢與統計。

批次處理模式(Batch Processing Pattern)在Repository層得到優化應用。[TrainingRecordRepository.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.Repository/TrainingRecordRepository.cs) 的`BatchInsertDuplicateKeyUpdateAsync`方法通過動態構建批次SQL語句,將多筆記錄的插入或更新操作合併為單一的資料庫往返,顯著提升了大量數據寫入的性能。該方法還實現了分批處理邏輯,通過`GroupBy`將大批量數據切分為多個小批次,避免了單一SQL語句過大導致的執行失敗或內存溢出問題。

從整體視角審視,Mtr.StravaConnect的設計模式應用體現了對經典設計原則的深刻理解與靈活運用。系統並非教條式地套用設計模式,而是根據實際業務場景選擇最合適的模式組合,在保持代碼可讀性與可維護性的同時,實現了高效的功能交付。這種務實而優雅的設計策略為系統的長期演進奠定了堅實基礎。
