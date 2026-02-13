# 邏輯視點與分層架構

## 職責分配體系

Mtr.StravaConnect 系統採用了典型的六層架構設計,各層職責明確且邊界清晰。最上層的 WebApi 層作為系統的唯一對外入口,承擔 HTTP 協議適配、請求路由、響應封裝等職責。該層的設計哲學在於保持極簡,避免業務邏輯的滲入。從 [StravaController.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.WebApi/Controllers/StravaController.cs) 的實現可以看出,控制器方法僅包含參數接收與服務調用,真正的業務邏輯被完全委託給 Service 層,這種薄控制器設計確保了表現層的穩定性與可測試性。

Service 層是系統的業務核心,封裝了所有與 Strava 數據處理相關的業務規則。該層不僅負責單一業務操作的執行,更承擔了跨資料源的編排職責。以 [StravaService.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.Service/StravaService.cs) 中的訓練記錄插入流程為例,該服務需要協調 TokenStore 獲取憑證、通過 ExternalApi 擷取原始活動資料、執行業務規則過濾與轉換、最終通過 Repository 持久化至資料庫。這種多步驟的業務流編排展現了服務層作為業務邏輯中樞的核心價值。

Repository 層是資料存取的唯一通道,所有對 SQLite 資料庫的操作均被封裝在此層內部。該層通過介面抽象對外暴露資料存取能力,使得上層業務邏輯不感知具體的資料庫實現細節。特別值得關注的是,Repository 層實現了針對性能敏感場景的批次操作優化,如 [TrainingRecordRepository.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.Repository/TrainingRecordRepository.cs) 中的 `BatchInsertDuplicateKeyUpdateAsync` 方法,通過動態構建批次 SQL 語句,顯著提升了大量數據寫入的執行效率。

ExternalApi 層專注於第三方服務的整合,目前僅包含 Strava API 的封裝。該層通過 Refit 框架實現了聲明式的 HTTP 客戶端定義,將 API 調用抽象為簡潔的介面方法。這種設計不僅簡化了整合代碼,更通過強類型約束提供了編譯期的安全保障。當 Strava API 發生變更時,影響範圍被嚴格限定在此層內部,體現了防腐層模式的價值。

Model 層橫跨所有層級,定義了系統的核心資料結構與業務規則。該層包含了三類模型:用於 API 交互的 Request/Response 模型、用於資料庫持久化的 Entity 模型,以及封裝業務規則的共用類別(如 [RunType.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.Model/Shared/RunType.cs))。這種分類策略避免了模型的職責混雜,確保了各層級能夠使用最適合其場景的數據表達形式。

Common 層與 MiddleWare 層分別提供了橫切關注點的支援能力。Common 層通過擴展方法模式提供了系統級的工具函式,如日期時間處理、字串轉換等。MiddleWare 層則實現了 ASP.NET Core 管道中的攔截邏輯,如 [TraceIdMiddleware.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.MiddleWare/TraceIdMiddleware.cs) 通過在請求處理前後注入追蹤標識,實現了全鏈路的日誌關聯能力。

## 資料流向與約束

系統中的資料流動呈現清晰的單向傳遞特徵。外部 HTTP 請求首先抵達 WebApi 層的控制器,經過模型綁定與驗證後轉化為強類型的 Request 物件。控制器將 Request 物件傳遞給 Service 層的業務方法,服務層根據業務邏輯決定是否需要呼叫外部 API 或查詢資料庫。當需要與 Strava API 交互時,Service 層通過依賴注入獲取的 IStravaApiClient 介面發起 HTTP 請求,ExternalApi 層返回的原始資料經過服務層的業務規則處理後,轉化為符合系統領域模型的 Entity 物件,最終通過 Repository 層持久化至 SQLite 資料庫。

在資料回流路徑上,Repository 層查詢返回的 Entity 物件被服務層轉換為 Response 模型,這一轉換過程包含了業務聚合邏輯,如將訓練記錄按週或按月分組、計算統計數據等。服務層返回的 Response 物件被控制器包裝在統一的響應結構中(如 [ResponseBase.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.Model/Response/ResponseBase.cs)),最終序列化為 JSON 格式返回給前端應用。這種資料流向的設計確保了各層級僅處理與其職責相符的資料形態,避免了資料結構的跨層泄露。

系統在資料流動過程中實施了嚴格的約束規則。首先是單向依賴約束:上層可依賴下層的介面,但下層絕不依賴上層的實現。這一規則通過依賴注入容器的配置強制執行,確保了系統的可測試性。其次是資料隔離約束:Entity 模型僅在 Repository 與 Service 層之間流動,不直接暴露給 WebApi 層,這避免了資料庫結構的外洩。最後是防腐層約束:來自 Strava API 的原始資料必須經過 Service 層的轉換才能進入系統的核心領域模型,這確保了外部系統的變更不會直接衝擊內部業務邏輯。

