# ContractChangeRecordMapper 對應表

本文件記錄 `ContractChangeRecordMapper` 所有方法與對應 API 端點的對照關係。

## Mapper 資訊

- **類別名稱**: `com.tutorabc.product.studyprogress.dal_products.DBMapper.MuchNew.ContractChangeRecordMapper`
- **類型**: MyBatis Interface Mapper
- **方法總數**: 1
- **對應資源**: [contract-change-records.md](../contract-change-records.md)

## 方法對應表

| 原始方法 | 參數 | 返回類型 | 對應 API 端點 | HTTP 方法 | 還原方式 |
|---------|------|---------|-------------|----------|----------|
| selectByAccountSn | @Param("accountSn") Integer accountSn, @Param("changeTypeSn") Integer changeTypeSn | List&lt;ContractChangeRecordEntity&gt; | `/contract-change-records?accountSn={accountSn}&changeTypeSn={changeTypeSn}` | GET | accountSn 必填，changeTypeSn 可選 |

## 設計說明

### 複合查詢參數設計

由於本 Mapper 僅有一個方法，但接受兩個參數，對應為單一 API 端點：

**selectByAccountSn(Integer accountSn, Integer changeTypeSn)** → `GET /contract-change-records?accountSn={accountSn}&changeTypeSn={changeTypeSn}`
- 用途：查詢特定帳戶的合約變更記錄，可選擇性依變更類型篩選
- 返回：陣列
- 必填參數：accountSn（帳戶 SN）
- 可選參數：changeTypeSn（變更類型 SN）

**設計特點**：
- accountSn 為必填參數，未提供則返回 400 錯誤
- changeTypeSn 為可選參數，未提供則返回所有類型的變更記錄
- 提供精確篩選與全量查詢兩種模式

## 遷移指南

### 程式碼遷移範例

**原始程式碼（使用 Mapper）**：
```java
// 範例 1: 查詢帳戶所有變更記錄
List<ContractChangeRecordEntity> allRecords = 
    contractChangeRecordMapper.selectByAccountSn(12345, null);

// 範例 2: 查詢特定類型的變更記錄
List<ContractChangeRecordEntity> typeRecords = 
    contractChangeRecordMapper.selectByAccountSn(12345, 1);
```

**遷移後程式碼（使用 API Client）**：
```java
// 範例 1: 查詢帳戶所有變更記錄
List<ContractChangeRecordResponse> allRecords = 
    apiClient.getContractChangeRecords(12345, null);

// 範例 2: 查詢特定類型的變更記錄
List<ContractChangeRecordResponse> typeRecords = 
    apiClient.getContractChangeRecords(12345, 1);
```

## 注意事項

1. **返回類型變化**: Mapper 返回 `List<ContractChangeRecordEntity>`，API 返回 `List<ContractChangeRecordResponse>`
2. **必填參數**: accountSn 為必填，不提供會返回 400 錯誤
3. **可選參數**: changeTypeSn 為可選，null 時返回所有類型
4. **稽核特性**: 變更記錄具有稽核價值，建議只允許查詢不允許修改或刪除
5. **資料量考量**: 記錄可能很多，建議 API 實作分頁或增加日期範圍參數
6. **快取建議**: 變更記錄相對穩定（只增不改），建議實作 10-15 分鐘的本地快取
7. **效能考量**: 從直接資料庫查詢改為 HTTP API 調用，需注意網路延遲

## 相關資源

- API 規格文件: [contract-change-records.md](../contract-change-records.md)
- 相關 Mapper: [ClientTemporalContractMapper.md](ClientTemporalContractMapper.md)
