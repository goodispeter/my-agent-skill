# Configuration Products API

## 概述

此 API 提供產品配置的查詢功能，包含產品的基本資訊、設定與狀態。

**對應 Mapper**: `CfgProductMapper`  
**合併方法數**: 2 個方法 → 2 個 API 端點（RESTful 慣例）

## API 端點

### 1. 查詢全部產品配置

```
GET /api/v1/cfg-products
```

查詢所有產品配置列表。

#### 請求參數

無

#### 回應格式

**成功回應** (HTTP 200)

```json
{
  "success": true,
  "data": [
    {
      "productSn": 1001,
      "productName": "一對一英語課程",
      "productCode": "ENGL_1on1",
      "productType": "ONLINE_LESSON",
      "description": "專業外師一對一線上英語課程",
      "status": "ACTIVE",
      "price": 500.00,
      "currency": "TWD",
      "duration": 45,
      "durationUnit": "MINUTES",
      "isActive": true,
      "createdAt": "2024-01-01T00:00:00Z",
      "updatedAt": "2024-01-15T12:00:00Z"
    }
  ],
  "total": 1,
  "timestamp": "2024-02-11T10:00:00Z"
}
```

### 2. 查詢單一產品配置

```
GET /api/v1/cfg-products/{productSn}
```

根據產品 SN 查詢單筆產品配置資料。

#### 路徑參數

| 參數名稱 | 類型 | 必填 | 說明 | 範例 |
|---------|------|------|------|------|
| productSn | integer | 是 | 產品 SN (主鍵) | 1001 |

#### 回應格式

**成功回應** (HTTP 200)

```json
{
  "success": true,
  "data": {
    "productSn": 1001,
    "productName": "一對一英語課程",
    "productCode": "ENGL_1on1",
    "productType": "ONLINE_LESSON",
    "description": "專業外師一對一線上英語課程",
    "status": "ACTIVE",
    "price": 500.00,
    "currency": "TWD",
    "duration": 45,
    "durationUnit": "MINUTES",
    "isActive": true,
    "createdAt": "2024-01-01T00:00:00Z",
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
    "message": "產品配置不存在",
    "details": "ProductSn: 1001 查無資料"
  },
  "timestamp": "2024-02-11T10:00:00Z"
}
```

## DTO 定義

### CfgProductResponse

| 欄位名稱 | 類型 | 說明 | 範例 |
|---------|------|------|------|
| productSn | integer | 產品 SN (主鍵) | 1001 |
| productName | string | 產品名稱 | "一對一英語課程" |
| productCode | string | 產品代碼 | "ENGL_1on1" |
| productType | string | 產品類型 | "ONLINE_LESSON" |
| description | string | 產品描述 | "專業外師一對一線上英語課程" |
| status | string | 產品狀態 | "ACTIVE" |
| price | decimal | 產品價格 | 500.00 |
| currency | string | 幣別 | "TWD" |
| duration | integer | 時長 | 45 |
| durationUnit | string | 時長單位 | "MINUTES" |
| isActive | boolean | 是否啟用 | true |
| createdAt | string | 建立時間 (ISO 8601) | "2024-01-01T00:00:00Z" |
| updatedAt | string | 更新時間 (ISO 8601) | "2024-01-15T12:00:00Z" |

### ProductType 列舉值

| 值 | 說明 |
|----|------|
| ONLINE_LESSON | 線上課程 |
| OFFLINE_LESSON | 實體課程 |
| PACKAGE | 課程套餐 |
| MATERIAL | 教材 |

## Mapper 方法對應

### 參數還原對應表

| 原始 Mapper 方法 | 對應 API 呼叫 | 說明 |
|-----------------|--------------|------|
| `selectAllMuchNew()` | `GET /cfg-products` | 查詢全部產品配置 |
| `selectByProductSn(int productSn)` | `GET /cfg-products/{productSn}` | 傳入 productSn 路徑參數查詢單筆 |

### 程式碼遷移範例

**遷移前 (Mapper)**:
```java
// 查詢全部
List<CfgProductEntity> all = cfgProductMapper.selectAllMuchNew();

// 查詢單筆
CfgProductEntity product = cfgProductMapper.selectByProductSn(1001);
```

**遷移後 (API Client)**:
```java
// 查詢全部
List<CfgProductResponse> all = apiClient.getCfgProducts();

// 查詢單筆
CfgProductResponse product = apiClient.getCfgProductByProductSn(1001);
```

## 使用範例

### 範例 1: 查詢全部產品配置

**請求**:
```http
GET /api/v1/cfg-products HTTP/1.1
Host: api.example.com
Accept: application/json
Authorization: Bearer {access_token}
```

### 範例 2: 查詢單一產品配置

**請求**:
```http
GET /api/v1/cfg-products/1001 HTTP/1.1
Host: api.example.com
Accept: application/json
Authorization: Bearer {access_token}
```

## 注意事項

1. **RESTful 設計**: 採用 RESTful 慣例，列表與單筆查詢使用不同端點
2. **快取策略**: 
   - 列表查詢建議快取 10-15 分鐘
   - 單筆查詢建議快取 15-30 分鐘
3. **資料關聯**: 產品配置與產品活動 (cfg-product-activities) 有關聯
4. **效能考量**: 列表可能資料量較大，建議實作分頁功能

## 相關資源

- Mapper 對應文件: [_mappers/CfgProductMapper.md](_mappers/CfgProductMapper.md)
- 相關 API: [cfg-product-activities.md](cfg-product-activities.md)