## 邊界隔離策略

各層之間的隔離通過多種機制共同實現。介面抽象是最核心的隔離手段,Service 層與 Repository 層之間通過 ITrainingRecordRepository 等介面進行解耦,使得服務層不直接依賴具體的資料庫實現類別。這種隔離策略的價值在單元測試場景中得以充分體現:測試代碼可輕鬆構造假的 Repository 實現,無需啟動真實的資料庫環境即可完成業務邏輯的驗證。

DTO(Data Transfer Object)模式在層級間的資料傳遞中被廣泛應用。WebApi 層使用的 Request/Response 模型與 Repository 層使用的 Entity 模型之間存在明確的轉換邊界,這一轉換過程發生在 Service 層內部。以訓練記錄的插入流程為例,外部 API 返回的 ResponseStravaActivities 物件經過服務層的業務規則處理(如配速計算、跑步類型識別),轉化為 TrainingRecordEntity 物件,最終持久化至資料庫。這種 DTO 轉換策略確保了各層使用最適合其場景的資料結構,避免了單一模型的職責過載。

依賴注入容器在邊界隔離中扮演了關鍵角色。系統在 [Program.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.WebApi/Program.cs) 的啟動配置中註冊了所有層級的依賴關係,運行時由容器負責物件的創建與生命週期管理。這種控制反轉(Inversion of Control)機制使得各層級的實現類別僅關注自身職責,不需要了解依賴物件的具體創建過程。通過 Scoped 生命週期的配置,系統確保了每個 HTTP 請求擁有獨立的服務實例,避免了多線程場景下的狀態污染問題。

防腐層策略在 ExternalApi 層得到了充分應用。該層作為系統與外部世界的唯一接觸點,負責將 Strava API 的數據結構轉化為系統內部的領域模型。當 Strava API 發生版本升級或數據格式變更時,變更範圍被限定在 ExternalApi 層內部,上層的業務邏輯無需感知這種變化。這種設計策略顯著降低了系統對第三方服務的耦合度,提升了架構的穩定性。

## 異常處理流

系統採用了分層的異常處理策略,確保異常信息能夠在各層級間準確傳遞並最終轉化為用戶友好的響應。在最底層的 Repository 層,資料庫操作可能拋出 SqliteException 等底層異常,這些異常被允許向上傳播至 Service 層。服務層根據業務場景決定是否需要捕獲並轉換這些異常,例如將 SqliteException 轉化為更具業務語義的 DataAccessException。

Service 層是異常處理的核心決策點。該層負責識別哪些異常屬於預期的業務場景(如 Token 失效、數據不存在),哪些屬於系統級錯誤。對於業務異常,服務層通過返回特定的 Response 物件(包含錯誤碼與錯誤訊息)進行處理,避免異常的向上拋出。對於系統級異常,服務層允許其繼續向上傳播至 WebApi 層,由全域異常處理器統一捕獲。

從 [StravaService.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.Service/StravaService.cs) 的 `InsertTrainingRecordsAsync` 方法可以看出,系統對於關鍵業務異常(如 Token 不可用)採用了主動檢查並拋出 `UnauthorizedAccessException` 的策略。這種明確的異常類型使得上層能夠精準識別異常原因,從而採取針對性的處理措施。對於 Strava API 呼叫過程中可能發生的網絡異常或超時異常,系統選擇讓其自然向上傳播,最終由 WebApi 層的異常中介軟體統一轉化為 HTTP 500 錯誤響應。

MiddleWare 層的 [TraceIdMiddleware.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.MiddleWare/TraceIdMiddleware.cs) 在異常處理鏈中發揮了重要作用。該中介軟體確保每個請求都具有唯一的 TraceId,並將其推送至 Serilog 的日誌上下文。當異常發生時,所有相關的日誌記錄都會自動包含該 TraceId,使得開發團隊能夠快速定位問題的完整執行軌跡。這種分散式追蹤機制顯著提升了生產環境問題診斷的效率。

值得注意的是,系統在 Service 層實現了針對並發場景的異常處理邏輯。[StravaTokenBackgroundService.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.Service/Background/StravaTokenBackgroundService.cs) 的 Token 刷新邏輯包含了異常捕獲與日誌記錄,確保即使刷新失敗,背景服務也不會崩潰而是繼續運行並在下一個週期重試。這種韌性設計保障了系統在面對網絡波動或臨時性故障時的可用性。

從整體架構視角來看,Mtr.StravaConnect 的分層設計體現了對關注點分離原則的深刻理解。各層級通過清晰的職責定義、嚴格的依賴約束以及優雅的隔離機制,構建了一個高內聚、低耦合的系統結構。這種分層架構不僅滿足了當前的業務需求,更為未來的功能擴展、性能優化以及技術棧演進奠定了堅實的基礎。
