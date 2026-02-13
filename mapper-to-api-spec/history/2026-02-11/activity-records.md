# Activity Records API

## 概述

此 API 提供活動記錄的查詢與更新功能，包含記錄查詢與計算結果更新。

**對應 Mapper**: `ActivityRecordMapper`  
**方法數**: 2 個方法（1 個查詢 + 1 個更新）

## API 端點

### 1. 範圍查詢活動記錄

```
GET /api/v1/activity-records
```

依 SN 範圍查詢活動記錄列表。

#### 請求參數

| 參數名稱 | 類型 | 位置 | 必填 | 說明 | 範例 |
|---------|------|------|------|------|------|
| from | integer | query | 是 | 起始 SN（包含） | 1 |
| to | integer | query | 是 | 結束 SN（包含） | 100 |

#### 回應格式

**成功回應** (HTTP 200)

```json
{
  "success": true,
  "data": [
    {
      "sn": 1,
      "clientSn": 12345,
      "activitySn": 2001,
      "activityName": "新春活動",
      "recordType": "PARTICIPATION",
      "recordDate": "2024-01-15",
      "recordValue": "100",
      "calculateResult": "COMPLETED",
      "status": "ACTIVE",
      "remarks": "完成活動參與",
      "createdAt": "2024-01-15T10:00:00Z",
      "updatedAt": "2024-01-15T12:00:00Z"
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
    "message": "缺少必填參數 from 或 to",
    "details": "from 和 to 皆為必填參數"
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
    "message": "from 必須小於等於 to",
    "details": "from: 100, to: 1"
  },
  "timestamp": "2024-02-11T10:00:00Z"
}
```

### 2. 批次更新計算結果

```
PUT /api/v1/activity-records/calculate-results
```

批次更新活動記錄的計算結果。

#### 請求參數

**Request Body** (JSON):
```json
{
  "records": [
    {
      "sn": 1,
      "calculateResult": "COMPLETED"
    },
    {
      "sn": 2,
      "calculateResult": "FAILED"
    }
  ]
}
```

| 欄位名稱 | 類型 | 必填 | 說明 | 範例 |
|---------|------|------|------|------|
| records | array | 是 | 更新記錄陣列 | - |
| records[].sn | integer | 是 | 記錄 SN | 1 |
| records[].calculateResult | string | 是 | 計算結果 | "COMPLETED" |

#### 回應格式

**成功回應** (HTTP 200)

```json
{
  "success": true,
  "data": {
    "updatedCount": 2,
    "failedCount": 0,
    "failedRecords": []
  },
  "timestamp": "2024-02-11T10:00:00Z"
}
```

**部分成功** (HTTP 207 Multi-Status)
```json
{
  "success": true,
  "data": {
    "updatedCount": 1,
    "failedCount": 1,
    "failedRecords": [
      {
        "sn": 999,
        "reason": "記錄不存在"
      }
    ]
  },
  "timestamp": "2024-02-11T10:00:00Z"
}
```

**請求錯誤** (HTTP 400)
```json
{
  "success": false,
  "error": {
    "code": "INVALID_REQUEST",
    "message": "records 陣列不能為空",
    "details": "至少需要提供一筆更新記錄"
  },
  "timestamp": "2024-02-11T10:00:00Z"
}
```

## DTO 定義

### ActivityRecordResponse

| 欄位名稱 | 類型 | 說明 | 範例 |
|---------|------|------|------|
| sn | integer | 記錄 SN (主鍵) | 1 |
| clientSn | integer | 客戶 SN (外鍵) | 12345 |
| activitySn | integer | 活動 SN (外鍵) | 2001 |
| activityName | string | 活動名稱 | "新春活動" |
| recordType | string | 記錄類型 | "PARTICIPATION" |
| recordDate | string | 記錄日期 (YYYY-MM-DD) | "2024-01-15" |
| recordValue | string | 記錄值 | "100" |
| calculateResult | string | 計算結果 | "COMPLETED" |
| status | string | 狀態 | "ACTIVE" |
| remarks | string | 備註 | "完成活動參與" |
| createdAt | string | 建立時間 (ISO 8601) | "2024-01-15T10:00:00Z" |
| updatedAt | string | 更新時間 (ISO 8601) | "2024-01-15T12:00:00Z" |

### RecordType 列舉值

| 值 | 說明 |
|----|------|
| PARTICIPATION | 參與記錄 |
| ACHIEVEMENT | 成就記錄 |
| REWARD | 獎勵記錄 |

### CalculateResult 列舉值

| 值 | 說明 |
|----|------|
| PENDING | 待計算 |
| PROCESSING | 計算中 |
| COMPLETED | 已完成 |
| FAILED | 計算失敗 |

