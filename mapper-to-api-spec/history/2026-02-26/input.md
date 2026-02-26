# Input DTO 定義

本文件定義所有 API 的 Input 參數結構。

---

## ActivityRecordRangeInput

根據序號範圍查詢獎學金活動記錄的輸入參數。

| 欄位 | 類型    | 必填 | 說明     |
| ---- | ------- | ---- | -------- |
| from | Integer | ✓    | 起始序號 |
| to   | Integer | ✓    | 結束序號 |

**JSON 範例**:

```json
{
  "from": 1001,
  "to": 1100
}
```

---

## UpdateScholarshipInput

更新獎學金計算結果的輸入參數（複雜查詢，詳見 [updateScholarShip.md](updateScholarShip.md)）。

| 欄位    | 類型    | 必填 | 說明                        |
| ------- | ------- | ---- | --------------------------- |
| fromId  | Integer | ✓    | 起始合約序號                |
| toId    | Integer | ✓    | 結束合約序號                |
| brandId | Integer | ✓    | 品牌 ID（用於呼叫外部 API） |

**JSON 範例**:

```json
{
  "fromId": 1001,
  "toId": 1100,
  "brandId": 2
}
```

---

## ActivityRuleDetailQueryInput

查詢活動規則明細的輸入參數（合併 `selectAll` + `selectByLevelNo`）。

| 欄位    | 類型   | 必填 | 說明                       |
| ------- | ------ | ---- | -------------------------- |
| levelNo | String | ✗    | 等級編號（不傳則查詢全部） |

**JSON 範例**:

查詢特定等級：

```json
{
  "levelNo": "L001"
}
```

查詢全部：

```json
{}
```

**Mapper 方法還原**:

- `selectAll()` → 不傳任何參數或傳空物件
- `selectByLevelNo("L001")` → `{"levelNo": "L001"}`

---

## ActivityRuleHeadQueryInput

查詢活動規則表頭的輸入參數（合併 `selectAll` + `selectByActivitySn`）。

| 欄位       | 類型    | 必填 | 說明                       |
| ---------- | ------- | ---- | -------------------------- |
| activitySn | Integer | ✗    | 活動序號（不傳則查詢全部） |

**JSON 範例**:

查詢特定活動：

```json
{
  "activitySn": 5001
}
```

查詢全部：

```json
{}
```

**Mapper 方法還原**:

- `selectAll()` → 不傳任何參數或傳空物件
- `selectByActivitySn(5001)` → `{"activitySn": 5001}`

---

## ActivityRuleRewardQueryInput

查詢活動規則獎勵的輸入參數（合併 `selectAll` + `selectBylevelNo`）。

| 欄位    | 類型   | 必填 | 說明                       |
| ------- | ------ | ---- | -------------------------- |
| levelNo | String | ✗    | 等級編號（不傳則查詢全部） |

**JSON 範例**:

查詢特定等級：

```json
{
  "levelNo": "L001"
}
```

查詢全部：

```json
{}
```

**Mapper 方法還原**:

- `selectAll()` → 不傳任何參數或傳空物件
- `selectBylevelNo("L001")` → `{"levelNo": "L001"}`

---

## CfgActivityQueryInput

查詢活動設定的輸入參數（合併 `selectAll` + `selectByPrimaryKey`）。

| 欄位 | 類型    | 必填 | 說明                                       |
| ---- | ------- | ---- | ------------------------------------------ |
| sn   | Integer | ✗    | 活動序號（不傳則查詢全部，傳入則查詢單筆） |

**JSON 範例**:

查詢單筆：

```json
{
  "sn": 101
}
```

查詢全部：

```json
{}
```

**Mapper 方法還原**:

- `selectAll()` → 不傳任何參數或傳空物件
- `selectByPrimaryKey(101)` → `{"sn": 101}`

**注意**: 當傳入 `sn` 時，回傳結果仍為陣列格式（為了保持一致性），但只會包含一筆資料。

---

## ProductActivityExampleInput

使用 MyBatis Example 物件查詢產品活動設定的輸入參數。

