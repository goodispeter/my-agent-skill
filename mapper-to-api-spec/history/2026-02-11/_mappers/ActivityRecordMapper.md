# ActivityRecordMapper 對應表

本文件記錄 `ActivityRecordMapper` 所有方法與對應 API 端點的對照關係。

## Mapper 資訊

- **類別名稱**: `com.tutorabc.product.studyprogress.dal_products.DBMapper.TutorERP.ActivityRecordMapper`
- **類型**: MyBatis Interface Mapper
- **方法總數**: 2
- **對應資源**: [activity-records.md](../activity-records.md)

## 方法對應表

| 原始方法 | 參數 | 返回類型 | 對應 API 端點 | HTTP 方法 | 還原方式 |
|---------|------|---------|-------------|----------|----------|
| selectbySn | @Param("from") Integer from, @Param("to") Integer to | List&lt;ActivityRecordEntity&gt; | `/activity-records?from={from}&to={to}` | GET | from 和 to 皆為必填參數 |
| updateCalculateResult | List&lt;ActivityRecordEntity&gt; records | Integer | `/activity-records/calculate-results` | PUT | Request Body 傳入更新記錄陣列 |

## 設計說明

### 混合 CRUD 設計

本 Mapper 包含查詢與更新兩種操作，對應為不同的 API 端點：

1. **selectbySn(Integer from, Integer to)** → `GET /activity-records?from={from}&to={to}`
   - 用途：依 SN 範圍查詢活動記錄
   - 返回：陣列
   - 必填參數：from（起始 SN）、to（結束 SN）
   - HTTP 方法：GET（查詢操作）

2. **updateCalculateResult(List<ActivityRecordEntity> records)** → `PUT /activity-records/calculate-results`
   - 用途：批次更新活動記錄的計算結果
   - 返回：更新筆數與詳細結果
   - 必填參數：records（更新記錄陣列）
   - HTTP 方法：PUT（更新操作）

**設計理由**：
- 查詢與更新操作語意不同，應使用不同的 HTTP 方法和端點
- 範圍查詢使用 GET 符合 RESTful 慣例
- 批次更新使用 PUT 表示資源更新操作
- 更新端點使用具體的子資源路徑（/calculate-results）明確表達操作意圖

## 遷移指南

### 程式碼遷移範例

**原始程式碼（使用 Mapper）**：
```java
// 範例 1: 範圍查詢活動記錄
List<ActivityRecordEntity> records = 
    activityRecordMapper.selectbySn(1, 100);

// 範例 2: 批次更新計算結果
List<ActivityRecordEntity> updates = new ArrayList<>();
ActivityRecordEntity record1 = new ActivityRecordEntity();
record1.setSn(1);
record1.setCalculateResult("COMPLETED");
updates.add(record1);

ActivityRecordEntity record2 = new ActivityRecordEntity();
record2.setSn(2);
record2.setCalculateResult("FAILED");
updates.add(record2);

Integer updatedCount = activityRecordMapper.updateCalculateResult(updates);
System.out.println("Updated: " + updatedCount);
```

**遷移後程式碼（使用 API Client）**：
```java
// 範例 1: 範圍查詢活動記錄
List<ActivityRecordResponse> records = 
    apiClient.getActivityRecords(1, 100);

// 範例 2: 批次更新計算結果
List<ActivityRecordUpdateRequest> updates = new ArrayList<>();
updates.add(new ActivityRecordUpdateRequest(1, "COMPLETED"));
updates.add(new ActivityRecordUpdateRequest(2, "FAILED"));

ActivityRecordUpdateResponse response = 
    apiClient.updateActivityRecordCalculateResults(updates);
System.out.println("Updated: " + response.getUpdatedCount());
System.out.println("Failed: " + response.getFailedCount());
```

## 注意事項

1. **返回類型變化**: 
   - 查詢：Mapper 返回 `List<ActivityRecordEntity>`，API 返回 `List<ActivityRecordResponse>`
   - 更新：Mapper 返回 `Integer`（更新筆數），API 返回詳細的更新結果物件
2. **必填參數**: 
   - 查詢：from 和 to 皆為必填
   - 更新：records 陣列不能為空
3. **範圍限制**: 建議限制 from/to 範圍（例如：最多 1000 筆）和更新陣列大小（例如：最多 100 筆）
4. **部分成功處理**: API 更新操作返回詳細的成功/失敗資訊，比 Mapper 單純返回更新筆數更具彈性
5. **冪等性**: 更新操作應支援冪等性，重複調用不會產生副作用
6. **快取考量**: 
   - 查詢結果建議快取 3-5 分鐘
   - 更新後應清除相關快取
7. **效能考量**: 批次更新需注意資料量控制，避免單次更新過多記錄

## 相關資源

- API 規格文件: [activity-records.md](../activity-records.md)
- 相關 Mapper: [CfgActivityMapper.md](CfgActivityMapper.md)
