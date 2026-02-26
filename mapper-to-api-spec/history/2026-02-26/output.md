# Output DTO 定義

本文件定義所有 API 的 Output 回應結構。

---

## ActivityRecordOutput

獎學金活動記錄的輸出結構。

| 欄位          | 類型     | 說明                                 |
| ------------- | -------- | ------------------------------------ |
| sn            | Integer  | 記錄序號（主鍵）                     |
| clientSn      | Integer  | 客戶序號                             |
| contractSn    | Integer  | 合約序號                             |
| type          | Integer  | 獎學金類型（1: 出席率, 2: 按月統計） |
| totalPoint    | Integer  | 總點數                               |
| contractSDate | Date     | 合約開始日期                         |
| contractEDate | Date     | 合約結束日期                         |
| note          | String   | 計算結果說明                         |
| updateTime    | DateTime | 更新時間                             |

**JSON 範例**:

```json
{
  "sn": 1001,
  "clientSn": 12345,
  "contractSn": 5001,
  "type": 1,
  "totalPoint": 6500,
  "contractSDate": "2023-01-01",
  "contractEDate": "2023-12-31",
  "note": "True-100%-應使用 : 90-已出席 : 92-歸還 : 0",
  "updateTime": "2024-02-26T10:30:00"
}
```

---

## UpdateScholarshipOutput

更新獎學金計算的輸出結構（詳見 [updateScholarShip.md](updateScholarShip.md)）。

| 欄位    | 類型    | 說明     |
| ------- | ------- | -------- |
| success | Boolean | 執行結果 |
| message | String  | 訊息     |
| data    | Integer | 更新筆數 |

**JSON 範例**:

```json
{
  "success": true,
  "message": "執行成功",
  "data": 95
}
```

---

## ActivityRuleDetailOutput

活動規則明細的輸出結構。

| 欄位            | 類型     | 說明     |
| --------------- | -------- | -------- |
| levelNo         | String   | 等級編號 |
| activitySn      | Integer  | 活動序號 |
| ruleName        | String   | 規則名稱 |
| ruleDescription | String   | 規則描述 |
| conditionType   | String   | 條件類型 |
| conditionValue  | String   | 條件值   |
| priority        | Integer  | 優先順序 |
| isActive        | Boolean  | 是否啟用 |
| createdAt       | DateTime | 建立時間 |
| updatedAt       | DateTime | 更新時間 |

**JSON 範例**:

```json
{
  "levelNo": "LEVEL_1",
  "activitySn": 1001,
  "ruleName": "新手獎勵規則",
  "ruleDescription": "完成首次課程即可獲得獎勵",
  "conditionType": "FIRST_LESSON",
  "conditionValue": "1",
  "priority": 1,
  "isActive": true,
  "createdAt": "2024-01-01T00:00:00",
  "updatedAt": "2024-01-15T10:00:00"
}
```

---

## ActivityRuleHeadOutput

活動規則表頭的輸出結構。

| 欄位       | 類型     | 說明             |
| ---------- | -------- | ---------------- |
| sn         | Integer  | 規則序號（主鍵） |
| activitySn | Integer  | 活動序號         |
| ruleName   | String   | 規則名稱         |
| ruleType   | String   | 規則類型         |
| startDate  | Date     | 開始日期         |
| endDate    | Date     | 結束日期         |
| isActive   | Boolean  | 是否啟用         |
| priority   | Integer  | 優先順序         |
| createdAt  | DateTime | 建立時間         |
| updatedAt  | DateTime | 更新時間         |

**JSON 範例**:

```json
{
  "sn": 101,
  "activitySn": 1001,
  "ruleName": "新春活動規則",
  "ruleType": "PROMOTION",
  "startDate": "2024-01-01",
  "endDate": "2024-02-29",
  "isActive": true,
  "priority": 10,
  "createdAt": "2024-01-01T00:00:00",
  "updatedAt": "2024-01-15T10:00:00"
}
```

---

## ActivityRuleRewardOutput

活動規則獎勵的輸出結構。

| 欄位              | 類型     | 說明             |
| ----------------- | -------- | ---------------- |
| sn                | Integer  | 獎勵序號（主鍵） |
| levelNo           | String   | 等級編號         |
| rewardType        | String   | 獎勵類型         |
| rewardValue       | String   | 獎勵值           |
| rewardDescription | String   | 獎勵描述         |
| isActive          | Boolean  | 是否啟用         |
| createdAt         | DateTime | 建立時間         |
| updatedAt         | DateTime | 更新時間         |

