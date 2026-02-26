# Mapper to API Specification - Product Study Progress

**產生日期**: 2026-02-26  
**來源專案**: OMS_StudyProgress / dal_products

---

## 總覽

本次分析了 Product Study Progress 專案中的所有 Mapper 檔案，產生簡潔的 API 規格文件。

### 統計資料

- **Mapper 總數**: 12
- **方法總數**: 20
- **資料庫連線**: 2 個（TutorERP, MuchNew）
- **簡單查詢**: 19 個
- **複雜查詢**: 1 個（updateScholarShip）

---

## 檔案結構

```
2026-02-26/
├── API-SPEC.md                  # 🎯 主表格（所有 Mapper 方法一覽）
├── input.md                     # 🎯 所有 Input DTO 定義
├── output.md                    # 🎯 所有 Output DTO 定義
├── updateScholarShip.md         # 複雜方法：獎學金計算更新
└── README.md                    # 本文件
```

---

## Mapper 清單

### TutorERP Database (8 個 Mapper)

1. **ActivityRecordMapper** - 獎學金活動記錄
   - `selectbySn(Integer from, Integer to)` - 根據序號範圍查詢
   - `updateScholarShip` - 計算並更新獎學金發放資格 ⭐ 複雜

2. **ActivityRuleDetailMapper** - 活動規則明細
   - `selectAll()` - 查詢所有記錄
   - `selectByLevelNo(String levelNo)` - 根據等級編號查詢

3. **ActivityRuleHeadMapper** - 活動規則表頭
   - `selectAll()` - 查詢所有記錄
   - `selectByActivitySn(int activitySn)` - 根據活動序號查詢

4. **ActivityRuleRewardMapper** - 活動規則獎勵
   - `selectAll()` - 查詢所有記錄
   - `selectBylevelNo(String levelNo)` - 根據等級編號查詢

5. **CfgActivityMapper** - 活動設定
   - `selectAll()` - 查詢所有記錄
   - `selectByPrimaryKey(Integer sn)` - 根據主鍵查詢

6. **CfgProductActivityMapper** - 產品活動設定
   - `selectByExample(CfgProductActivityEntityExample example)` - 條件查詢

7. **CfgProductMapper** - 產品設定
   - `selectByProductSn(int productSn)` - 根據產品序號查詢
   - `selectAllMuchNew()` - 查詢所有 MuchNew 產品

8. **SessionTypeMapper** - 課程類型
   - `selectAll()` - 查詢所有記錄
   - `selectByWebSite(String webSite)` - 根據網站查詢
   - `selectBySn(Integer sourceSn)` - 根據序號查詢

### MuchNew Database (4 個 Mapper)

9. **CfgContractActivityMapper** - 合約活動設定
   - `selectByContractSns(List contractSns)` - 根據合約序號清單查詢

10. **ClientPurchaseMapper** - 客戶購買記錄
    - `selectByClientSn(Integer clientSn)` - 根據客戶序號查詢

11. **ClientTemporalContractMapper** - 客戶臨時合約
    - `selectByPrimaryKey(Integer sn)` - 根據主鍵查詢
    - `selectByClientSn(Integer clientSn)` - 根據客戶序號查詢

12. **ContractChangeRecordMapper** - 合約變更記錄
    - `selectByAccountSn(Integer accountSn, Integer changeTypeSn)` - 根據帳號序號查詢

---

## 核心檔案說明

### 1. API-SPEC.md

主表格文件，列出所有 20 個 Mapper 方法的基本資訊。

**表格欄位**:

- Mapper: Mapper 名稱
- Method: 方法名稱
- 連線DB: 資料庫連線（TutorERP / MuchNew）
- Input: 輸入參數（連結到 input.md）
- Output: 輸出結構（連結到 output.md）
- Desc: 方法描述
- Detail: 詳細說明（複雜方法連結到獨立檔案）

### 2. input.md

集中定義所有 Input DTO，包含 13 種輸入參數類型：

- ActivityRecordRangeInput
- UpdateScholarshipInput
- LevelNoInput
- ActivitySnInput
- ActivityKeyInput
- ProductActivityExampleInput
- ProductSnInput
- WebSiteInput
- SessionTypeSnInput
- ContractSnsInput
- ClientSnInput
- ContractKeyInput
- ContractChangeQueryInput

每個 DTO 包含：

- 欄位定義表格（欄位、類型、必填、說明）
- JSON 範例

### 3. output.md

集中定義所有 Output DTO，包含 12 種輸出結構類型：

