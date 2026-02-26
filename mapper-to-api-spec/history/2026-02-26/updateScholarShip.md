# updateScholarShip

## 方法資訊

- **Mapper**: ActivityRecordMapper
- **使用的 Mapper 方法**:
  - `selectbySn(Integer from, Integer to)` - 查詢獎學金記錄
  - `updateCalculateResult(List<ActivityRecordEntity> records)` - 批次更新計算結果
- **連線DB**: TutorERP
- **複雜度**: 複雜（多 Mapper 整合、外部 API 呼叫、批次處理）

---

## 說明

根據合約序號範圍，查詢獎學金活動記錄，並透過呼叫外部 API 取得預約記錄，計算學員是否符合獎學金發放資格。

支援兩種計算類型：

- **Type 1**: 根據總點數計算應上課堂數，檢查出席率是否達標（90%）
- **Type 2**: 按月份統計，檢查每月出席次數是否達標（每月至少10堂）

---

## Input

參考: [input.md - UpdateScholarshipInput](input.md#updatescholarshipinput)

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

## Output

參考: [output.md - UpdateScholarshipOutput](output.md#updatescholarshipoutput)

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

## 處理流程

```
1. 查詢獎學金記錄
   └─ Mapper: selectbySn(fromId, toId)

2. 分批處理（每 10 筆一組）
   └─ 使用 Java 8 Stream Partition

3. 平行處理每批資料
   ├─ 呼叫外部 API 取得預約記錄
   │  └─ TutorGroupAPI.postBookingRecordLite()
   │
   ├─ 根據活動類型計算資格
   │  ├─ Type 1: checkResultType1()
   │  │  └─ 計算出席率 = 已出席堂數 / 應上堂數
   │  │  └─ 判斷是否 >= 90%
   │  │
   │  └─ Type 2: checkResultType2()
   │     └─ 按月份統計出席次數
   │     └─ 判斷每月是否 >= 10 堂
   │
   └─ 更新計算結果
      └─ Mapper: updateCalculateResult(records)
```

---

## 外部 API 呼叫

### TutorGroupAPI - postBookingRecordLite

**用途**: 查詢學員的預約（上課）記錄

**請求參數**:

```json
{
  "clientSn": [1001, 1002, 1003],
  "contractSn": [5001, 5002, 5003],
  "tutorBrand": "2"
}
```

**回應欄位**:

- `contractSn`: 合約序號
- `sessionStartTime`: 課程開始時間
- `status`: 課程狀態（0: 待上課, 1+: 已完成）
- `refundStatus`: 退費狀態（1: 已退費）
- `useSessions`: 使用堂數

---

## Type 1 計算邏輯

**適用條件**: `type = 1`

### 計算公式

```java
// 計算應上課堂數
shouldRecordSession = (totalPoint / 65) * 9 / 10

// 統計已出席堂數（排除已退費、超過合約結束日期後的課程）
bookedSession = sum(record.useSessions where status > 0 and refundStatus != 1)

// 計算出席率
percent = bookedSession * 100 / shouldRecordSession

// 判斷結果
result = bookedSession >= shouldRecordSession ? "True-100%" : "False-{percent}%"

// 產生說明文字
note = "{result}-應使用:{shouldRecordSession}-已出席:{bookedSession}-歸還:{alreadyRefundSession}"
```

### 輸出範例

**符合資格**:

```
True-100%-應使用 : 90-已出席 : 92-歸還 : 0
```

**不符資格**:

```
False-87%-應使用 : 90-已出席 : 78-歸還 : 2
```

### 過濾條件

1. **排除已退費的課程**: `refundStatus = 1`
2. **排除超過合約結束日期後的課程**: `sessionStartTime > contractEDate + 1 day`
3. **只計算已完成的課程**: `status > 0`

---

## Type 2 計算邏輯

**適用條件**: `type = 2`

### 計算公式

```java
// 將課程按合約開始日期後的天數分配到 6 個月
// 每個月計算出席次數（排除已退費的課程）

for each month (0-5):
  used[month] = count(records where status > 0 and refundStatus != 1)

// 統計不達標的月份
unSatisfiedMonth = count(month where used[month] < 10)
unSatisfiedCount = sum(10 - used[month] where used[month] < 10)

// 判斷結果
result = unSatisfiedMonth == 0 ? "True" : "False-少{unSatisfiedMonth}個月-少{unSatisfiedCount}堂課"

// 產生說明文字
note = "{result}-出席次數 {used[0]} / {used[1]} / {used[2]} / {used[3]} / {used[4]} / {used[5]} /"
```

### 月份區間劃分

從合約開始日期算起：

- **第 1 個月**: 0-30 天
- **第 2 個月**: 31-61 天
- **第 3 個月**: 62-92 天
- **第 4 個月**: 93-123 天
- **第 5 個月**: 124-154 天
- **第 6 個月**: 155-185 天

### 輸出範例

**符合資格**:

```
True-出席次數 12 / 11 / 13 / 10 / 12 / 11 /
```

**不符資格**:

```
False-少2個月-少8堂課-出席次數 12 / 8 / 9 / 10 / 12 / 11 /
```

說明: 第 2 個月只上了 8 堂（少 2 堂），第 3 個月只上了 9 堂（少 1 堂），總共少 2 個月、8 堂課

---

## 批次與平行處理

### 分批策略

```java
// 將查詢結果分批（每批 10 筆）
Collection<List<ActivityRecordEntity>> multipleRecords =
    Java8StreamPartition.partition(records, 10);
```

**原因**:

- 外部 API 限制單次最多查詢 10 筆 clientSn/contractSn
- 避免單次處理過多資料導致效能問題

### 平行處理

```java
multipleRecords.parallelStream().forEach(batch -> {
    // 1. 呼叫外部 API 取得預約記錄
    List<BookingRecordDto> bookingRecords = getSessions(batch, brandId);

    // 2. 計算每筆記錄的資格
    batch.forEach(record -> {
        // 篩選該合約的預約記錄
        List<BookingRecordDto> subList = bookingRecords.stream()
            .filter(br -> br.getContractSn().equals(record.getContractSn()))
            .collect(Collectors.toList());

        // 根據類型計算
        String note = record.getType() == 1
            ? checkResultType1(record, subList)
            : checkResultType2(record, subList);

        record.setNote(note);
    });

    // 3. 批次更新計算結果
    mapper.updateCalculateResult(batch);
});
```

**優勢**:

- 使用 `parallelStream()` 提升處理速度
- 每批資料獨立處理，互不影響

---

## 注意事項

### 效能考量

1. **批次處理**
   - 每 10 筆一組，避免單次查詢過多資料
   - 減少外部 API 呼叫次數

2. **平行處理**
   - 使用 `parallelStream()` 提升處理速度
   - 適合資料量大的場景

3. **外部 API 限制**
   - 單次最多 10 筆 clientSn/contractSn
   - 需要控制批次大小

### 錯誤處理

1. **外部 API 呼叫失敗**
   - 返回空列表，不中斷整體處理
   - 記錄錯誤訊息到 log

2. **資料驗證**
   - 檢查 clientSn 和 contractSn 陣列長度（1-10）
   - 超過範圍則返回錯誤訊息

### 資料一致性

1. **批次更新**
   - 每批資料在平行處理中獨立執行
   - 每批資料的更新是原子性的（單次 `updateCalculateResult` 呼叫）

2. **計算結果儲存**
   - 計算結果存入 `note` 欄位
   - `update_time` 自動更新為當前時間

### 日期處理

1. **Type 1 日期過濾**
   - 合約結束日期 +1 天作為實際結束日期
   - 過濾超過實際結束日期後的課程

2. **Type 2 月份計算**
   - 使用 `TimeUnit.DAYS.convert()` 計算天數差異
   - 根據天數區間分配到對應月份

---

## 實作 Mapper 方法

### ActivityRecordMapper.java

```java
public interface ActivityRecordMapper {
    /**
     * 查詢獎學金記錄（根據序號範圍）
     * @param from 起始序號
     * @param to 結束序號
     * @return 獎學金記錄清單
     */
    List<ActivityRecordEntity> selectbySn(
        @Param("from") Integer from,
        @Param("to") Integer to
    );

    /**
     * 批次更新計算結果
     * @param records 獎學金記錄清單（需包含 sn 和 note）
     * @return 更新筆數
     */
    Integer updateCalculateResult(List<ActivityRecordEntity> records);
}
```

### XML Mapper

```xml
<!-- 查詢獎學金記錄 -->
<select id="selectbySn" resultType="ActivityRecordEntity">
    SELECT
        sn,
        client_sn,
        contract_sn,
        type,
        total_point,
        contract_s_date,
        contract_e_date,
        note,
        update_time
    FROM activity_records
    WHERE sn BETWEEN #{from} AND #{to}
    ORDER BY sn
</select>

<!-- 批次更新計算結果 -->
<update id="updateCalculateResult">
    <foreach collection="list" item="item" separator=";">
        UPDATE activity_records
        SET note = #{item.note},
            update_time = NOW()
        WHERE sn = #{item.sn}
    </foreach>
</update>
```

---

## Service 實作範例

### ScholarshipBulkServiceImpl.java

```java
@Service
public class ScholarshipBulkServiceImpl implements IScholarShipBulkService {

    @Autowired
    private ActivityRecordMapper mapper;

    @Override
    public RewardResponse<Integer> updateScholarShip(UpdateScholarshipsParams params) {
        RewardResponse<Integer> response =
            RewardResponseExtension.getDefaultRewardResponse();

        // 1. 查詢獎學金記錄
        List<ActivityRecordEntity> records =
            mapper.selectbySn(params.getFromId(), params.getToId());

        // 2. 分批處理（每 10 筆一組）
        Collection<List<ActivityRecordEntity>> multipleRecords =
            Java8StreamPartition.partition(records, 10);

        // 3. 平行處理每批資料
        multipleRecords.parallelStream().forEach(batch -> {
            // 3.1 呼叫外部 API 取得預約記錄
            List<BookingRecordDto> bookingRecords =
                getSessions(batch, params.getBrandId());

            // 3.2 計算每筆記錄的資格
            batch.forEach(record -> {
                // 篩選該合約的預約記錄
                List<BookingRecordDto> subList = bookingRecords.stream()
                    .filter(br -> br.getContractSn().equals(record.getContractSn()))
                    .collect(Collectors.toList());

                // 根據類型計算
                String note = record.getType() == 1
                    ? checkResultType1(record, subList)
                    : checkResultType2(record, subList);

                record.setNote(note);
            });

            // 3.3 批次更新計算結果
            mapper.updateCalculateResult(batch);
        });

        return response;
    }

    // 其他私有方法: checkResultType1, checkResultType2, getSessions...
}
```

---

## 總結

這個案例展示了：

1. **多個 Mapper 方法整合為單一 API**
   - `selectbySn` + `updateCalculateResult` → `updateScholarShip` API
   - 雖然使用 2 個 Mapper 方法，但對外只是一個業務功能

2. **複雜業務邏輯需要詳細說明**
   - Type 1 / Type 2 不同的計算邏輯與公式
   - 批次與平行處理流程設計
   - 外部 API 整合細節

3. **獨立檔案的價值**
   - 完整的處理流程說明
   - 計算邏輯公式與範例
   - 效能、錯誤處理、資料一致性注意事項
   - 實作程式碼參考範例

這樣的文件可以幫助開發人員快速理解複雜業務邏輯，並正確實作對應的 API 服務。
