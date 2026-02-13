# Activity Rule Rewards API

## 概述

此 API 提供活動規則獎勵的查詢功能，包含獎勵類型、獎勵內容等詳細資訊。

**對應 Mapper**: `ActivityRuleRewardMapper`  
**合併方法數**: 2 個方法 → 1 個 API 端點

## API 端點

### 查詢活動規則獎勵

```
GET /api/v1/activity-rule-rewards
```

查詢活動規則獎勵列表，支援依等級編號篩選。

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
      "sn": 1,
      "levelNo": "LEVEL_1",
      "rewardType": "POINTS",
      "rewardName": "學習積分",
      "rewardValue": "100",
      "rewardUnit": "點",
      "description": "完成任務獲得 100 學習積分",
      "imageUrl": "https://cdn.example.com/rewards/points.png",
      "isActive": true,
      "priority": 1,
      "validDays": 30,
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
    "message": "查詢活動規則獎勵失敗",
    "details": "資料庫連線異常"
  },
  "timestamp": "2024-02-11T10:00:00Z"
}
```

## DTO 定義

### ActivityRuleRewardResponse

| 欄位名稱 | 類型 | 說明 | 範例 |
|---------|------|------|------|
| sn | integer | 獎勵 SN (主鍵) | 1 |
| levelNo | string | 等級編號 (外鍵) | "LEVEL_1" |
| rewardType | string | 獎勵類型 | "POINTS" |
| rewardName | string | 獎勵名稱 | "學習積分" |
| rewardValue | string | 獎勵數值 | "100" |
| rewardUnit | string | 獎勵單位 | "點" |
| description | string | 獎勵描述 | "完成任務獲得 100 學習積分" |
| imageUrl | string | 獎勵圖片 URL | "https://cdn.example.com/rewards/points.png" |
| isActive | boolean | 是否啟用 | true |
| priority | integer | 優先順序 | 1 |
| validDays | integer | 有效天數 | 30 |
| createdAt | string | 建立時間 (ISO 8601) | "2024-01-01T00:00:00Z" |
| updatedAt | string | 更新時間 (ISO 8601) | "2024-01-15T12:00:00Z" |

### RewardType 列舉值

| 值 | 說明 |
|----|------|
| POINTS | 積分獎勵 |
| COUPON | 優惠券 |
| GIFT | 實體禮品 |
| DISCOUNT | 折扣券 |
| FREE_LESSON | 免費課程 |

## Mapper 方法對應

### 參數還原對應表

| 原始 Mapper 方法 | 對應 API 呼叫 | 說明 |
|-----------------|--------------|------|
| `selectAll()` | `GET /activity-rule-rewards` | 不傳任何參數，查詢全部獎勵 |
| `selectBylevelNo(String levelNo)` | `GET /activity-rule-rewards?levelNo={levelNo}` | 傳入 levelNo 參數篩選 |

### 程式碼遷移範例

**遷移前 (Mapper)**:
```java
// 查詢全部
List<ActivityRuleRewardEntity> allRewards = activityRuleRewardMapper.selectAll();

// 依等級查詢
List<ActivityRuleRewardEntity> level1Rewards = 
    activityRuleRewardMapper.selectBylevelNo("LEVEL_1");
```

**遷移後 (API Client)**:
```java
// 查詢全部
List<ActivityRuleRewardResponse> allRewards = 
    apiClient.getActivityRuleRewards(null);

// 依等級查詢
List<ActivityRuleRewardResponse> level1Rewards = 
    apiClient.getActivityRuleRewards("LEVEL_1");
```

## 使用範例

### 範例 1: 查詢全部活動規則獎勵

**請求**:
```http
GET /api/v1/activity-rule-rewards HTTP/1.1
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
      "levelNo": "LEVEL_1",
      "rewardType": "POINTS",
      "rewardName": "學習積分",
      "rewardValue": "100",
      "rewardUnit": "點",
      "description": "完成任務獲得 100 學習積分",
      "imageUrl": "https://cdn.example.com/rewards/points.png",
      "isActive": true,
      "priority": 1,
      "validDays": 30,
      "createdAt": "2024-01-01T00:00:00Z",
      "updatedAt": "2024-01-15T12:00:00Z"
    },
    {
      "sn": 2,
      "levelNo": "LEVEL_2",
      "rewardType": "COUPON",
      "rewardName": "課程優惠券",
      "rewardValue": "500",
      "rewardUnit": "元",
      "description": "500 元課程折扣券",
      "imageUrl": "https://cdn.example.com/rewards/coupon.png",
      "isActive": true,
      "priority": 2,
      "validDays": 60,
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
GET /api/v1/activity-rule-rewards?levelNo=LEVEL_1 HTTP/1.1
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
      "levelNo": "LEVEL_1",
      "rewardType": "POINTS",
      "rewardName": "學習積分",
      "rewardValue": "100",
      "rewardUnit": "點",
      "description": "完成任務獲得 100 學習積分",
      "imageUrl": "https://cdn.example.com/rewards/points.png",
      "isActive": true,
      "priority": 1,
      "validDays": 30,
      "createdAt": "2024-01-01T00:00:00Z",
      "updatedAt": "2024-01-15T12:00:00Z"
    }
  ],
  "total": 1,
  "timestamp": "2024-02-11T10:00:00Z"
}
```

## 注意事項

1. **獎勵有效期**: validDays 表示獎勵從發放日起的有效天數
2. **獎勵類型**: 不同 rewardType 需要不同的兌換流程
3. **圖片資源**: imageUrl 建議使用 CDN 加速
4. **快取策略**: 獎勵資料相對穩定，建議實作快取機制，快取時間建議 10-15 分鐘
5. **資料關聯**: 獎勵與活動規則明細 (activity-rule-details) 透過 levelNo 關聯
6. **效能考量**: 若需要同時查詢規則和獎勵，建議前端合併請求或實作組合查詢 API

## 相關資源

- Mapper 對應文件: [_mappers/ActivityRuleRewardMapper.md](_mappers/ActivityRuleRewardMapper.md)
- 相關 API: [activity-rule-details.md](activity-rule-details.md)
- 相關 API: [activity-rule-heads.md](activity-rule-heads.md)
