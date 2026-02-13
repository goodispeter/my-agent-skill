# 核心算法與業務邏輯

## 訓練記錄處理算法

系統的核心業務邏輯圍繞訓練記錄的擷取、轉換與聚合展開。從 [StravaService.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.Service/StravaService.cs) 的`InsertTrainingRecordsAsync`方法可以看出完整的處理流程。首先,系統通過TokenStore獲取有效的Access Token,這一步驟包含了Token可用性檢查,若Token不存在則主動拋出UnauthorizedAccessException,遵循快速失敗(Fail-Fast)原則。隨後,系統通過資料庫查詢獲取該用戶最後一筆訓練記錄的日期,將其轉換為Unix時間戳並加1秒作為查詢起始點,這種增量擷取策略避免了重複數據的拉取,體現了對API資源的節約使用。

活動數據的過濾邏輯展現了對業務規則的精確建模。系統僅保留SportType包含"Run"、"WeightTraining"或"Yoga"的活動記錄,這一過濾條件通過LINQ的`Where`子句實現,使得代碼具有良好的可讀性。過濾後的活動數據需要經過複雜的轉換才能存儲為訓練記錄實體,這一轉換過程包含了多項計算邏輯。

配速計算算法採用了標準的運動科學公式。系統通過Strava返回的平均速度(單位:米/秒)計算每公里所需秒數,公式為`secPerKm = 1000 / averageSpeed`。計算結果被轉換為分鐘與秒的組合格式,如"5:30"表示每公里5分30秒。這種轉換邏輯通過整數除法與取模運算實現,確保了結果的數值正確性。值得注意的是,系統在計算前會檢查速度是否大於0,避免了除零異常的發生,體現了防禦性編程思想。

訓練計畫週數的計算邏輯運用了日期時間運算技巧。系統通過 [DateTimeExtensions.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.Common/Extensions/DateTimeExtensions.cs) 的`CalculatePlanWeek`方法,首先獲取活動日期與計畫開始日期各自所在週的起始日,隨後通過日期差計算週數差異。這種基於週起始日的計算策略確保了跨週活動的週數歸屬準確性,避免了簡單日期差除以7可能導致的邊界錯誤。

## 跑步類型識別算法

跑步類型的自動識別是系統的一項關鍵智能化功能。[RunType.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.Model/Shared/RunType.cs) 的`Classify`方法實現了基於活動名稱的類型推斷。算法首先對活動名稱進行預處理,檢查開頭是否為已知的賽事關鍵字(通過遍歷RaceTarget.List實現),若匹配則移除該關鍵字後再進行類型識別。這種預處理邏輯避免了賽事名稱干擾訓練類型判斷的情況,如"馬拉松 長距離"應被識別為"長距離"而非"馬拉松賽事"。

類型匹配邏輯採用了多重策略組合。每個RunType實例的`Matches`方法通過將活動名稱拆分為單詞數組,檢查是否有單詞與類型代碼(Code)完全匹配,或者整體名稱是否包含英文名(En)或中文名(Zh)。這種多語言支持的匹配策略使得系統能夠識別"I-S"、"Interval Short"、"短間歇"等多種命名方式。匹配過程使用了大小寫不敏感的比較,提升了用戶輸入的容錯性。

分類算法的優先級策略體現了對業務規則的深度理解。系統首先嘗試匹配具有父類型的訓練類型(如"短間歇"、"長間歇"),若匹配失敗再嘗試匹配頂層類型(如"強度訓練"、"慢跑")。這種兩階段匹配策略確保了更具體的類型優先被識別,避免了"短間歇"被誤判為"強度訓練"的情況。通過`FirstOrDefault`的LINQ方法鏈,算法在找到首個匹配項後即刻返回,體現了短路求值的效率優化。

## 數據聚合與統計算法

訓練記錄的多維度聚合是系統提供數據洞察的核心能力。從 [StravaService.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.Service/StravaService.cs) 的`GetTrainingRecordsAsync`方法可以看出,系統實現了按月聚合與按週聚合兩種維度的統計。按月聚合通過`GroupBy(r => new { r.StartDate.Year, r.StartDate.Month})`實現,匿名類型作為分組鍵確保了年月組合的唯一性。每個月度分組內部計算總距離(Sum)、總移動時間(通過擴展方法轉換為秒後累加再轉回HMS格式)、主訓練計數(Count)、總爬升(Sum)等統計指標。

