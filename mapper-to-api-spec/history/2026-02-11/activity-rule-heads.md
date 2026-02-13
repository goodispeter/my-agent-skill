# Activity Rule Heads API

## 概述

此 API 提供活動規則主檔的查詢功能，包含活動規則的主要資訊與配置。

**對應 Mapper**: `ActivityRuleHeadMapper`  
**合併方法數**: 2 個方法 → 1 個 API 端點

## API 端點

### 查詢活動規則主檔

```
GET /api/v1/activity-rule-heads
```

查詢活動規則主檔列表，支援依活動 SN 篩選。

#### 請求參數

| 參數名稱 | 類型 | 位置 | 必填 | 說明 | 範例 |
|---------|------|------|------|------|------|
| activitySn | integer | query | 否 | 活動 SN 篩選 | 1001 |

#### 回應格式

**成功回應** (HTTP 200)

```json
{
  "success": true,
  "data": [
    {
      "sn": 1,
      "activitySn": 1001,
      "ruleName": "新春活動規則",
      "ruleCode": "SPRING_2024",
      "ruleType": "REWARD",
      "description": "新春活動獎勵規則",
      "startDate": "2024-01-01",
      "endDate": "2024-02-29",
      "isActive": true,
      "priority": 10,
      "createdBy": "admin",
      "createdAt": "2024-01-01T00:00:00Z",
      "updatedBy": "admin",
      "updatedAt": "2024-01-15T12:00:00Z"
    }
  ],
  "total": 1,
  "timestamp": "2024-02-11T10:00:00Z"
}
```

#### 錯誤回應

**參數錯誤** (HTTP 400)
```json
{
  "success": false,
  "error": {
    "code": "INVALID_PARAMETER",
    "message": "activitySn 必須為正整數",
    "details": "activitySn: -1 不合法"
  },
  "timestamp": "2024-02-11T10:00:00Z"
}
```

**伺服器錯誤** (HTTP 500)
```json
{
  "success": false,
  "error": {
    "code": "INTERNAL_ERROR",
    "message": "查詢活動規則主檔失敗",
    "details": "資料庫連線異常"
  },
  "timestamp": "2024-02-11T10:00:00Z"
}
```

## DTO 定義

### ActivityRuleHeadResponse

| 欄位名稱 | 類型 | 說明 | 範例 |
|---------|------|------|------|
| sn | integer | 規則主檔 SN (主鍵) | 1 |
| activitySn | integer | 活動 SN (外鍵) | 1001 |
| ruleName | string | 規則名稱 | "新春活動規則" |
| ruleCode | string | 規則代碼 | "SPRING_2024" |
| ruleType | string | 規則類型 | "REWARD" |
| description | string | 規則描述 | "新春活動獎勵規則" |
| startDate | string | 開始日期 (YYYY-MM-DD) | "2024-01-01" |
| endDate | string | 結束日期 (YYYY-MM-DD) | "2024-02-29" |
| isActive | boolean | 是否啟用 | true |
| priority | integer | 優先順序 | 10 |
| createdBy | string | 建立者 | "admin" |
| createdAt | string | 建立時間 (ISO 8601) | "2024-01-01T00:00:00Z" |
| updatedBy | string | 更新者 | "admin" |
| updatedAt | string | 更新時間 (ISO 8601) | "2024-01-15T12:00:00Z" |

### RuleType 列舉值

| 值 | 說明 |
|----|------|
| REWARD | 獎勵規則 |
| DISCOUNT | 折扣規則 |
| CONDITION | 條件規則 |
| PROMOTION | 促銷規則 |

## Mapper 方法對應

### 參數還原對應表

| 原始 Mapper 方法 | 對應 API 呼叫 | 說明 |
|-----------------|--------------|------|
| `selectAll()` | `GET /activity-rule-heads` | 不傳任何參數，查詢全部規則主檔 |
| `selectByActivitySn(int activitySn)` | `GET /activity-rule-heads?activitySn={activitySn}` | 傳入 activitySn 參數篩選 |