**JSON 範例**:

```json
{
  "sn": 201,
  "levelNo": "LEVEL_1",
  "rewardType": "POINT",
  "rewardValue": "100",
  "rewardDescription": "新手獎勵 100 點",
  "isActive": true,
  "createdAt": "2024-01-01T00:00:00",
  "updatedAt": "2024-01-15T10:00:00"
}
```

---

## CfgActivityOutput

活動設定的輸出結構。

| 欄位                | 類型     | 說明             |
| ------------------- | -------- | ---------------- |
| sn                  | Integer  | 活動序號（主鍵） |
| activityName        | String   | 活動名稱         |
| activityCode        | String   | 活動代碼         |
| activityType        | String   | 活動類型         |
| description         | String   | 活動描述         |
| startDate           | Date     | 開始日期         |
| endDate             | Date     | 結束日期         |
| status              | String   | 活動狀態         |
| targetAudience      | String   | 目標受眾         |
| maxParticipants     | Integer  | 最大參與人數     |
| currentParticipants | Integer  | 當前參與人數     |
| budgetAmount        | Decimal  | 預算金額         |
| isActive            | Boolean  | 是否啟用         |
| createdBy           | String   | 建立者           |
| createdAt           | DateTime | 建立時間         |
| updatedBy           | String   | 更新者           |
| updatedAt           | DateTime | 更新時間         |

**JSON 範例**:

```json
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
  "budgetAmount": 50000.0,
  "isActive": true,
  "createdBy": "admin",
  "createdAt": "2023-12-01T00:00:00",
  "updatedBy": "admin",
  "updatedAt": "2024-01-15T12:00:00"
}
```

---

## CfgProductActivityOutput

產品活動設定的輸出結構。

| 欄位         | 類型     | 說明             |
| ------------ | -------- | ---------------- |
| sn           | Integer  | 配置序號（主鍵） |
| productSn    | Integer  | 產品序號         |
| productName  | String   | 產品名稱         |
| activitySn   | Integer  | 活動序號         |
| activityName | String   | 活動名稱         |
| activityCode | String   | 活動代碼         |
| startDate    | Date     | 開始日期         |
| endDate      | Date     | 結束日期         |
| discountRate | Decimal  | 折扣率           |
| status       | String   | 狀態             |
| priority     | Integer  | 優先順序         |
| isActive     | Boolean  | 是否啟用         |
| createdAt    | DateTime | 建立時間         |
| updatedAt    | DateTime | 更新時間         |

**JSON 範例**:

```json
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
  "createdAt": "2024-01-01T00:00:00",
  "updatedAt": "2024-01-15T12:00:00"
}
```

---

## CfgProductOutput

產品設定的輸出結構。

| 欄位        | 類型     | 說明             |
| ----------- | -------- | ---------------- |
| productSn   | Integer  | 產品序號（主鍵） |
| productName | String   | 產品名稱         |
| productCode | String   | 產品代碼         |
| productType | String   | 產品類型         |
| description | String   | 產品描述         |
| price       | Decimal  | 價格             |
| status      | String   | 狀態             |
| isActive    | Boolean  | 是否啟用         |
| createdAt   | DateTime | 建立時間         |
| updatedAt   | DateTime | 更新時間         |

**JSON 範例**:

```json
{
  "productSn": 2001,
  "productName": "一對一英語課程",
  "productCode": "PROD_EN_001",
  "productType": "COURSE",
  "description": "專業外師一對一英語課程",
  "price": 12000.0,
  "status": "ACTIVE",
  "isActive": true,
  "createdAt": "2023-06-01T00:00:00",
  "updatedAt": "2024-01-10T10:00:00"
}
```

---

## SessionTypeOutput

課程類型的輸出結構。

| 欄位        | 類型     | 說明                 |
| ----------- | -------- | -------------------- |
| sn          | Integer  | 課程類型序號（主鍵） |
| typeName    | String   | 類型名稱             |
| typeCode    | String   | 類型代碼             |
| webSite     | String   | 適用網站             |
| sourceSn    | Integer  | 來源序號             |
| duration    | Integer  | 課程時長（分鐘）     |
| description | String   | 描述                 |
| isActive    | Boolean  | 是否啟用             |
| createdAt   | DateTime | 建立時間             |
| updatedAt   | DateTime | 更新時間             |

**JSON 範例**:

```json
{
  "sn": 301,
  "typeName": "一對一課程",
  "typeCode": "OO1",
  "webSite": "TutorABC",
  "sourceSn": 1,
  "duration": 45,
  "description": "真人外師一對一互動課程",
  "isActive": true,
  "createdAt": "2023-01-01T00:00:00",
  "updatedAt": "2024-01-01T10:00:00"
}
```

