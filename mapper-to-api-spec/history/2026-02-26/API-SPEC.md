# API 規格文件

## 文件資訊

- **產生日期**: 2026-02-26
- **來源路徑**: D:\workspace\oms\OMS_StudyProgress\dal_products\src\main\java\com\tutorabc\product\studyprogress\dal_products\DBMapper
- **Mapper 總數**: 12
- **方法總數**: 20

---

## API 列表

### TutorERP Database

| Mapper                   | API 名稱                 | 連線DB   | Input                                                                 | Output                                                           | 合併的 Mapper 方法                       | Desc                                                                                                              | Detail                       |
| ------------------------ | ------------------------ | -------- | --------------------------------------------------------------------- | ---------------------------------------------------------------- | ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------- | ---------------------------- |
| ActivityRecordMapper     | selectbySn               | TutorERP | [ActivityRecordRangeInput](input.md#activityrecordrangeinput)         | [ActivityRecordOutput](output.md#activityrecordoutput)[]         | selectbySn                               | 根據序號範圍查詢獎學金活動記錄                                                                                    | -                            |
| ActivityRecordMapper     | updateScholarShip        | TutorERP | [UpdateScholarshipInput](input.md#updatescholarshipinput)             | [UpdateScholarshipOutput](output.md#updatescholarshipoutput)     | selectbySn + updateCalculateResult       | 計算並更新獎學金發放資格                                                                                          | [詳情](updateScholarShip.md) |
| ActivityRuleDetailMapper | queryActivityRuleDetails | TutorERP | [ActivityRuleDetailQueryInput](input.md#activityruledetailqueryinput) | [ActivityRuleDetailOutput](output.md#activityruledetailoutput)[] | selectAll + selectByLevelNo              | 查詢活動規則明細（levelNo 可選，不傳則查全部）                                                                    | -                            |
| ActivityRuleHeadMapper   | queryActivityRuleHeads   | TutorERP | [ActivityRuleHeadQueryInput](input.md#activityruleheadqueryinput)     | [ActivityRuleHeadOutput](output.md#activityruleheadoutput)[]     | selectAll + selectByActivitySn           | 查詢活動規則表頭（activitySn 可選，不傳則查全部）                                                                 | -                            |
| ActivityRuleRewardMapper | queryActivityRuleRewards | TutorERP | [ActivityRuleRewardQueryInput](input.md#activityrulerewardqueryinput) | [ActivityRuleRewardOutput](output.md#activityrulerewardoutput)[] | selectAll + selectBylevelNo              | 查詢活動規則獎勵（levelNo 可選，不傳則查全部）                                                                    | -                            |
| CfgActivityMapper        | queryCfgActivities       | TutorERP | [CfgActivityQueryInput](input.md#cfgactivityqueryinput)               | [CfgActivityOutput](output.md#cfgactivityoutput)[]               | selectAll + selectByPrimaryKey           | 查詢活動設定（sn 可選，不傳則查全部，傳了查單筆）                                                                 | -                            |
| CfgProductActivityMapper | selectByExample          | TutorERP | [ProductActivityExampleInput](input.md#productactivityexampleinput)   | [CfgProductActivityOutput](output.md#cfgproductactivityoutput)[] | selectByExample                          | 根據條件範例查詢產品活動設定                                                                                      | -                            |
| CfgProductMapper         | queryCfgProducts         | TutorERP | [CfgProductQueryInput](input.md#cfgproductqueryinput)                 | [CfgProductOutput](output.md#cfgproductoutput)[]                 | selectByProductSn + selectAllMuchNew     | 查詢產品設定（productSn 可選，不傳則查全部 MuchNew 產品，全查條件:online_date > '2016-01-01' and muchnew_sn > 0） | -                            |
| SessionTypeMapper        | querySessionTypes        | TutorERP | [SessionTypeQueryInput](input.md#sessiontypequeryinput)               | [SessionTypeOutput](output.md#sessiontypeoutput)[]               | selectAll + selectByWebSite + selectBySn | 查詢課程類型（webSite/sn 可選，都不傳則查全部）                                                                   | -                            |

### MuchNew Database

| Mapper                       | API 名稱                     | 連線DB  | Input                                                                         | Output                                                                   | 合併的 Mapper 方法                    | Desc                                              | Detail |
| ---------------------------- | ---------------------------- | ------- | ----------------------------------------------------------------------------- | ------------------------------------------------------------------------ | ------------------------------------- | ------------------------------------------------- | ------ |
| CfgContractActivityMapper    | selectByContractSns          | MuchNew | [ContractSnsInput](input.md#contractsnsinput)                                 | [CfgContractActivityOutput](output.md#cfgcontractactivityoutput)[]       | selectByContractSns                   | 根據合約序號清單查詢合約活動設定                  | -      |
| ClientPurchaseMapper         | selectByClientSn             | MuchNew | [ClientSnInput](input.md#clientsninput)                                       | [ClientPurchaseOutput](output.md#clientpurchaseoutput)[]                 | selectByClientSn                      | 根據客戶序號查詢購買記錄                          | -      |
| ClientTemporalContractMapper | queryClientTemporalContracts | MuchNew | [ClientTemporalContractQueryInput](input.md#clienttemporalcontractqueryinput) | [ClientTemporalContractOutput](output.md#clienttemporalcontractoutput)[] | selectByPrimaryKey + selectByClientSn | 查詢臨時合約（sn 或 clientSn 擇一，都不傳則錯誤） | -      |
| ContractChangeRecordMapper   | selectByAccountSn            | MuchNew | [ContractChangeQueryInput](input.md#contractchangequeryinput)                 | [ContractChangeRecordOutput](output.md#contractchangerecordoutput)[]     | selectByAccountSn                     | 根據帳號序號查詢合約變更記錄                      | -      |

---

## 統計資訊

- **TutorERP Database**: 8 個 Mapper，16 個方法 → 9 個 API
- **MuchNew Database**: 4 個 Mapper，4 個方法 → 4 個 API
- **總計**: 12 個 Mapper，20 個方法 → **13 個 API**
- **合併效果**: 透過可選參數設計，減少 7 個 API 端點

### 合併說明

以下 Mapper 方法透過可選參數合併為單一 API：

1. **ActivityRuleDetailMapper** (2→1): `selectAll` + `selectByLevelNo` → `queryActivityRuleDetails`
2. **ActivityRuleHeadMapper** (2→1): `selectAll` + `selectByActivitySn` → `queryActivityRuleHeads`
3. **ActivityRuleRewardMapper** (2→1): `selectAll` + `selectBylevelNo` → `queryActivityRuleRewards`
4. **CfgActivityMapper** (2→1): `selectAll` + `selectByPrimaryKey` → `queryCfgActivities`
5. **CfgProductMapper** (2→1): `selectByProductSn` + `selectAllMuchNew` → `queryCfgProducts`
6. **SessionTypeMapper** (3→1): `selectAll` + `selectByWebSite` + `selectBySn` → `querySessionTypes`
7. **ClientTemporalContractMapper** (2→1): `selectByPrimaryKey` + `selectByClientSn` → `queryClientTemporalContracts`

---

## 補充說明

- **Input**: 點擊連結查看 [input.md](input.md) 中的完整定義
- **Output**: 點擊連結查看 [output.md](output.md) 中的完整定義
- **Detail**: 複雜方法會產生獨立檔案，點擊連結查看詳細說明
