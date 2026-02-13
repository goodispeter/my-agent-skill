# CfgContractActivityMapper 對應表

本文件記錄 `CfgContractActivityMapper` 所有方法與對應 API 端點的對照關係。

## Mapper 資訊

- **類別名稱**: `com.tutorabc.product.studyprogress.dal_products.DBMapper.MuchNew.CfgContractActivityMapper`
- **類型**: MyBatis Interface Mapper
- **方法總數**: 1
- **對應資源**: [cfg-contract-activities.md](../cfg-contract-activities.md)

## 方法對應表

| 原始方法 | 參數 | 返回類型 | 對應 API 端點 | HTTP 方法 | 還原方式 |
|---------|------|---------|-------------|----------|----------|
| selectByContractSns | List contractSns | List&lt;CfgContractActivityEntity&gt; | `/cfg-contract-activities/batch-query` | POST | Request Body 傳入 contractSns 陣列 |

## 設計說明

### 批次查詢設計

由於本 Mapper 接受 List 參數進行批次查詢，對應為 POST API 端點：

**selectByContractSns(List contractSns)** → `POST /cfg-contract-activities/batch-query`
- 用途：批次查詢多個合約的活動配置
- 返回：陣列（包含所有匹配的配置）
- 必填參數：contractSns（合約 SN 陣列）
- HTTP 方法：POST（因為參數為 List，避免 GET URL 長度限制）

**設計理由**：
- List 參數不適合用 GET 查詢字串（URL 長度限制）
- POST Request Body 支援複雜資料結構
- 符合批次查詢的 RESTful 設計慣例
- 提升查詢效能，減少多次 API 調用

## 遷移指南

### 程式碼遷移範例

**原始程式碼（使用 Mapper）**：
```java
// 批次查詢合約活動配置
List<Integer> contractSns = Arrays.asList(1001, 1002, 1003);
List<CfgContractActivityEntity> activities = 
    cfgContractActivityMapper.selectByContractSns(contractSns);
```

**遷移後程式碼（使用 API Client）**：
```java
// 批次查詢合約活動配置
List<Integer> contractSns = Arrays.asList(1001, 1002, 1003);
List<CfgContractActivityResponse> activities = 
    apiClient.batchQueryCfgContractActivities(contractSns);
```

## 注意事項

1. **返回類型變化**: Mapper 返回 `List<CfgContractActivityEntity>`，API 返回 `List<CfgContractActivityResponse>`
2. **HTTP 方法**: 使用 POST 而非 GET，因為參數為複雜資料結構
3. **陣列大小限制**: 建議限制 contractSns 陣列大小（例如：最多 100 個），避免查詢負擔過重
4. **空結果處理**: 若某些合約沒有關聯活動，不會出現在結果中（非錯誤）
5. **快取建議**: 合約活動配置相對穩定，建議實作 10-15 分鐘的本地快取
6. **效能優勢**: 批次查詢可顯著減少 API 調用次數，提升整體效能
7. **資料關聯**: 合約活動配置連接合約與活動兩個實體

## 相關資源

- API 規格文件: [cfg-contract-activities.md](../cfg-contract-activities.md)
- 相關 Mapper: [CfgActivityMapper.md](CfgActivityMapper.md)
- 相關 Mapper: [ClientTemporalContractMapper.md](ClientTemporalContractMapper.md)