## Mapper 方法對應

### 參數還原對應表

| 原始 Mapper 方法 | 對應 API 呼叫 | 說明 |
|-----------------|--------------|------|
| `selectbySn(@Param("from") Integer from, @Param("to") Integer to)` | `GET /activity-records?from={from}&to={to}` | from 和 to 皆為必填參數 |
| `updateCalculateResult(List<ActivityRecordEntity> records)` | `PUT /activity-records/calculate-results` | Request Body 傳入更新記錄陣列 |

### 程式碼遷移範例

**遷移前 (Mapper)**:
```java
// 範圍查詢活動記錄
List<ActivityRecordEntity> records = 
    activityRecordMapper.selectbySn(1, 100);

// 批次更新計算結果
List<ActivityRecordEntity> updates = new ArrayList<>();
ActivityRecordEntity record1 = new ActivityRecordEntity();
record1.setSn(1);
record1.setCalculateResult("COMPLETED");
updates.add(record1);

ActivityRecordEntity record2 = new ActivityRecordEntity();
record2.setSn(2);
record2.setCalculateResult("FAILED");
updates.add(record2);

Integer updatedCount = activityRecordMapper.updateCalculateResult(updates);
```

**遷移後 (API Client)**:
```java
// 範圍查詢活動記錄
List<ActivityRecordResponse> records = 
    apiClient.getActivityRecords(1, 100);

// 批次更新計算結果
List<ActivityRecordUpdateRequest> updates = new ArrayList<>();
updates.add(new ActivityRecordUpdateRequest(1, "COMPLETED"));
updates.add(new ActivityRecordUpdateRequest(2, "FAILED"));

ActivityRecordUpdateResponse response = 
    apiClient.updateActivityRecordCalculateResults(updates);
System.out.println("Updated: " + response.getUpdatedCount());
```

## 使用範例

### 範例 1: 範圍查詢活動記錄

**請求**:
```http
GET /api/v1/activity-records?from=1&to=100 HTTP/1.1
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
      "activitySn": 2001,
      "activityName": "新春活動",
      "recordType": "PARTICIPATION",
      "recordDate": "2024-01-15",
      "recordValue": "100",
      "calculateResult": "COMPLETED",
      "status": "ACTIVE",
      "remarks": "完成活動參與",
      "createdAt": "2024-01-15T10:00:00Z",
      "updatedAt": "2024-01-15T12:00:00Z"
    },
    {
      "sn": 2,
      "clientSn": 12346,
      "activitySn": 2001,
      "activityName": "新春活動",
      "recordType": "ACHIEVEMENT",
      "recordDate": "2024-01-16",
      "recordValue": "200",
      "calculateResult": "PENDING",
      "status": "ACTIVE",
      "remarks": "達成活動目標",
      "createdAt": "2024-01-16T10:00:00Z",
      "updatedAt": "2024-01-16T10:00:00Z"
    }
  ],
  "total": 2,
  "timestamp": "2024-02-11T10:00:00Z"
}
```

### 範例 2: 批次更新計算結果

**請求**:
```http
PUT /api/v1/activity-records/calculate-results HTTP/1.1
Host: api.example.com
Content-Type: application/json
Accept: application/json
Authorization: Bearer {access_token}

{
  "records": [
    {
      "sn": 1,
      "calculateResult": "COMPLETED"
    },
    {
      "sn": 2,
      "calculateResult": "FAILED"
    }
  ]
}
```

**回應**:
```json
{
  "success": true,
  "data": {
    "updatedCount": 2,
    "failedCount": 0,
    "failedRecords": []
  },
  "timestamp": "2024-02-11T10:00:00Z"
}
```

## 注意事項

1. **範圍查詢限制**: 建議限制 from 和 to 的範圍（例如：單次最多查詢 1000 筆）
2. **批次更新限制**: 建議限制單次更新的記錄數（例如：最多 100 筆）
3. **冪等性**: 更新操作應支援冪等性，重複更新相同資料不會產生副作用
4. **部分成功處理**: 批次更新時，部分記錄可能失敗，應返回詳細的成功/失敗資訊
5. **快取策略**: 
   - 查詢結果建議快取 3-5 分鐘（記錄可能頻繁變動）
   - 更新後應清除相關快取
6. **效能考量**: 範圍查詢和批次更新都需要注意資料量控制
7. **資料關聯**: 活動記錄與客戶資料、活動配置有關聯

## 相關資源

- Mapper 對應文件: [_mappers/ActivityRecordMapper.md](_mappers/ActivityRecordMapper.md)
- 相關 API: [cfg-activities.md](cfg-activities.md)
