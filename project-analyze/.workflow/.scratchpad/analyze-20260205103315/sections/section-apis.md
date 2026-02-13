# API 設計與規範

## RESTful 架構風格

Mtr.StravaConnect系統採用RESTful architectural style作為API設計的指導原則,通過資源導向的URI設計與標準HTTP動詞的語義化映射,構建了清晰且符合業界慣例的API介面。從 [StravaController.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.WebApi/Controllers/StravaController.cs) 的路由配置可以看出,系統的所有端點均以`/api/strava`為基礎路徑,通過子資源路徑與HTTP方法的组合實現功能區分。

系統提供的三個核心端點展現了對RESTful原則的理解。`GET /api/strava/refresh-token`用於觸發Token刷新操作,雖然GET方法通常用於數據查詢,但在此場景下該端點返回刷新後的Token列表,可被視為對Token資源當前狀態的查詢,符合冪等性要求。`GET /api/strava/training-records`通過Query參數接收user、year、targetCode等過濾條件,返回符合條件的訓練記錄聚合數據,這種參數化查詢設計使得同一端點能夠靈活應對不同維度的數據存取需求。`POST /api/strava/training-records`則用於從Strava同步新的訓練記錄,通過請求體傳遞同步配置,符合POST方法用於資源創建或觸發副作用操作的語義。

## 統一響應格式設計

系統通過 [ResponseBase.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.Model/Response/ResponseBase.cs) 實現了統一的響應結構封裝,這種設計策略確保了所有API端點返回的數據具有一致的外層格式。ResponseBase<T>作為泛型基類,包含Status、Message、Data等標準欄位,前端應用無需針對不同端點實現差異化的響應解析邏輯,顯著降低了客戶端開發的複雜度。

統一響應格式的價值在標準化錯誤處理中得以充分體現。當業務邏輯執行成功時,系統通過`ResponseBase<T>.Ok(data)`靜態方法構建成功響應,Status欄位自動設置為成功狀態碼,Message可選填充成功提示信息。當業務異常發生時,系統可通過對應的Error方法構建錯誤響應,確保前端能夠以統一的方式判斷請求結果並提取錯誤信息。這種設計遵循了DRY(Don't Repeat Yourself)原則,將響應構建邏輯集中在ResponseBase類別中,避免了控制器方法中的重複代碼。

## 參數驗證與模型綁定

系統充分利用了ASP.NET Core的模型綁定與驗證機制,通過強類型的Request模型接收客戶端參數。[FromQuery]特性標註的參數會自動從URL查詢字串中提取並綁定,[FromBody]特性標註的參數則從請求體中反序列化。這種聲明式的參數綁定設計使得控制器方法的簽名清晰表達了數據來源,無需手動解析HttpContext即可獲取所需參數。

模型驗證邏輯通過Data Annotations實現集中管理。雖然當前實現中未見顯式的驗證特性標註,但ASP.NET Core的內建驗證管道會自動執行模型驗證,當驗證失敗時自動返回400 Bad Request響應並附帶詳細的驗證錯誤信息。這種框架級的驗證機制確保了無效請求在進入業務邏輯前即被攔截,遵循了快速失敗原則。

## CORS 與跨域策略

系統在 [Program.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.WebApi/Program.cs) 中配置了精細化的CORS策略,這對於前後端分離的現代Web應用至關重要。通過`AddCors`方法註冊名為"AllowFrontend"的策略,系統明確指定了允許的來源域名(`http://localhost:5173`用於開發環境,`https://goodispeter.github.io`用於生產環境),這種白名單機制防止了未授權域名的惡意請求,體現了對Web安全的重視。

CORS配置中的`AllowAnyHeader`與`AllowAnyMethod`策略提供了靈活性,允許前端攜帶任意HTTP標頭與使用任意HTTP方法。`AllowCredentials`的啟用使得前端能夠在跨域請求中攜帶Cookie或Authorization標頭,為未來可能的身份認證機制預留了空間。這種配置策略在安全性與易用性之間取得了良好平衡,既保護了API免受CSRF攻擊,又不過度限製合法客戶端的存取方式。

## API 版本管理考量

雖然當前系統未實現顯式的API版本控制,但架構設計為未來的版本演進預留了空間。Control路由前綴`/api/strava`可輕鬆擴展為`/api/v1/strava`,通過URL路徑的版本號區分不同版本的API。ResponseBase<T>的泛型設計確保了即使響應結構發生變更,也可通過引入新的Response模型類別實現版本隔離,而無需修改泛型基類。

未來若需支持多版本並存,系統可通過ASP.NET Core的API Versioning中介軟體實現優雅的版本路由。通過在控制器或方法級別標註版本特性,同一端點可提供多個版本的實現,客戶端通過URL路徑、查詢參數或標頭指定所需版本。這種策略使得系統能夠在不破壞現有客戶端的前提下引入破壞性變更,遵循了契約穩定性原則。

## 性能與快取策略

系統在API層面採用了記憶體內Token快取策略以優化性能。通過Singleton的StravaTokenStore,所有請求共享同一組預刷新的Access Token,避免了每次請求都執行OAuth認證流程的開銷。這種快取策略的有效性時間窗口為五小時(由背景服務的刷新間隔決定),在此期間API響應速度顯著提升。

對於訓練記錄查詢,系統未實現HTTP層面的快取(如Etag或Last-Modified),所有請求均直達資料庫進行實時查詢。這種設計選擇源於訓練數據的動態性:用戶隨時可能從Strava同步新的活動記錄,若啟用快取可能導致前端顯示過時數據。未來若查詢性能成為瓶頸,可考慮引入短期的記憶體快取(如Redis)配合主動失效策略,在保證數據新鮮度的前提下提升響應速度。

## 錯誤處理與狀態碼約定

系統採用標準HTTP狀態碼表達API執行結果。成功請求返回200 OK,配合ResponseBase中的成功標誌位提供應用層的成功確認。當業務邏輯拋出UnauthorizedAccessException時(如Token不可用),系統應配置全域異常處理器將其映射為401 Unauthorized響應,明確告知客戶端認證失敗。對於參數驗證失敗,ASP.NET Core自動返回400 Bad Request並附帶ModelState中的詳細錯誤信息。

未捕獲的異常會觸發500 Internal Server Error響應,這類錯誤通常指向系統級故障(如資料庫連線失敗、網絡超時)。系統通過Serilog記錄詳細的異常堆棧與上下文信息,配合TraceId實現問題的快速定位。這種分層的錯誤處理策略確保了客戶端能夠根據狀態碼採取針對性的處理邏輯,同時為後端開發團隊提供了充分的診斷信息。

從整體視角審視,Mtr.StravaConnect的API設計展現了對RESTful原則的深度理解與對工程實踐的務實應用。系統通過清晰的資源路由、統一的響應格式、靈活的CORS配置以及合理的性能優化策略,構建了一個易用、安全且高效的API服務層。這種設計不僅滿足了當前前後端協作的需求,更為未來的API演進與多端適配奠定了堅實基礎。