- ActivityRecordOutput
- UpdateScholarshipOutput
- ActivityRuleDetailOutput
- ActivityRuleHeadOutput
- ActivityRuleRewardOutput
- CfgActivityOutput
- CfgProductActivityOutput
- CfgProductOutput
- SessionTypeOutput
- CfgContractActivityOutput
- ClientPurchaseOutput
- ClientTemporalContractOutput
- ContractChangeRecordOutput

每個 DTO 包含：

- 欄位定義表格（欄位、類型、說明）
- JSON範例

### 4. updateScholarShip.md

**複雜方法獨立檔案**，包含完整的業務邏輯說明：

- **方法資訊**: 使用的 Mapper 方法、連線DB、複雜度
- **Input/Output**: 請求與回應格式（連結到 input.md / output.md）
- **處理流程**: 完整的業務流程圖
- **外部 API 呼叫**: TutorGroupAPI.postBookingRecordLite 說明
- **Type 1 / Type 2 計算邏輯**: 兩種不同的計算方式與範例輸出
- **批次與平行處理**: 實際程式碼範例
- **注意事項**: 效能、錯誤處理、資料一致性、日期處理
- **實作 Mapper 方法**: Java Interface 與 XML Mapper 範例
- **Service 實作範例**: 完整的 Service 層程式碼

---

## 使用方式

### 快速查詢 API 規格

1. 開啟 **API-SPEC.md** 查看所有方法一覽表
2. 點擊 Input 欄位的連結查看輸入參數定義
3. 點擊 Output 欄位的連結查看輸出結構定義
4. 若方法為複雜查詢，點擊 Detail 欄位的連結查看詳細說明

### 查詢特定 Input/Output

1. 開啟 **input.md** 或 **output.md**
2. 使用 Ctrl+F 搜尋 DTO 名稱
3. 或透過錨點連結直接跳轉

### 理解複雜業務邏輯

1. 開啟 **updateScholarShip.md**
2. 閱讀完整的處理流程、計算邏輯、注意事項
3. 參考實作程式碼範例

---

## 設計特點

### 1. 簡潔性

- 只產出三個核心檔案（API-SPEC.md, input.md, output.md）
- 簡單查詢直接填在表格中，不產生多餘檔案
- 複雜查詢才產生獨立說明檔案

### 2. 可讀性

- 清晰的表格格式
- 完整的 JSON 範例
- 錨點連結方便快速跳轉

### 3. 完整性

- 所有 Mapper 方法都有記錄
- Input 和 Output 定義完整
- 複雜方法有詳細的處理流程說明

### 4. 可維護性

- 集中管理 DTO 定義（input.md / output.md）
- 統一的文件格式
- 清楚的檔案結構

---

## 真實案例：updateScholarShip

這個方法展示了如何將多個 Mapper 方法整合為單一 API：

### 使用的 Mapper 方法

1. `selectbySn(Integer from, Integer to)` - 查詢獎學金記錄
2. `updateCalculateResult(List<ActivityRecordEntity> records)` - 批次更新結果

### 複雜度來源

- **多 Mapper 整合**: 2 個 Mapper 方法協同作業
- **外部 API 呼叫**: 呼叫 TutorGroupAPI 取得預約記錄
- **批次處理**: 每 10 筆一組分批處理
- **平行處理**: 使用 parallelStream 提升效能
- **複雜計算邏輯**: Type 1 和 Type 2 兩種不同的計算方式

### 文件價值

獨立的 updateScholarShip.md 提供：

- 完整的業務邏輯說明
- 詳細的計算公式與範例
- 批次處理實作方式
- 效能與錯誤處理注意事項
- 可直接參考的程式碼範例

---

## 下一步建議

1. **API 實作**
   - 根據 API-SPEC.md 實作對應的 RESTful API
   - 參考 input.md 定義 Request DTO
   - 參考 output.md 定義 Response DTO

2. **測試**
   - 根據 JSON 範例建立測試資料
   - 驗證複雜方法的計算邏輯

3. **文件維護**
   - 當 Mapper 有新增或修改時，更新對應的 API-SPEC.md
   - 保持 input.md 和 output.md 與實際 DTO 同步

---

## 參考資料

- **來源專案**: D:\workspace\oms\OMS_StudyProgress\dal_products
- **Mapper 路徑**: src\main\java\com\tutorabc\product\studyprogress\dal_products\DBMapper
- **Service 實作**: src\main\java\com\tutorabc\product\studyprogress\dal_products\Service\Impl
