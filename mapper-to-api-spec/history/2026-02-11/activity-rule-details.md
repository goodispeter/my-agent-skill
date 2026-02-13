# Activity Rule Details API

## 概述

此 API 提供活動規則明細的查詢功能，包含活動等級、規則條件等詳細資訊。

**對應 Mapper**: `ActivityRuleDetailMapper`  
**合併方法數**: 2 個方法 → 1 個 API 端點

## API 端點

### 查詢活動規則明細

```
GET /api/v1/activity-rule-details
```

查詢活動規則明細列表，支援依等級編號篩選。

#### 請求參數

| 參數名稱 | 類型 | 位置 | 必填 | 說明 | 範例 |
|---------|------|------|------|------|------|
| levelNo | string | query | 否 | 等級編號篩選 | "LEVEL_1" |

#### 回應格式

**成功回應** (HTTP 200)

```json
{
  "success": true,
  "data": [
    {
      "levelNo": "LEVEL_1",
      "activitySn": 1001,
      "ruleName": "新手獎勵規則",
      "ruleDescription": "完成首次課程即可獲得獎勵",
      "conditionType": "FIRST_LESSON",
      "conditionValue": "1",
      "priority": 1,
      "isActive": true,
      "createdAt": "2024-01-01T00:00:00Z",
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
    "message": "levelNo 格式不正確",
    "details": "levelNo 應符合格式: LEVEL_[數字]"
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
    "message": "查詢活動規則明細失敗",
    "details": "資料庫連線異常"
  },
  "timestamp": "2024-02-11T10:00:00Z"
}
```

## DTO 定義

### ActivityRuleDetailResponse

| 欄位名稱 | 類型 | 說明 | 範例 |
|---------|------|------|------|
| levelNo | string | 等級編號 | "LEVEL_1" |
| activitySn | integer | 活動 SN | 1001 |
| ruleName | string | 規則名稱 | "新手獎勵規則" |
| ruleDescription | string | 規則描述 | "完成首次課程即可獲得獎勵" |
| conditionType | string | 條件類型 | "FIRST_LESSON" |
| conditionValue | string | 條件值 | "1" |
| priority | integer | 優先順序 | 1 |
| isActive | boolean | 是否啟用 | true |
| createdAt | string | 建立時間 (ISO 8601) | "2024-01-01T00:00:00Z" |
| updatedAt | string | 更新時間 (ISO 8601) | "2024-01-15T12:00:00Z" |

## Mapper 方法對應

### 參數還原對應表

| 原始 Mapper 方法 | 對應 API 呼叫 | 說明 |
|-----------------|--------------|------|
| `selectAll()` | `GET /activity-rule-details` | 不傳任何參數，查詢全部規則明細 |
| `selectByLevelNo(String levelNo)` | `GET /activity-rule-details?levelNo={levelNo}` | 傳入 levelNo 參數篩選 |

### 程式碼遷移範例

**遷移前 (Mapper)**:
```java
// 查詢全部
List<ActivityRuleDetailEntity> allRules = activityRuleDetailMapper.selectAll();

// 依等級查詢
List<ActivityRuleDetailEntity> level1Rules = 
    activityRuleDetailMapper.selectByLevelNo("LEVEL_1");
```

**遷移後 (API Client)**:
```java
// 查詢全部
List<ActivityRuleDetailResponse> allRules = 
    apiClient.getActivityRuleDetails(null);

// 依等級查詢
List<ActivityRuleDetailResponse> level1Rules = 
    apiClient.getActivityRuleDetails("LEVEL_1");
```

## 使用範例

### 範例 1: 查詢全部活動規則明細

**請求**:
```http
GET /api/v1/activity-rule-details HTTP/1.1
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
      "levelNo": "LEVEL_1",
      "activitySn": 1001,
      "ruleName": "新手獎勵規則",
      "ruleDescription": "完成首次課程即可獲得獎勵",
      "conditionType": "FIRST_LESSON",
      "conditionValue": "1",
      "priority": 1,
      "isActive": true,
      "createdAt": "2024-01-01T00:00:00Z",
      "updatedAt": "2024-01-15T12:00:00Z"
    },
    {
      "levelNo": "LEVEL_2",
      "activitySn": 1001,
      "ruleName": "進階獎勵規則",
      "ruleDescription": "完成 10 堂課程獲得獎勵",
      "conditionType": "LESSON_COUNT",
      "conditionValue": "10",
      "priority": 2,
      "isActive": true,
      "createdAt": "2024-01-01T00:00:00Z",
      "updatedAt": "2024-01-15T12:00:00Z"
    }
  ],
  "total": 2,
  "timestamp": "2024-02-11T10:00:00Z"
}
```

### 範例 2: 依等級編號查詢

**請求**:
```http
GET /api/v1/activity-rule-details?levelNo=LEVEL_1 HTTP/1.1
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
      "levelNo": "LEVEL_1",
      "activitySn": 1001,
      "ruleName": "新手獎勵規則",
      "ruleDescription": "完成首次課程即可獲得獎勵",
      "conditionType": "FIRST_LESSON",
      "conditionValue": "1",
      "priority": 1,
      "isActive": true,
      "createdAt": "2024-01-01T00:00:00Z",
      "updatedAt": "2024-01-15T12:00:00Z"
    }
  ],
  "total": 1,
  "timestamp": "2024-02-11T10:00:00Z"
}
```

## 注意事項

1. **資料一致性**: 活動規則明細與活動規則主檔 (activity-rule-heads) 需保持關聯性
2. **快取策略**: 規則資料相對穩定，建議實作快取機制，快取時間建議 5-10 分鐘
3. **效能考量**: 若資料量大，建議實作分頁功能
4. **權限控制**: 建議依使用者角色限制可查詢的規則明細

## 相關資源

- Mapper 對應文件: [_mappers/ActivityRuleDetailMapper.md](_mappers/ActivityRuleDetailMapper.md)
- 相關 API: [activity-rule-heads.md](activity-rule-heads.md)
- 相關 API: [activity-rule-rewards.md](activity-rule-rewards.md)
