# Configuration Contract Activities API

## 概述

此 API 提供合約活動配置的查詢功能，管理合約與活動的關聯關係。

**對應 Mapper**: `CfgContractActivityMapper`  
**方法數**: 1 個方法

## API 端點

### 批次查詢合約活動配置

```
POST /api/v1/cfg-contract-activities/batch-query
```

批次查詢多個合約的活動配置資訊。

#### 請求參數

**Request Body** (JSON):
```json
{
  "contractSns": [1001, 1002, 1003]
}
```

| 欄位名稱 | 類型 | 必填 | 說明 | 範例 |
|---------|------|------|------|------|
| contractSns | array[integer] | 是 | 合約 SN 陣列 | [1001, 1002, 1003] |

#### 回應格式

**成功回應** (HTTP 200)

```json
{
  "success": true,
  "data": [
    {
      "sn": 1,
      "contractSn": 1001,
      "activitySn": 2001,
      "activityName": "新春活動",
      "activityCode": "SPRING_2024",
      "joinDate": "2024-01-01",
      "status": "ACTIVE",
      "priority": 10,
      "isActive": true,
      "createdAt": "2024-01-01T00:00:00Z",
      "updatedAt": "2024-01-15T12:00:00Z"
    },
    {
      "sn": 2,
      "contractSn": 1002,
      "activitySn": 2001,
      "activityName": "新春活動",
      "activityCode": "SPRING_2024",
      "joinDate": "2024-01-05",
      "status": "ACTIVE",
      "priority": 10,
      "isActive": true,
      "createdAt": "2024-01-05T00:00:00Z",
      "updatedAt": "2024-01-15T12:00:00Z"
    }
  ],
  "total": 2,
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
    "message": "缺少必填參數 contractSns",
    "details": "contractSns 不能為空"
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
    "message": "contractSns 陣列不能為空",
    "details": "至少需要提供一個合約 SN"
  },
  "timestamp": "2024-02-11T10:00:00Z"
}
```

## DTO 定義

### CfgContractActivityResponse

| 欄位名稱 | 類型 | 說明 | 範例 |
|---------|------|------|------|
| sn | integer | 配置 SN (主鍵) | 1 |
| contractSn | integer | 合約 SN (外鍵) | 1001 |
| activitySn | integer | 活動 SN (外鍵) | 2001 |
| activityName | string | 活動名稱 | "新春活動" |
| activityCode | string | 活動代碼 | "SPRING_2024" |
| joinDate | string | 加入日期 (YYYY-MM-DD) | "2024-01-01" |
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
| `selectByContractSns(List contractSns)` | `POST /cfg-contract-activities/batch-query` | Request Body 傳入 contractSns 陣列 |

### 程式碼遷移範例

**遷移前 (Mapper)**:
```java
// 批次查詢合約活動配置
List<Integer> contractSns = Arrays.asList(1001, 1002, 1003);
List<CfgContractActivityEntity> activities = 
    cfgContractActivityMapper.selectByContractSns(contractSns);
```

**遷移後 (API Client)**:
```java
// 批次查詢合約活動配置
List<Integer> contractSns = Arrays.asList(1001, 1002, 1003);
List<CfgContractActivityResponse> activities = 
    apiClient.batchQueryCfgContractActivities(contractSns);
```

## 使用範例

### 範例 1: 批次查詢合約活動配置

**請求**:
```http
POST /api/v1/cfg-contract-activities/batch-query HTTP/1.1
Host: api.example.com
Content-Type: application/json
Accept: application/json
Authorization: Bearer {access_token}

{
  "contractSns": [1001, 1002, 1003]
}
```

**回應**:
```json
{
  "success": true,
  "data": [
    {
      "sn": 1,
      "contractSn": 1001,
      "activitySn": 2001,
      "activityName": "新春活動",
      "activityCode": "SPRING_2024",
      "joinDate": "2024-01-01",
      "status": "ACTIVE",
      "priority": 10,
      "isActive": true,
      "createdAt": "2024-01-01T00:00:00Z",
      "updatedAt": "2024-01-15T12:00:00Z"
    },
    {
      "sn": 2,
      "contractSn": 1002,
      "activitySn": 2001,
      "activityName": "新春活動",
      "activityCode": "SPRING_2024",
      "joinDate": "2024-01-05",
      "status": "ACTIVE",
      "priority": 10,
      "isActive": true,
      "createdAt": "2024-01-05T00:00:00Z",
      "updatedAt": "2024-01-15T12:00:00Z"
    },
    {
      "sn": 3,
      "contractSn": 1003,
      "activitySn": 2002,
      "activityName": "夏季促銷",
      "activityCode": "SUMMER_2024",
      "joinDate": "2024-06-01",
      "status": "PENDING",
      "priority": 5,
      "isActive": true,
      "createdAt": "2024-01-10T00:00:00Z",
      "updatedAt": "2024-01-15T12:00:00Z"
    }
  ],
  "total": 3,
  "timestamp": "2024-02-11T10:00:00Z"
}
```

## 注意事項

1. **批次查詢**: 使用 POST 方法進行批次查詢，避免 GET URL 長度限制
2. **陣列大小限制**: 建議限制 contractSns 陣列大小（例如：最多 100 個）
3. **空結果**: 若某些合約沒有關聯活動，仍會返回 200，但該合約不會出現在結果中
4. **效能考量**: 批次查詢可減少 API 調用次數，但單次查詢不宜過大
5. **快取策略**: 合約活動配置相對穩定，建議快取 10-15 分鐘
6. **資料關聯**: 合約活動配置與活動配置 (cfg-activities)、臨時合約有關聯

## 相關資源

- Mapper 對應文件: [_mappers/CfgContractActivityMapper.md](_mappers/CfgContractActivityMapper.md)
- 相關 API: [cfg-activities.md](cfg-activities.md)
- 相關 API: [client-temporal-contracts.md](client-temporal-contracts.md)