### 程式碼遷移範例

**遷移前 (Mapper)**:
```java
// 查詢全部
List<ActivityRuleHeadEntity> allRules = activityRuleHeadMapper.selectAll();

// 依活動 SN 查詢
List<ActivityRuleHeadEntity> activityRules = 
    activityRuleHeadMapper.selectByActivitySn(1001);
```

**遷移後 (API Client)**:
```java
// 查詢全部
List<ActivityRuleHeadResponse> allRules = 
    apiClient.getActivityRuleHeads(null);

// 依活動 SN 查詢
List<ActivityRuleHeadResponse> activityRules = 
    apiClient.getActivityRuleHeads(1001);
```

## 使用範例

### 範例 1: 查詢全部活動規則主檔

**請求**:
```http
GET /api/v1/activity-rule-heads HTTP/1.1
Host: api.example.com
Accept: application/json
Authorization: Bearer {access_token}
```

**回應**:
```json
{
  "success": true,
  "data": [
    {
      "sn": 1,
      "activitySn": 1001,
      "ruleName": "新春活動規則",
      "ruleCode": "SPRING_2024",
      "ruleType": "REWARD",
      "description": "新春活動獎勵規則",
      "startDate": "2024-01-01",
      "endDate": "2024-02-29",
      "isActive": true,
      "priority": 10,
      "createdBy": "admin",
      "createdAt": "2024-01-01T00:00:00Z",
      "updatedBy": "admin",
      "updatedAt": "2024-01-15T12:00:00Z"
    },
    {
      "sn": 2,
      "activitySn": 1002,
      "ruleName": "夏季促銷規則",
      "ruleCode": "SUMMER_2024",
      "ruleType": "DISCOUNT",
      "description": "夏季課程折扣規則",
      "startDate": "2024-06-01",
      "endDate": "2024-08-31",
      "isActive": true,
      "priority": 5,
      "createdBy": "admin",
      "createdAt": "2024-05-01T00:00:00Z",
      "updatedBy": "admin",
      "updatedAt": "2024-05-15T12:00:00Z"
    }
  ],
  "total": 2,
  "timestamp": "2024-02-11T10:00:00Z"
}
```

### 範例 2: 依活動 SN 查詢

**請求**:
```http
GET /api/v1/activity-rule-heads?activitySn=1001 HTTP/1.1
Host: api.example.com
Accept: application/json
Authorization: Bearer {access_token}
```

**回應**:
```json
{
  "success": true,
  "data": [
    {
      "sn": 1,
      "activitySn": 1001,
      "ruleName": "新春活動規則",
      "ruleCode": "SPRING_2024",
      "ruleType": "REWARD",
      "description": "新春活動獎勵規則",
      "startDate": "2024-01-01",
      "endDate": "2024-02-29",
      "isActive": true,
      "priority": 10,
      "createdBy": "admin",
      "createdAt": "2024-01-01T00:00:00Z",
      "updatedBy": "admin",
      "updatedAt": "2024-01-15T12:00:00Z"
    }
  ],
  "total": 1,
  "timestamp": "2024-02-11T10:00:00Z"
}
```

## 注意事項

1. **日期有效性**: 注意檢查 startDate 和 endDate 的有效期間
2. **規則優先順序**: priority 數值越大優先順序越高
3. **快取策略**: 規則資料相對穩定，建議實作快取機制，快取時間建議 10-15 分鐘
4. **規則關聯**: 規則主檔與規則明細 (activity-rule-details) 和獎勵 (activity-rule-rewards) 有一對多關係
5. **效能考量**: 若需要同時查詢規則明細和獎勵，建議實作關聯查詢 API

## 相關資源

- Mapper 對應文件: [_mappers/ActivityRuleHeadMapper.md](_mappers/ActivityRuleHeadMapper.md)
- 相關 API: [activity-rule-details.md](activity-rule-details.md)
- 相關 API: [activity-rule-rewards.md](activity-rule-rewards.md)
- 相關 API: [cfg-activities.md](cfg-activities.md)
