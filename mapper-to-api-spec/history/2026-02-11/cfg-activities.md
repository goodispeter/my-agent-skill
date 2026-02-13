# Configuration Activities API

## 概述

此 API 提供活動配置的查詢功能，包含活動的基本資訊、設定與狀態。

**對應 Mapper**: `CfgActivityMapper`  
**合併方法數**: 2 個方法 → 2 個 API 端點（RESTful 慣例）

## API 端點

### 1. 查詢全部活動配置

```
GET /api/v1/cfg-activities
```

查詢所有活動配置列表。

#### 請求參數

無

#### 回應格式

**成功回應** (HTTP 200)

```json
{
  "success": true,
  "data": [
    {
      "sn": 1001,
      "activityName": "新春活動",
      "activityCode": "SPRING_2024",
      "activityType": "PROMOTION",
      "description": "2024 新春促銷活動",
      "startDate": "2024-01-01",
      "endDate": "2024-02-29",
      "status": "ACTIVE",
      "targetAudience": "ALL",
      "maxParticipants": 1000,
      "currentParticipants": 256,
      "budgetAmount": 50000.00,
      "isActive": true,
      "createdBy": "admin",
      "createdAt": "2023-12-01T00:00:00Z",
      "updatedBy": "admin",
      "updatedAt": "2024-01-15T12:00:00Z"
    }
  ],
  "total": 1,
  "timestamp": "2024-02-11T10:00:00Z"
}
```

### 2. 查詢單一活動配置

```
GET /api/v1/cfg-activities/{sn}
```

根據活動配置 SN 查詢單筆資料。

#### 路徑參數

| 參數名稱 | 類型 | 必填 | 說明 | 範例 |
|---------|------|------|------|------|
| sn | integer | 是 | 活動配置 SN (主鍵) | 1001 |

#### 回應格式

**成功回應** (HTTP 200)

```json
{
  "success": true,
  "data": {
    "sn": 1001,
    "activityName": "新春活動",
    "activityCode": "SPRING_2024",
    "activityType": "PROMOTION",
    "description": "2024 新春促銷活動",
    "startDate": "2024-01-01",
    "endDate": "2024-02-29",
    "status": "ACTIVE",
    "targetAudience": "ALL",
    "maxParticipants": 1000,
    "currentParticipants": 256,
    "budgetAmount": 50000.00,
    "isActive": true,
    "createdBy": "admin",
    "createdAt": "2023-12-01T00:00:00Z",
    "updatedBy": "admin",
    "updatedAt": "2024-01-15T12:00:00Z"
  },
  "timestamp": "2024-02-11T10:00:00Z"
}
```

**資料不存在** (HTTP 404)
```json
{
  "success": false,
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "活動配置不存在",
    "details": "SN: 1001 查無資料"
  },
  "timestamp": "2024-02-11T10:00:00Z"
}
```

**參數錯誤** (HTTP 400)
```json
{
  "success": false,
  "error": {
    "code": "INVALID_PARAMETER",
    "message": "sn 必須為正整數",
    "details": "sn: abc 不合法"
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
    "message": "查詢活動配置失敗",
    "details": "資料庫連線異常"
  },
  "timestamp": "2024-02-11T10:00:00Z"
}
```

## DTO 定義

### CfgActivityResponse

| 欄位名稱 | 類型 | 說明 | 範例 |
|---------|------|------|------|
| sn | integer | 活動配置 SN (主鍵) | 1001 |
| activityName | string | 活動名稱 | "新春活動" |
| activityCode | string | 活動代碼 | "SPRING_2024" |
| activityType | string | 活動類型 | "PROMOTION" |
| description | string | 活動描述 | "2024 新春促銷活動" |
| startDate | string | 開始日期 (YYYY-MM-DD) | "2024-01-01" |
| endDate | string | 結束日期 (YYYY-MM-DD) | "2024-02-29" |
| status | string | 活動狀態 | "ACTIVE" |
| targetAudience | string | 目標受眾 | "ALL" |
| maxParticipants | integer | 最大參與人數 | 1000 |
| currentParticipants | integer | 當前參與人數 | 256 |
| budgetAmount | decimal | 預算金額 | 50000.00 |
| isActive | boolean | 是否啟用 | true |
| createdBy | string | 建立者 | "admin" |
| createdAt | string | 建立時間 (ISO 8601) | "2023-12-01T00:00:00Z" |
| updatedBy | string | 更新者 | "admin" |
| updatedAt | string | 更新時間 (ISO 8601) | "2024-01-15T12:00:00Z" |

