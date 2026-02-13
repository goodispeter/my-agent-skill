# ClientTemporalContractMapper 對應表

本文件記錄 `ClientTemporalContractMapper` 所有方法與對應 API 端點的對照關係。

## Mapper 資訊

- **類別名稱**: `com.tutorabc.product.studyprogress.dal_products.DBMapper.MuchNew.ClientTemporalContractMapper`
- **類型**: MyBatis Interface Mapper
- **方法總數**: 2
- **對應資源**: [client-temporal-contracts.md](../client-temporal-contracts.md)

## 方法對應表

| 原始方法 | 參數 | 返回類型 | 對應 API 端點 | HTTP 方法 | 還原方式 |
|---------|------|---------|-------------|----------|----------|
| selectByClientSn | Integer clientSn | List&lt;ClientTemporalContractEntity&gt; | `/client-temporal-contracts?clientSn={clientSn}` | GET | 傳入 clientSn 參數 |
| selectByPrimaryKey | Integer sn | ClientTemporalContractEntity | `/client-temporal-contracts/{sn}` | GET | sn 作為路徑參數 |

## 設計說明

### RESTful 設計慣例

本 Mapper 的方法對應採用 **RESTful 設計慣例**，包含兩種查詢模式：

1. **selectByClientSn(Integer clientSn)** → `GET /client-temporal-contracts?clientSn={clientSn}`
   - 用途：查詢特定客戶的合約列表
   - 返回：陣列
   - 必填參數：clientSn（客戶 SN）

2. **selectByPrimaryKey(Integer sn)** → `GET /client-temporal-contracts/{sn}`
   - 用途：查詢單一合約
   - 返回：單一物件
   - 符合 RESTful Resource 端點設計

**設計理由**：
- 提供兩種不同的查詢需求：客戶視角 vs 合約視角
- 符合 HTTP 語意和 RESTful 最佳實踐
- 路徑參數用於資源定位，查詢參數用於資源篩選

## 遷移指南

### 程式碼遷移範例

**原始程式碼（使用 Mapper）**：
```java
// 範例 1: 依客戶查詢合約列表
List<ClientTemporalContractEntity> contracts = 
    clientTemporalContractMapper.selectByClientSn(12345);

// 範例 2: 查詢單一合約
ClientTemporalContractEntity contract = 
    clientTemporalContractMapper.selectByPrimaryKey(1);
```

**遷移後程式碼（使用 API Client）**：
```java
// 範例 1: 依客戶查詢合約列表
List<ClientTemporalContractResponse> contracts = 
    apiClient.getClientTemporalContracts(12345);

// 範例 2: 查詢單一合約
ClientTemporalContractResponse contract = 
    apiClient.getClientTemporalContractById(1);
```

## 注意事項

1. **返回類型變化**: 
   - `selectByClientSn()`: List&lt;ClientTemporalContractEntity&gt; → List&lt;ClientTemporalContractResponse&gt;
   - `selectByPrimaryKey()`: ClientTemporalContractEntity → ClientTemporalContractResponse (單一物件)
2. **404 處理**: 單筆查詢若找不到資料，API 返回 404 而非 null
3. **必填參數**: 列表查詢中 clientSn 為必填
4. **快取策略差異**:
   - 列表查詢建議快取 3-5 分鐘
   - 單筆查詢建議快取 5-10 分鐘
5. **資料關聯**: 臨時合約與客戶資料、合約變更記錄有關聯

## 相關資源

- API 規格文件: [client-temporal-contracts.md](../client-temporal-contracts.md)
- 相關 Mapper: [ContractChangeRecordMapper.md](ContractChangeRecordMapper.md)
