# Client Purchases API

## 概述

此 API 提供客戶購買記錄的查詢功能，包含購買訂單、產品與狀態資訊。

**對應 Mapper**: `ClientPurchaseMapper`  
**方法數**: 1 個方法

## API 端點

### 查詢客戶購買記錄

```
GET /api/v1/client-purchases
```

查詢客戶購買記錄列表，依客戶 SN 篩選。

#### 請求參數

| 參數名稱 | 類型 | 位置 | 必填 | 說明 | 範例 |
|---------|------|------|------|------|------|
| clientSn | integer | query | 是 | 客戶 SN | 12345 |

#### 回應格式

**成功回應** (HTTP 200)

```json
{
  "success": true,
  "data": [
    {
      "sn": 1,
      "clientSn": 12345,
      "orderNo": "ORD20240101001",
      "productSn": 1001,
      "productName": "一對一英語課程",
      "quantity": 10,
      "unitPrice": 500.00,
      "totalAmount": 5000.00,
      "currency": "TWD",
      "purchaseDate": "2024-01-01",
      "status": "COMPLETED",
      "paymentMethod": "CREDIT_CARD",
      "createdAt": "2024-01-01T10:00:00Z",
      "updatedAt": "2024-01-01T10:30:00Z"
    }
  ],
  "total": 1,
  "timestamp": "2024-02-11T10:00:00Z"
}
```

#### 錯誤回應

**缺少必填參數** (HTTP 400)
```json
{
  "success": false,
  "error": {
    "code": "MISSING_REQUIRED_PARAMETER",
    "message": "缺少必填參數 clientSn",
    "details": "clientSn 為必填參數"
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
    "message": "clientSn 必須為正整數",
    "details": "clientSn: -1 不合法"
  },
  "timestamp": "2024-02-11T10:00:00Z"
}
```

## DTO 定義

### ClientPurchaseResponse

| 欄位名稱 | 類型 | 說明 | 範例 |
|---------|------|------|------|
| sn | integer | 購買記錄 SN (主鍵) | 1 |
| clientSn | integer | 客戶 SN (外鍵) | 12345 |
| orderNo | string | 訂單編號 | "ORD20240101001" |
| productSn | integer | 產品 SN (外鍵) | 1001 |
| productName | string | 產品名稱 | "一對一英語課程" |
| quantity | integer | 購買數量 | 10 |
| unitPrice | decimal | 單價 | 500.00 |
| totalAmount | decimal | 總金額 | 5000.00 |
| currency | string | 幣別 | "TWD" |
| purchaseDate | string | 購買日期 (YYYY-MM-DD) | "2024-01-01" |
| status | string | 訂單狀態 | "COMPLETED" |
| paymentMethod | string | 付款方式 | "CREDIT_CARD" |
| createdAt | string | 建立時間 (ISO 8601) | "2024-01-01T10:00:00Z" |
| updatedAt | string | 更新時間 (ISO 8601) | "2024-01-01T10:30:00Z" |

### Status 列舉值

| 值 | 說明 |
|----|------|
| PENDING | 待付款 |
| COMPLETED | 已完成 |
| CANCELLED | 已取消 |
| REFUNDED | 已退款 |

## Mapper 方法對應

### 參數還原對應表

| 原始 Mapper 方法 | 對應 API 呼叫 | 說明 |
|-----------------|--------------|------|
| `selectByClientSn(Integer clientSn)` | `GET /client-purchases?clientSn={clientSn}` | 傳入 clientSn 參數查詢客戶購買記錄 |

### 程式碼遷移範例

**遷移前 (Mapper)**:
```java
// 查詢客戶購買記錄
List<ClientPurchaseEntity> purchases = 
    clientPurchaseMapper.selectByClientSn(12345);
```

**遷移後 (API Client)**:
```java
// 查詢客戶購買記錄
List<ClientPurchaseResponse> purchases = 
    apiClient.getClientPurchases(12345);
```

## 使用範例

### 範例 1: 查詢客戶購買記錄

**請求**:
```http
GET /api/v1/client-purchases?clientSn=12345 HTTP/1.1
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
      "clientSn": 12345,
      "orderNo": "ORD20240101001",
      "productSn": 1001,
      "productName": "一對一英語課程",
      "quantity": 10,
      "unitPrice": 500.00,
      "totalAmount": 5000.00,
      "currency": "TWD",
      "purchaseDate": "2024-01-01",
      "status": "COMPLETED",
      "paymentMethod": "CREDIT_CARD",
      "createdAt": "2024-01-01T10:00:00Z",
      "updatedAt": "2024-01-01T10:30:00Z"
    },
    {
      "sn": 2,
      "clientSn": 12345,
      "orderNo": "ORD20240115002",
      "productSn": 1002,
      "productName": "小班課程套餐",
      "quantity": 20,
      "unitPrice": 300.00,
      "totalAmount": 6000.00,
      "currency": "TWD",
      "purchaseDate": "2024-01-15",
      "status": "COMPLETED",
      "paymentMethod": "ATM",
      "createdAt": "2024-01-15T14:00:00Z",
      "updatedAt": "2024-01-15T14:30:00Z"
    }
  ],
  "total": 2,
  "timestamp": "2024-02-11T10:00:00Z"
}
```

## 注意事項

1. **必填參數**: clientSn 為必填參數，未提供則返回 400 錯誤
2. **資料量**: 購買記錄可能很多，建議實作分頁或日期範圍篩選
3. **快取策略**: 購買記錄相對穩定，建議快取 5-10 分鐘
4. **資料關聯**: 購買記錄與產品配置、客戶資料有關聯
5. **排序建議**: 建議依購買日期倒序排列（最新在前）

## 相關資源

- Mapper 對應文件: [_mappers/ClientPurchaseMapper.md](_mappers/ClientPurchaseMapper.md)
- 相關 API: [cfg-products.md](cfg-products.md)
