# ClientPurchaseMapper 對應表

本文件記錄 `ClientPurchaseMapper` 所有方法與對應 API 端點的對照關係。

## Mapper 資訊

- **類別名稱**: `com.tutorabc.product.studyprogress.dal_products.DBMapper.MuchNew.ClientPurchaseMapper`
- **類型**: MyBatis Interface Mapper
- **方法總數**: 1
- **對應資源**: [client-purchases.md](../client-purchases.md)

## 方法對應表

| 原始方法 | 參數 | 返回類型 | 對應 API 端點 | HTTP 方法 | 還原方式 |
|---------|------|---------|-------------|----------|----------|
| selectByClientSn | Integer clientSn | List&lt;ClientPurchaseEntity&gt; | `/client-purchases?clientSn={clientSn}` | GET | 傳入 clientSn 參數 |

## 設計說明

### 單一查詢方法設計

由於本 Mapper 僅有一個查詢方法，對應為單一 API 端點：

**selectByClientSn(Integer clientSn)** → `GET /client-purchases?clientSn={clientSn}`
- 用途：查詢特定客戶的購買記錄
- 返回：陣列
- 必填參數：clientSn（客戶 SN）

**設計特點**：
- clientSn 為必填參數，未提供則返回 400 錯誤
- 適合客戶管理系統、訂單管理系統整合
- 建議依購買日期倒序排列

## 遷移指南

### 程式碼遷移範例

**原始程式碼（使用 Mapper）**：
```java
// 查詢客戶購買記錄
List<ClientPurchaseEntity> purchases = 
    clientPurchaseMapper.selectByClientSn(12345);
```

**遷移後程式碼（使用 API Client）**：
```java
// 查詢客戶購買記錄
List<ClientPurchaseResponse> purchases = 
    apiClient.getClientPurchases(12345);
```

## 注意事項

1. **返回類型變化**: Mapper 返回 `List<ClientPurchaseEntity>`，API 返回 `List<ClientPurchaseResponse>`
2. **必填參數**: clientSn 為必填，不提供會返回 400 錯誤（與 Mapper 可能返回 null 不同）
3. **資料量考量**: 購買記錄可能很多，建議 API 實作分頁或增加日期範圍參數
4. **快取建議**: 購買記錄相對穩定，建議實作 5-10 分鐘的本地快取
5. **效能考量**: 從直接資料庫查詢改為 HTTP API 調用，需注意網路延遲

## 相關資源

- API 規格文件: [client-purchases.md](../client-purchases.md)
- 相關 API: [cfg-products.md](../cfg-products.md)
