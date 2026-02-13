# Contract Change Records API

## 概述

此 API 提供合約變更記錄的查詢功能，記錄合約的所有變更歷史。

**對應 Mapper**: `ContractChangeRecordMapper`  
**方法數**: 1 個方法

## API 端點

### 查詢合約變更記錄

```
GET /api/v1/contract-change-records
```

查詢合約變更記錄列表，支援依帳戶 SN 與變更類型篩選。

#### 請求參數

| 參數名稱 | 類型 | 位置 | 必填 | 說明 | 範例 |
|---------|------|------|------|------|------|
| accountSn | integer | query | 是 | 帳戶 SN | 12345 |
| changeTypeSn | integer | query | 否 | 變更類型 SN | 1 |

#### 回應格式

**成功回應** (HTTP 200)

```json
{
  "success": true,
  "data": [
    {
      "sn": 1,
      "accountSn": 12345,
      "contractSn": 9876,
      "changeTypeSn": 1,
      "changeTypeName": "新增堂數",
      "changeDate": "2024-01-15",
      "beforeValue": "10",
      "afterValue": "15",
      "changeAmount": 5,
      "reason": "購買額外課程",
      "operatorId": "admin001",
      "operatorName": "Admin User",
      "remarks": "客戶要求增加課程堂數",
      "createdAt": "2024-01-15T10:00:00Z",
      "updatedAt": "2024-01-15T10:00:00Z"
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
    "message": "缺少必填參數 accountSn",
    "details": "accountSn 為必填參數"
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
    "message": "accountSn 必須為正整數",
    "details": "accountSn: -1 不合法"
  },
  "timestamp": "2024-02-11T10:00:00Z"
}
```

## DTO 定義

### ContractChangeRecordResponse

| 欄位名稱 | 類型 | 說明 | 範例 |
|---------|------|------|------|
| sn | integer | 記錄 SN (主鍵) | 1 |
| accountSn | integer | 帳戶 SN (外鍵) | 12345 |
| contractSn | integer | 合約 SN (外鍵) | 9876 |
| changeTypeSn | integer | 變更類型 SN | 1 |
| changeTypeName | string | 變更類型名稱 | "新增堂數" |
| changeDate | string | 變更日期 (YYYY-MM-DD) | "2024-01-15" |
| beforeValue | string | 變更前數值 | "10" |
| afterValue | string | 變更後數值 | "15" |
| changeAmount | integer | 變更數量 | 5 |
| reason | string | 變更原因 | "購買額外課程" |
| operatorId | string | 操作者 ID | "admin001" |
| operatorName | string | 操作者姓名 | "Admin User" |
| remarks | string | 備註 | "客戶要求增加課程堂數" |
| createdAt | string | 建立時間 (ISO 8601) | "2024-01-15T10:00:00Z" |
| updatedAt | string | 更新時間 (ISO 8601) | "2024-01-15T10:00:00Z" |

### ChangeType 常見類型

| SN | 類型名稱 |
|----|---------|
| 1 | 新增堂數 |
| 2 | 扣除堂數 |
| 3 | 延長期限 |
| 4 | 變更產品 |
| 5 | 合約暫停 |
| 6 | 合約恢復 |

## Mapper 方法對應

### 參數還原對應表

| 原始 Mapper 方法 | 對應 API 呼叫 | 說明 |
|-----------------|--------------|------|
| `selectByAccountSn(Integer accountSn, Integer changeTypeSn)` | `GET /contract-change-records?accountSn={accountSn}&changeTypeSn={changeTypeSn}` | accountSn 必填，changeTypeSn 可選 |

### 程式碼遷移範例

**遷移前 (Mapper)**:
```java
// 查詢帳戶所有變更記錄
List<ContractChangeRecordEntity> allRecords = 
    contractChangeRecordMapper.selectByAccountSn(12345, null);

// 查詢特定類型的變更記錄
List<ContractChangeRecordEntity> typeRecords = 
    contractChangeRecordMapper.selectByAccountSn(12345, 1);
```

**遷移後 (API Client)**:
```java
// 查詢帳戶所有變更記錄
List<ContractChangeRecordResponse> allRecords = 
    apiClient.getContractChangeRecords(12345, null);

// 查詢特定類型的變更記錄
List<ContractChangeRecordResponse> typeRecords = 
    apiClient.getContractChangeRecords(12345, 1);
```

## 使用範例

### 範例 1: 查詢帳戶所有變更記錄

**請求**:
```http
GET /api/v1/contract-change-records?accountSn=12345 HTTP/1.1
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
      "accountSn": 12345,
      "contractSn": 9876,
      "changeTypeSn": 1,
      "changeTypeName": "新增堂數",
      "changeDate": "2024-01-15",
      "beforeValue": "10",
      "afterValue": "15",
      "changeAmount": 5,
      "reason": "購買額外課程",
      "operatorId": "admin001",
      "operatorName": "Admin User",
      "remarks": "客戶要求增加課程堂數",
      "createdAt": "2024-01-15T10:00:00Z",
      "updatedAt": "2024-01-15T10:00:00Z"
    },
    {
      "sn": 2,
      "accountSn": 12345,
      "contractSn": 9876,
      "changeTypeSn": 3,
      "changeTypeName": "延長期限",
      "changeDate": "2024-02-01",
      "beforeValue": "2024-01-31",
      "afterValue": "2024-02-28",
      "changeAmount": 0,
      "reason": "客戶申請延期",
      "operatorId": "admin002",
      "operatorName": "Another Admin",
      "remarks": "因故無法按時完成課程",
      "createdAt": "2024-02-01T09:00:00Z",
      "updatedAt": "2024-02-01T09:00:00Z"
    }
  ],
  "total": 2,
  "timestamp": "2024-02-11T10:00:00Z"
}
```

### 範例 2: 查詢特定類型的變更記錄

**請求**:
```http
GET /api/v1/contract-change-records?accountSn=12345&changeTypeSn=1 HTTP/1.1
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
      "accountSn": 12345,
      "contractSn": 9876,
      "changeTypeSn": 1,
      "changeTypeName": "新增堂數",
      "changeDate": "2024-01-15",
      "beforeValue": "10",
      "afterValue": "15",
      "changeAmount": 5,
      "reason": "購買額外課程",
      "operatorId": "admin001",
      "operatorName": "Admin User",
      "remarks": "客戶要求增加課程堂數",
      "createdAt": "2024-01-15T10:00:00Z",
      "updatedAt": "2024-01-15T10:00:00Z"
    }
  ],
  "total": 1,
  "timestamp": "2024-02-11T10:00:00Z"
}
```

## 注意事項

1. **必填參數**: accountSn 為必填參數，changeTypeSn 為可選參數
2. **稽核用途**: 變更記錄具有稽核價值，不應允許刪除或修改
3. **資料量**: 變更記錄可能很多，建議實作分頁或日期範圍篩選
4. **快取策略**: 變更記錄相對穩定（只增不改），建議快取 10-15 分鐘
5. **排序建議**: 建議依變更日期倒序排列（最新在前）
6. **資料關聯**: 變更記錄與臨時合約、帳戶資料有關聯

## 相關資源

- Mapper 對應文件: [_mappers/ContractChangeRecordMapper.md](_mappers/ContractChangeRecordMapper.md)
- 相關 API: [client-temporal-contracts.md](client-temporal-contracts.md)