按週聚合的實現運用了自定義的週範圍計算邏輯。`GetWeekRange`擴展方法通過計算日期與週一的偏移量,獲取該日期所在週的起始與結束日期。分組鍵為週範圍元組,確保了同一週的活動被正確歸類。這種基於日曆週的聚合策略符合運動訓練的實際需求,使得週期性訓練計畫的執行情況能夠被準確追蹤。

移動時間的聚合涉及複雜的格式轉換邏輯。原始數據以"HH:mm:ss"字串格式存儲,聚合時需先通過 [TimeExtensions.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.Common/Extensions/TimeExtensions.cs) 的`ToSeconds`方法轉換為總秒數,累加後再通過`ToHmsFormat`轉換回HMS格式。這種雙向轉換策略確保了時間累加的數值正確性,避免了直接操作時間字串可能導致的格式錯誤。

## 批次數據處理優化

系統在處理大量訓練記錄插入時採用了精心設計的批次處理策略。[TrainingRecordRepository.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.Repository/TrainingRecordRepository.cs) 的`BatchInsertDuplicateKeyUpdateAsync`方法首先將輸入的實體集合按指定批次大小進行分組,通過`GroupBy(_ => index++ / batchSize)`實現了優雅的分批邏輯。每個批次的數據被動態構建為單一的SQL語句,採用SQLite的`INSERT OR REPLACE`語法實現幂等性插入,這確保了重複執行不會產生數據重複。

SQL語句的動態構建涉及複雜的字串拼接與轉義處理。系統通過`Select`投影將每個實體轉換為VALUES子句的一行,字串類型欄位通過`Replace("'", "''")`進行單引號轉義,防止SQL注入風險。可空欄位通過三元運算符判斷,空值時輸出"NULL"字面量而非C#的null引用。這種細緻的值處理邏輯確保了SQL語句的語法正確性與安全性。

批次大小的選擇體現了對性能與穩定性的平衡。過大的批次會導致單一SQL語句超長,可能觸發資料庫的語句長度限制或內存溢出;過小的批次則會增加資料庫往返次數,降低整體吞吐量。系統通過參數化的批次大小設計,允許調用方根據實際數據量動態調整,體現了配置化的優化策略。

## 時區與日期處理算法

系統在處理日期時間時特別關注了時區轉換的正確性。[DateTimeExtensions.cs](D:/workspace/side/Mtr.StravaConnect/Mtr.StravaConnect.Common/Extensions/DateTimeExtensions.cs) 提供了`ToTaiwanTime`方法,通過`TimeZoneInfo.ConvertTimeFromUtc`將UTC時間轉換為台灣時區(UTC+8)。該方法首先根據DateTime的Kind屬性判斷時間類型,Local時間需先轉換為UTC,Unspecified時間則被視為UTC處理。這種分支邏輯確保了各種來源的時間數據都能被正確轉換。

Unix時間戳的轉換邏輯包含了重要的安全檢查。`ToUnixTimeSecondsSafe`方法在轉換前會檢查DateTime是否為預設值(0001-01-01),若是則返回null而非錯誤的時間戳。這種防禦性設計避免了未初始化日期被轉換為無效時間戳的問題,體現了對邊界條件的謹慎處理。

週標識符(WeekId)的計算採用了ISO 8601標準的週數定義。系統通過`Calendar.GetWeekOfYear`方法,結合FirstFourDayWeek規則與Monday作為週起始日,計算得到符合國際標準的週數。WeekId格式為`yyyyWW`,如"202539"表示2025年第39週,這種緊湊的數值格式便於資料庫索引與範圍查詢。

從整體視角審視,Mtr.StravaConnect的算法設計展現了對業務場景的深度理解與對工程細節的極致追求。系統通過精心設計的資料處理流程、智能化的類型識別邏輯、高效的批次處理策略以及嚴謹的時間處理算法,構建了一個穩健且高效的訓練管理平台。這些算法不僅滿足了當前的功能需求,更通過清晰的代碼結構與充分的錯誤處理,為未來的功能擴展與性能優化預留了充分空間。
