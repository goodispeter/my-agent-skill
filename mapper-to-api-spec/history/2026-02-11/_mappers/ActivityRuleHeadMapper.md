# ActivityRuleHeadMapper 對應表

本文件記錄 `ActivityRuleHeadMapper` 所有方法與對應 API 端點的對照關係。

## Mapper 資訊

- **類別名稱**: `com.tutorabc.product.studyprogress.dal_products.DBMapper.TutorERP.ActivityRuleHeadMapper`
- **類型**: MyBatis Interface Mapper
- **方法總數**: 2
- **對應資源**: [activity-rule-heads.md](../activity-rule-heads.md)

## 方法對應表

| 原始方法 | 參數 | 返回類型 | 對應 API 端點 | HTTP 方法 | 還原方式 |
|---------|------|---------|-------------|----------|----------|
| selectAll | - | List&lt;ActivityRuleHeadEntity&gt; | `/activity-rule-heads` | GET | 不傳任何參數 |
| selectByActivitySn | int activitySn | List&lt;ActivityRuleHeadEntity&gt; | `/activity-rule-heads?activitySn={activitySn}` | GET | 傳入 activitySn 參數 |

## 合併說明

### 查詢方法合併（2 個方法 → 1 個 API）

以下 2 個查詢方法已合併為單一 API 端點 `GET /activity-rule-heads`：

1. **selectAll()**
   - API 呼叫：`GET /activity-rule-heads`
   - 說明：不傳任何參數，返回全部規則主檔

2. **selectByActivitySn(int activitySn)**
   - API 呼叫：`GET /activity-rule-heads?activitySn=1001`
   - 說明：傳入 activitySn 參數進行篩選
   - 範例：`GET /activity-rule-heads?activitySn=1002`

**合併優勢**：
- 減少 2 個 API 端點為 1 個
- 提供統一的查詢介面
- 符合 RESTful 設計原則
- 簡化 API 維護

## 遷移指南

### 程式碼遷移範例

**原始程式碼（使用 Mapper）**：
```java
// 範例 1: 查詢全部規則主檔
List<ActivityRuleHeadEntity> all = activityRuleHeadMapper.selectAll();

// 範例 2: 依活動 SN 查詢
List<ActivityRuleHeadEntity> activityRules = 
    activityRuleHeadMapper.selectByActivitySn(1001);
```

**遷移後程式碼（使用 API Client）**：
```java
// 範例 1: 查詢全部規則主檔
List<ActivityRuleHeadResponse> all = apiClient.getActivityRuleHeads(null);

// 範例 2: 依活動 SN 查詢
List<ActivityRuleHeadResponse> activityRules = 
    apiClient.getActivityRuleHeads(1001);
```

## 注意事項

1. **返回類型變化**: Mapper 返回 `ActivityRuleHeadEntity`，API 返回 `ActivityRuleHeadResponse`（DTO）
2. **資料關聯**: 活動規則主檔與活動規則明細、獎勵資訊有一對多關係
3. **日期處理**: startDate 和 endDate 需注意有效期間判斷
4. **快取建議**: 規則資料變動頻率低，建議實作 10-15 分鐘的本地快取
5. **效能考量**: 從直接資料庫查詢改為 HTTP API 調用，需注意網路延遲

## 相關資源

- API 規格文件: [activity-rule-heads.md](../activity-rule-heads.md)
- 相關 Mapper: [ActivityRuleDetailMapper.md](ActivityRuleDetailMapper.md)
- 相關 Mapper: [ActivityRuleRewardMapper.md](ActivityRuleRewardMapper.md)
- 相關 API: [cfg-activities.md](../cfg-activities.md)