---

## CfgContractActivityOutput

合約活動設定的輸出結構。

| 欄位         | 類型     | 說明             |
| ------------ | -------- | ---------------- |
| sn           | Integer  | 配置序號（主鍵） |
| contractSn   | Integer  | 合約序號         |
| activitySn   | Integer  | 活動序號         |
| activityName | String   | 活動名稱         |
| activityCode | String   | 活動代碼         |
| joinDate     | Date     | 加入日期         |
| status       | String   | 狀態             |
| priority     | Integer  | 優先順序         |
| isActive     | Boolean  | 是否啟用         |
| createdAt    | DateTime | 建立時間         |
| updatedAt    | DateTime | 更新時間         |

**JSON 範例**:

```json
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
  "createdAt": "2024-01-01T00:00:00",
  "updatedAt": "2024-01-15T12:00:00"
}
```

---

## ClientPurchaseOutput

客戶購買記錄的輸出結構。

| 欄位         | 類型     | 說明             |
| ------------ | -------- | ---------------- |
| sn           | Integer  | 購買序號（主鍵） |
| clientSn     | Integer  | 客戶序號         |
| productSn    | Integer  | 產品序號         |
| productName  | String   | 產品名稱         |
| purchaseDate | Date     | 購買日期         |
| quantity     | Integer  | 數量             |
| unitPrice    | Decimal  | 單價             |
| totalAmount  | Decimal  | 總金額           |
| status       | String   | 狀態             |
| createdAt    | DateTime | 建立時間         |
| updatedAt    | DateTime | 更新時間         |

**JSON 範例**:

```json
{
  "sn": 5001,
  "clientSn": 10001,
  "productSn": 2001,
  "productName": "一對一英語課程",
  "purchaseDate": "2024-01-15",
  "quantity": 100,
  "unitPrice": 120.0,
  "totalAmount": 12000.0,
  "status": "COMPLETED",
  "createdAt": "2024-01-15T10:00:00",
  "updatedAt": "2024-01-15T10:30:00"
}
```

---

## ClientTemporalContractOutput

客戶臨時合約的輸出結構。

| 欄位             | 類型     | 說明             |
| ---------------- | -------- | ---------------- |
| sn               | Integer  | 合約序號（主鍵） |
| clientSn         | Integer  | 客戶序號         |
| contractType     | String   | 合約類型         |
| startDate        | Date     | 開始日期         |
| endDate          | Date     | 結束日期         |
| totalLessons     | Integer  | 總堂數           |
| usedLessons      | Integer  | 已使用堂數       |
| remainingLessons | Integer  | 剩餘堂數         |
| status           | String   | 狀態             |
| createdAt        | DateTime | 建立時間         |
| updatedAt        | DateTime | 更新時間         |

**JSON 範例**:

```json
{
  "sn": 8001,
  "clientSn": 10001,
  "contractType": "STANDARD",
  "startDate": "2024-01-01",
  "endDate": "2024-12-31",
  "totalLessons": 100,
  "usedLessons": 25,
  "remainingLessons": 75,
  "status": "ACTIVE",
  "createdAt": "2024-01-01T00:00:00",
  "updatedAt": "2024-02-20T15:30:00"
}
```

---

## ContractChangeRecordOutput

合約變更記錄的輸出結構。

| 欄位         | 類型     | 說明             |
| ------------ | -------- | ---------------- |
| sn           | Integer  | 記錄序號（主鍵） |
| accountSn    | Integer  | 帳號序號         |
| contractSn   | Integer  | 合約序號         |
| changeTypeSn | Integer  | 變更類型序號     |
| changeType   | String   | 變更類型名稱     |
| changeDate   | Date     | 變更日期         |
| beforeValue  | String   | 變更前值         |
| afterValue   | String   | 變更後值         |
| reason       | String   | 變更原因         |
| approvedBy   | String   | 核准者           |
| createdAt    | DateTime | 建立時間         |

**JSON 範例**:

```json
{
  "sn": 3001,
  "accountSn": 20001,
  "contractSn": 8001,
  "changeTypeSn": 3,
  "changeType": "LESSON_ADJUSTMENT",
  "changeDate": "2024-02-10",
  "beforeValue": "100",
  "afterValue": "120",
  "reason": "客戶加購課程",
  "approvedBy": "manager01",
  "createdAt": "2024-02-10T14:00:00"
}
```