| 欄位    | 類型                            | 必填 | 說明                         |
| ------- | ------------------------------- | ---- | ---------------------------- |
| example | CfgProductActivityEntityExample | ✓    | MyBatis Example 查詢條件物件 |

**說明**: 此方法使用 MyBatis Generator 產生的 Example 類別，支援複雜的動態查詢條件。

**範例** (使用 Example):

```java
CfgProductActivityEntityExample example = new CfgProductActivityEntityExample();
example.createCriteria()
    .andProductSnEqualTo(1001)
    .andActivitySnEqualTo(5001);
```

---

## CfgProductQueryInput

查詢產品設定的輸入參數（合併 `selectByProductSn` + `selectAllMuchNew`）。

| 欄位      | 類型    | 必填 | 說明                                    |
| --------- | ------- | ---- | --------------------------------------- |
| productSn | Integer | ✗    | 產品序號（不傳則查詢所有 MuchNew 產品） |

**JSON 範例**:

查詢特定產品：

```json
{
  "productSn": 2001
}
```

查詢所有 MuchNew 產品：

```json
{}
```

**Mapper 方法還原**:

- `selectAllMuchNew()` → 不傳任何參數或傳空物件
- `selectByProductSn(2001)` → `{"productSn": 2001}`

---

## SessionTypeQueryInput

查詢課程類型的輸入參數（合併 `selectAll` + `selectByWebSite` + `selectBySn`）。

| 欄位    | 類型    | 必填 | 說明           |
| ------- | ------- | ---- | -------------- |
| webSite | String  | ✗    | 網站名稱或代碼 |
| sn      | Integer | ✗    | 來源序號       |

**JSON 範例**:

查詢全部：

```json
{}
```

根據網站查詢：

```json
{
  "webSite": "TutorABC"
}
```

根據序號查詢：

```json
{
  "sn": 301
}
```

**Mapper 方法還原**:

- `selectAll()` → 不傳任何參數或傳空物件
- `selectByWebSite("TutorABC")` → `{"webSite": "TutorABC"}`
- `selectBySn(301)` → `{"sn": 301}`

**注意**: `webSite` 和 `sn` 可以同時傳入，會進行 AND 條件查詢。

---

## ContractSnsInput

根據合約序號清單查詢的輸入參數。

| 欄位        | 類型                | 必填 | 說明         |
| ----------- | ------------------- | ---- | ------------ |
| contractSns | List&lt;Integer&gt; | ✓    | 合約序號清單 |

**JSON 範例**:

```json
{
  "contractSns": [5001, 5002, 5003]
}
```

---

## ClientSnInput

根據客戶序號查詢的輸入參數。

| 欄位     | 類型    | 必填 | 說明     |
| -------- | ------- | ---- | -------- |
| clientSn | Integer | ✓    | 客戶序號 |

**JSON 範例**:

```json
{
  "clientSn": 10001
}
```

---

## ClientTemporalContractQueryInput

查詢臨時合約的輸入參數（合併 `selectByPrimaryKey` + `selectByClientSn`）。

| 欄位     | 類型    | 必填 | 說明     |
| -------- | ------- | ---- | -------- |
| sn       | Integer | ✗    | 合約主鍵 |
| clientSn | Integer | ✗    | 客戶序號 |

**JSON 範例**:

根據合約主鍵查詢：

```json
{
  "sn": 8001
}
```

根據客戶序號查詢：

```json
{
  "clientSn": 10001
}
```

**Mapper 方法還原**:

- `selectByPrimaryKey(8001)` → `{"sn": 8001}`
- `selectByClientSn(10001)` → `{"clientSn": 10001}`

**注意**: `sn` 和 `clientSn` 必須至少傳入一個。若兩者都傳，會進行 AND 條件查詢。

---

## ContractChangeQueryInput

查詢合約變更記錄的輸入參數。

| 欄位         | 類型    | 必填 | 說明         |
| ------------ | ------- | ---- | ------------ |
| accountSn    | Integer | ✓    | 帳號序號     |
| changeTypeSn | Integer | ✓    | 變更類型序號 |

**JSON 範例**:

```json
{
  "accountSn": 20001,
  "changeTypeSn": 3
}
```
