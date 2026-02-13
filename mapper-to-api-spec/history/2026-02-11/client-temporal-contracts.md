# Client Temporal Contracts API

## 概述

此 API 提供客戶臨時合約的查詢功能，包含合約資訊、狀態與期限等。

**對應 Mapper**: `ClientTemporalContractMapper`  
**合併方法數**: 2 個方法 → 2 個 API 端點（RESTful 慣例）

## API 端點

### 1. 查詢客戶臨時合約列表

```
GET /api/v1/client-temporal-contracts
```

查詢客戶臨時合約列表，依客戶 SN 篩選。

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
      "contractNo": "TEMP202401001",
      "contractType": "TRIAL",
      "startDate": "2024-01-01",
      "endDate": "2024-01-31",
      "status": "ACTIVE",
      "totalLessons": 10,
      "usedLessons": 3,
      "remainingLessons": 7,
      "isActive": true,
      "createdAt": "2024-01-01T00:00:00Z",
      "updatedAt": "2024-01-15T12:00:00Z"
    }
  ],
  "total": 1,
  "timestamp": "2024-02-11T10:00:00Z"
}
```

### 2. 查詢單一臨時合約

```
GET /api/v1/client-temporal-contracts/{sn}
```

根據合約 SN 查詢單筆臨時合約資料。

#### 路徑參數

| 參數名稱 | 類型 | 必填 | 說明 | 範例 |
|---------|------|------|------|------|
| sn | integer | 是 | 合約 SN (主鍵) | 1 |

#### 回應格式

**成功回應** (HTTP 200)

```json
{
  "success": true,
  "data": {
    "sn": 1,
    "clientSn": 12345,
    "contractNo": "TEMP202401001",
    "contractType": "TRIAL",
    "startDate": "2024-01-01",
    "endDate": "2024-01-31",
    "status": "ACTIVE",
    "totalLessons": 10,
    "usedLessons": 3,
    "remainingLessons": 7,
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
    "message": "臨時合約不存在",
    "details": "SN: 1 查無資料"
  },
  "timestamp": "2024-02-11T10:00:00Z"
}
```

## DTO 定義

### ClientTemporalContractResponse

| 欄位名稱 | 類型 | 說明 | 範例 |
|---------|------|------|------|
| sn | integer | 合約 SN (主鍵) | 1 |
| clientSn | integer | 客戶 SN (外鍵) | 12345 |
| contractNo | string | 合約編號 | "TEMP202401001" |
| contractType | string | 合約類型 | "TRIAL" |
| startDate | string | 開始日期 (YYYY-MM-DD) | "2024-01-01" |
| endDate | string | 結束日期 (YYYY-MM-DD) | "2024-01-31" |
| status | string | 合約狀態 | "ACTIVE" |
| totalLessons | integer | 總堂數 | 10 |
| usedLessons | integer | 已使用堂數 | 3 |
| remainingLessons | integer | 剩餘堂數 | 7 |
| isActive | boolean | 是否啟用 | true |
| createdAt | string | 建立時間 (ISO 8601) | "2024-01-01T00:00:00Z" |
| updatedAt | string | 更新時間 (ISO 8601) | "2024-01-15T12:00:00Z" |

### ContractType 列舉值

| 值 | 說明 |
|----|------|
| TRIAL | 試用合約 |
| TEMPORARY | 臨時合約 |
| EXTENSION | 延展合約 |

### Status 列舉值

| 值 | 說明 |
|----|------|
| ACTIVE | 進行中 |
| EXPIRED | 已過期 |
| CANCELLED | 已取消 |
| COMPLETED | 已完成 |

## Mapper 方法對應

### 參數還原對應表

| 原始 Mapper 方法 | 對應 API 呼叫 | 說明 |
|-----------------|--------------|------|
| `selectByClientSn(Integer clientSn)` | `GET /client-temporal-contracts?clientSn={clientSn}` | 傳入 clientSn 參數查詢列表 |
| `selectByPrimaryKey(Integer sn)` | `GET /client-temporal-contracts/{sn}` | 傳入 sn 路徑參數查詢單筆 |

### 程式碼遷移範例

**遷移前 (Mapper)**:
```java
// 依客戶查詢合約列表
List<ClientTemporalContractEntity> contracts = 
    clientTemporalContractMapper.selectByClientSn(12345);

// 查詢單一合約
ClientTemporalContractEntity contract = 
    clientTemporalContractMapper.selectByPrimaryKey(1);
```

**遷移後 (API Client)**:
```java
// 依客戶查詢合約列表
List<ClientTemporalContractResponse> contracts = 
    apiClient.getClientTemporalContracts(12345);

// 查詢單一合約
ClientTemporalContractResponse contract = 
    apiClient.getClientTemporalContractById(1);
```

## 使用範例

### 範例 1: 查詢客戶臨時合約列表

**請求**:
```http
GET /api/v1/client-temporal-contracts?clientSn=12345 HTTP/1.1
Host: api.example.com
Accept: application/json
Authorization: Bearer {access_token}
```

### 範例 2: 查詢單一臨時合約

**請求**:
```http
GET /api/v1/client-temporal-contracts/1 HTTP/1.1
Host: api.example.com
Accept: application/json
Authorization: Bearer {access_token}
```

## 注意事項

1. **RESTful 設計**: 採用 RESTful 慣例，列表與單筆查詢使用不同端點
2. **必填參數**: 列表查詢中 clientSn 為必填參數
3. **日期有效性**: 注意檢查 startDate 和 endDate 的有效期間
4. **堂數計算**: remainingLessons = totalLessons - usedLessons
5. **快取策略**: 
   - 列表查詢建議快取 3-5 分鐘（合約狀態可能頻繁變動）
   - 單筆查詢建議快取 5-10 分鐘
6. **資料關聯**: 臨時合約與客戶資料、合約變更記錄有關聯

## 相關資源

- Mapper 對應文件: [_mappers/ClientTemporalContractMapper.md](_mappers/ClientTemporalContractMapper.md)
- 相關 API: [contract-change-records.md](contract-change-records.md)
