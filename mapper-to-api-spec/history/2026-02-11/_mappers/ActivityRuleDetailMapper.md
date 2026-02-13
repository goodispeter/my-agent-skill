# ActivityRuleDetailMapper 對應表

本文件記錄 `ActivityRuleDetailMapper` 所有方法與對應 API 端點的對照關係。

## Mapper 資訊

- **類別名稱**: `com.tutorabc.product.studyprogress.dal_products.DBMapper.TutorERP.ActivityRuleDetailMapper`
- **類型**: MyBatis Interface Mapper
- **方法總數**: 2
- **對應資源**: [activity-rule-details.md](../activity-rule-details.md)

## 方法對應表

| 原始方法 | 參數 | 返回類型 | 對應 API 端點 | HTTP 方法 | 還原方式 |
|---------|------|---------|-------------|----------|----------|
| selectAll | - | List&lt;ActivityRuleDetailEntity&gt; | `/activity-rule-details` | GET | 不傳任何參數 |
| selectByLevelNo | String levelNo | List&lt;ActivityRuleDetailEntity&gt; | `/activity-rule-details?levelNo={levelNo}` | GET | 傳入 levelNo 參數 |

## 合併說明

### 查詢方法合併（2 個方法 → 1 個 API）

以下 2 個查詢方法已合併為單一 API 端點 `GET /activity-rule-details`：

1. **selectAll()**
   - API 呼叫：`GET /activity-rule-details`
   - 說明：不傳任何參數，返回全部規則明細

2. **selectByLevelNo(String levelNo)**
   - API 呼叫：`GET /activity-rule-details?levelNo=LEVEL_1`
   - 說明：傳入 levelNo 參數進行篩選
   - 範例：`GET /activity-rule-details?levelNo=LEVEL_2`

**合併優勢**：
- 減少 2 個 API 端點為 1 個
- 提供統一的查詢介面
- 符合 RESTful 設計原則
- 簡化 API 維護

## 遷移指南

### 程式碼遷移範例

**原始程式碼（使用 Mapper）**：
```java
// 範例 1: 查詢全部規則明細
List<ActivityRuleDetailEntity> all = activityRuleDetailMapper.selectAll();

// 範例 2: 依等級編號查詢
List<ActivityRuleDetailEntity> level1 = 
    activityRuleDetailMapper.selectByLevelNo("LEVEL_1");
```

**遷移後程式碼（使用 API Client）**：
```java
// 範例 1: 查詢全部規則明細
List<ActivityRuleDetailResponse> all = apiClient.getActivityRuleDetails(null);

// 範例 2: 依等級編號查詢
List<ActivityRuleDetailResponse> level1 = 
    apiClient.getActivityRuleDetails("LEVEL_1");
```

## 注意事項

1. **返回類型變化**: Mapper 返回 `ActivityRuleDetailEntity`，API 返回 `ActivityRuleDetailResponse`（DTO）
2. **資料關聯**: 活動規則明細與活動規則主檔、獎勵資訊有關聯性
3. **快取建議**: 規則資料變動頻率低，建議實作 5-10 分鐘的本地快取
4. **效能考量**: 從直接資料庫查詢改為 HTTP API 調用，需注意網路延遲

## 相關資源

- API 規格文件: [activity-rule-details.md](../activity-rule-details.md)
- 相關 Mapper: [ActivityRuleHeadMapper.md](ActivityRuleHeadMapper.md)
- 相關 Mapper: [ActivityRuleRewardMapper.md](ActivityRuleRewardMapper.md)
