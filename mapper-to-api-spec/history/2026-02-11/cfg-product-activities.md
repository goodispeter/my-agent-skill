# Configuration Product Activities API

## 概述

此 API 提供產品活動配置的查詢功能，管理產品與活動的關聯關係。

**對應 Mapper**: `CfgProductActivityMapper`  
**方法數**: 1 個方法

## API 端點

### 條件查詢產品活動配置

```
POST /api/v1/cfg-product-activities/query
```

根據多種條件查詢產品活動配置，支援複雜的篩選條件組合。

#### 請求參數

**Request Body** (JSON):
```json
{
  "productSn": 1001,
  "activitySn": 2001,
  "status": "ACTIVE",
  "isActive": true,
  "startDateFrom": "2024-01-01",
  "startDateTo": "2024-12-31"
}
```

| 欄位名稱 | 類型 | 必填 | 說明 | 範例 |
|---------|------|------|------|------|
| productSn | integer | 否 | 產品 SN 篩選 | 1001 |
| activitySn | integer | 否 | 活動 SN 篩選 | 2001 |
| status | string | 否 | 狀態篩選 | "ACTIVE" |
| isActive | boolean | 否 | 是否啟用篩選 | true |
| startDateFrom | string | 否 | 開始日期起始 (YYYY-MM-DD) | "2024-01-01" |
| startDateTo | string | 否 | 開始日期結束 (YYYY-MM-DD) | "2024-12-31" |

#### 回應格式

**成功回應** (HTTP 200)

```json
{
  "success": true,
  "data": [
    {
      "sn": 1,
      "productSn": 1001,
      "productName": "一對一英語課程",
      "activitySn": 2001,
      "activityName": "新春活動",
      "activityCode": "SPRING_2024",
      "startDate": "2024-01-01",
      "endDate": "2024-02-29",
      "discountRate": 0.85,
      "status": "ACTIVE",
      "priority": 10,
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
    "message": "日期格式不正確",
    "details": "startDateFrom 應為 YYYY-MM-DD 格式"
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
    "message": "查詢產品活動配置失敗",
    "details": "資料庫連線異常"
  },
  "timestamp": "2024-02-11T10:00:00Z"
}
```

## DTO 定義

### CfgProductActivityResponse

| 欄位名稱 | 類型 | 說明 | 範例 |
|---------|------|------|------|
| sn | integer | 配置 SN (主鍵) | 1 |
| productSn | integer | 產品 SN (外鍵) | 1001 |
| productName | string | 產品名稱 | "一對一英語課程" |
| activitySn | integer | 活動 SN (外鍵) | 2001 |
| activityName | string | 活動名稱 | "新春活動" |
| activityCode | string | 活動代碼 | "SPRING_2024" |
| startDate | string | 開始日期 (YYYY-MM-DD) | "2024-01-01" |
| endDate | string | 結束日期 (YYYY-MM-DD) | "2024-02-29" |
| discountRate | decimal | 折扣率 (0.85 表示 85折) | 0.85 |
| status | string | 狀態 | "ACTIVE" |
| priority | integer | 優先順序 | 10 |
| isActive | boolean | 是否啟用 | true |
| createdAt | string | 建立時間 (ISO 8601) | "2024-01-01T00:00:00Z" |
| updatedAt | string | 更新時間 (ISO 8601) | "2024-01-15T12:00:00Z" |

### Status 列舉值

| 值 | 說明 |
|----|------|
| ACTIVE | 進行中 |
| PENDING | 待開始 |
| COMPLETED | 已完成 |
| CANCELLED | 已取消 |

## Mapper 方法對應

### 參數還原對應表

| 原始 Mapper 方法 | 對應 API 呼叫 | 說明 |
|-----------------|--------------|------|
| `selectByExample(CfgProductActivityEntityExample example)` | `POST /cfg-product-activities/query` | Request Body 傳入查詢條件物件 |

### 程式碼遷移範例

**遷移前 (Mapper)**:
```java
// 建立 Example 查詢條件
CfgProductActivityEntityExample example = new CfgProductActivityEntityExample();
CfgProductActivityEntityExample.Criteria criteria = example.createCriteria();
criteria.andProductSnEqualTo(1001);
criteria.andActivitySnEqualTo(2001);
criteria.andStatusEqualTo("ACTIVE");

// 執行查詢
List<CfgProductActivityEntity> activities = 
    cfgProductActivityMapper.selectByExample(example);
```

