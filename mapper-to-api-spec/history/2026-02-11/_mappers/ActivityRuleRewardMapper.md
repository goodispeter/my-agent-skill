# ActivityRuleRewardMapper 對應表

本文件記錄 `ActivityRuleRewardMapper` 所有方法與對應 API 端點的對照關係。

## Mapper 資訊

- **類別名稱**: `com.tutorabc.product.studyprogress.dal_products.DBMapper.TutorERP.ActivityRuleRewardMapper`
- **類型**: MyBatis Interface Mapper
- **方法總數**: 2
- **對應資源**: [activity-rule-rewards.md](../activity-rule-rewards.md)

## 方法對應表

| 原始方法 | 參數 | 返回類型 | 對應 API 端點 | HTTP 方法 | 還原方式 |
|---------|------|---------|-------------|----------|----------|
| selectAll | - | List&lt;ActivityRuleRewardEntity&gt; | `/activity-rule-rewards` | GET | 不傳任何參數 |
| selectBylevelNo | String levelNo | List&lt;ActivityRuleRewardEntity&gt; | `/activity-rule-rewards?levelNo={levelNo}` | GET | 傳入 levelNo 參數 |

## 合併說明

### 查詢方法合併（2 個方法 → 1 個 API）

以下 2 個查詢方法已合併為單一 API 端點 `GET /activity-rule-rewards`：

1. **selectAll()**
   - API 呼叫：`GET /activity-rule-rewards`
   - 說明：不傳任何參數，返回全部獎勵資訊

2. **selectBylevelNo(String levelNo)**
   - API 呼叫：`GET /activity-rule-rewards?levelNo=LEVEL_1`
   - 說明：傳入 levelNo 參數進行篩選
   - 範例：`GET /activity-rule-rewards?levelNo=LEVEL_2`

**合併優勢**：
- 減少 2 個 API 端點為 1 個
- 提供統一的查詢介面
- 符合 RESTful 設計原則
- 簡化 API 維護

## 遷移指南

### 程式碼遷移範例

**原始程式碼（使用 Mapper）**：
```java
// 範例 1: 查詢全部獎勵
List<ActivityRuleRewardEntity> all = activityRuleRewardMapper.selectAll();

// 範例 2: 依等級編號查詢
List<ActivityRuleRewardEntity> level1 = 
    activityRuleRewardMapper.selectBylevelNo("LEVEL_1");
```

**遷移後程式碼（使用 API Client）**：
```java
// 範例 1: 查詢全部獎勵
List<ActivityRuleRewardResponse> all = apiClient.getActivityRuleRewards(null);

// 範例 2: 依等級編號查詢
List<ActivityRuleRewardResponse> level1 = 
    apiClient.getActivityRuleRewards("LEVEL_1");
```

## 注意事項

1. **返回類型變化**: Mapper 返回 `ActivityRuleRewardEntity`，API 返回 `ActivityRuleRewardResponse`（DTO）
2. **資料關聯**: 活動規則獎勵與活動規則明細透過 levelNo 關聯
3. **獎勵有效期**: 需注意 validDays 欄位，計算獎勵有效期限
4. **快取建議**: 獎勵資料變動頻率低，建議實作 10-15 分鐘的本地快取
5. **效能考量**: 從直接資料庫查詢改為 HTTP API 調用，需注意網路延遲

## 相關資源

- API 規格文件: [activity-rule-rewards.md](../activity-rule-rewards.md)
- 相關 Mapper: [ActivityRuleDetailMapper.md](ActivityRuleDetailMapper.md)
- 相關 Mapper: [ActivityRuleHeadMapper.md](ActivityRuleHeadMapper.md)