### ActivityType 列舉值

| 值 | 說明 |
|----|------|
| PROMOTION | 促銷活動 |
| REWARD | 獎勵活動 |
| EVENT | 事件活動 |
| CAMPAIGN | 行銷活動 |

### Status 列舉值

| 值 | 說明 |
|----|------|
| DRAFT | 草稿 |
| ACTIVE | 進行中 |
| PAUSED | 暫停 |
| ENDED | 已結束 |
| CANCELLED | 已取消 |

## Mapper 方法對應

### 參數還原對應表

| 原始 Mapper 方法 | 對應 API 呼叫 | 說明 |
|-----------------|--------------|------|
| `selectAll()` | `GET /cfg-activities` | 查詢全部活動配置 |
| `selectByPrimaryKey(Integer sn)` | `GET /cfg-activities/{sn}` | 傳入 sn 路徑參數查詢單筆 |

### 程式碼遷移範例

**遷移前 (Mapper)**:
```java
// 查詢全部
List<CfgActivityEntity> all = cfgActivityMapper.selectAll();

// 查詢單筆
CfgActivityEntity activity = cfgActivityMapper.selectByPrimaryKey(1001);
```

**遷移後 (API Client)**:
```java
// 查詢全部
List<CfgActivityResponse> all = apiClient.getCfgActivities();

// 查詢單筆
CfgActivityResponse activity = apiClient.getCfgActivityById(1001);
```

## 使用範例

### 範例 1: 查詢全部活動配置

**請求**:
```http
GET /api/v1/cfg-activities HTTP/1.1
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
      "sn": 1001,
      "activityName": "新春活動",
      "activityCode": "SPRING_2024",
      "activityType": "PROMOTION",
      "description": "2024 新春促銷活動",
      "startDate": "2024-01-01",
      "endDate": "2024-02-29",
      "status": "ACTIVE",
      "targetAudience": "ALL",
      "maxParticipants": 1000,
      "currentParticipants": 256,
      "budgetAmount": 50000.00,
      "isActive": true,
      "createdBy": "admin",
      "createdAt": "2023-12-01T00:00:00Z",
      "updatedBy": "admin",
      "updatedAt": "2024-01-15T12:00:00Z"
    }
  ],
  "total": 1,
  "timestamp": "2024-02-11T10:00:00Z"
}
```

### 範例 2: 查詢單一活動配置

**請求**:
```http
GET /api/v1/cfg-activities/1001 HTTP/1.1
Host: api.example.com
Accept: application/json
Authorization: Bearer {access_token}
```

**回應**:
```json
{
  "success": true,
  "data": {
    "sn": 1001,
    "activityName": "新春活動",
    "activityCode": "SPRING_2024",
    "activityType": "PROMOTION",
    "description": "2024 新春促銷活動",
    "startDate": "2024-01-01",
    "endDate": "2024-02-29",
    "status": "ACTIVE",
    "targetAudience": "ALL",
    "maxParticipants": 1000,
    "currentParticipants": 256,
    "budgetAmount": 50000.00,
    "isActive": true,
    "createdBy": "admin",
    "createdAt": "2023-12-01T00:00:00Z",
    "updatedBy": "admin",
    "updatedAt": "2024-01-15T12:00:00Z"
  },
  "timestamp": "2024-02-11T10:00:00Z"
}
```

## 注意事項

1. **RESTful 設計**: 採用 RESTful 慣例，列表與單筆查詢使用不同端點
2. **日期有效性**: 注意檢查 startDate 和 endDate 的有效期間
3. **參與人數**: currentParticipants 應小於等於 maxParticipants
4. **快取策略**: 
   - 列表查詢建議快取 5-10 分鐘
   - 單筆查詢建議快取 15-30 分鐘
5. **資料關聯**: 活動配置與活動規則 (activity-rule-heads)、產品活動 (cfg-product-activities)、合約活動 (cfg-contract-activities) 有關聯
6. **效能考量**: 列表可能資料量較大，建議前端實作分頁或虛擬滾動

## 相關資源

- Mapper 對應文件: [_mappers/CfgActivityMapper.md](_mappers/CfgActivityMapper.md)
- 相關 API: [activity-rule-heads.md](activity-rule-heads.md)
- 相關 API: [cfg-product-activities.md](cfg-product-activities.md)
- 相關 API: [cfg-contract-activities.md](cfg-contract-activities.md)