**遷移後 (API Client)**:
```java
// 建立查詢條件物件
CfgProductActivityQueryRequest request = CfgProductActivityQueryRequest.builder()
    .productSn(1001)
    .activitySn(2001)
    .status("ACTIVE")
    .build();

// 執行查詢
List<CfgProductActivityResponse> activities = 
    apiClient.queryCfgProductActivities(request);
```

## 使用範例

### 範例 1: 查詢特定產品的所有活動

**請求**:
```http
POST /api/v1/cfg-product-activities/query HTTP/1.1
Host: api.example.com
Content-Type: application/json
Accept: application/json
Authorization: Bearer {access_token}

{
  "productSn": 1001
}
```

**回應**:
```json
{
  "success": true,
  "data": [
    {
      "sn": 1,
      "productSn": 1001,
      "productName": "一對一英語課程",
      "activitySn": 2001,
      "activityName": "新春活動",
      "activityCode": "SPRING_2024",
      "startDate": "2024-01-01",
      "endDate": "2024-02-29",
      "discountRate": 0.85,
      "status": "ACTIVE",
      "priority": 10,
      "isActive": true,
      "createdAt": "2024-01-01T00:00:00Z",
      "updatedAt": "2024-01-15T12:00:00Z"
    },
    {
      "sn": 2,
      "productSn": 1001,
      "productName": "一對一英語課程",
      "activitySn": 2002,
      "activityName": "夏季促銷",
      "activityCode": "SUMMER_2024",
      "startDate": "2024-06-01",
      "endDate": "2024-08-31",
      "discountRate": 0.90,
      "status": "PENDING",
      "priority": 5,
      "isActive": true,
      "createdAt": "2024-01-10T00:00:00Z",
      "updatedAt": "2024-01-15T12:00:00Z"
    }
  ],
  "total": 2,
  "timestamp": "2024-02-11T10:00:00Z"
}
```

### 範例 2: 組合條件查詢

**請求**:
```http
POST /api/v1/cfg-product-activities/query HTTP/1.1
Host: api.example.com
Content-Type: application/json
Accept: application/json
Authorization: Bearer {access_token}

{
  "productSn": 1001,
  "status": "ACTIVE",
  "isActive": true,
  "startDateFrom": "2024-01-01",
  "startDateTo": "2024-03-31"
}
```

**回應**:
```json
{
  "success": true,
  "data": [
    {
      "sn": 1,
      "productSn": 1001,
      "productName": "一對一英語課程",
      "activitySn": 2001,
      "activityName": "新春活動",
      "activityCode": "SPRING_2024",
      "startDate": "2024-01-01",
      "endDate": "2024-02-29",
      "discountRate": 0.85,
      "status": "ACTIVE",
      "priority": 10,
      "isActive": true,
      "createdAt": "2024-01-01T00:00:00Z",
      "updatedAt": "2024-01-15T12:00:00Z"
    }
  ],
  "total": 1,
  "timestamp": "2024-02-11T10:00:00Z"
}
```

### 範例 3: 查詢所有配置（不傳任何條件）

**請求**:
```http
POST /api/v1/cfg-product-activities/query HTTP/1.1
Host: api.example.com
Content-Type: application/json
Accept: application/json
Authorization: Bearer {access_token}

{}
```

## 注意事項

1. **彈性查詢**: 支援多種條件組合，所有參數皆為可選
2. **空條件查詢**: 不傳任何條件則返回全部資料（建議限制返回筆數或實作分頁）
3. **日期範圍**: startDateFrom 和 startDateTo 可組合使用進行日期範圍篩選
4. **效能考量**: 複雜條件查詢可能較慢，建議實作查詢結果快取
5. **快取策略**: 產品活動配置相對穩定，建議快取 10-15 分鐘
6. **資料關聯**: 產品活動配置連接產品與活動兩個實體
7. **分頁建議**: 建議實作分頁功能，避免一次返回過多資料

## 相關資源

- Mapper 對應文件: [_mappers/CfgProductActivityMapper.md](_mappers/CfgProductActivityMapper.md)
- 相關 API: [cfg-products.md](cfg-products.md)
- 相關 API: [cfg-activities.md](cfg-activities.md)
