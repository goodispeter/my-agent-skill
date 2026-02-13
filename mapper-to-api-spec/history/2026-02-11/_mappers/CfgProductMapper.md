# CfgProductMapper 對應表

本文件記錄 `CfgProductMapper` 所有方法與對應 API 端點的對照關係。

## Mapper 資訊

- **類別名稱**: `com.tutorabc.product.studyprogress.dal_products.DBMapper.TutorERP.CfgProductMapper`
- **類型**: MyBatis Interface Mapper
- **方法總數**: 2
- **對應資源**: [cfg-products.md](../cfg-products.md)

## 方法對應表

| 原始方法 | 參數 | 返回類型 | 對應 API 端點 | HTTP 方法 | 還原方式 |
|---------|------|---------|-------------|----------|----------|
| selectAllMuchNew | - | List&lt;CfgProductEntity&gt; | `/cfg-products` | GET | 不傳任何參數 |
| selectByProductSn | int productSn | CfgProductEntity | `/cfg-products/{productSn}` | GET | productSn 作為路徑參數 |

## 設計說明

### RESTful 設計慣例

本 Mapper 的方法對應採用 **RESTful 設計慣例**，並未合併為單一端點：

1. **selectAllMuchNew()** → `GET /cfg-products`
   - 用途：查詢資源列表
   - 返回：陣列
   - 符合 RESTful Collection 端點設計

2. **selectByProductSn(int productSn)** → `GET /cfg-products/{productSn}`
   - 用途：查詢單一資源
   - 返回：單一物件
   - 符合 RESTful Resource 端點設計

**設計理由**：
- 列表查詢與單筆查詢語意不同，應分別使用不同端點
- 符合 HTTP 語意：集合端點 vs 資源端點
- 提升 API 可讀性和語意清晰度

## 遷移指南

### 程式碼遷移範例

**原始程式碼（使用 Mapper）**：
```java
// 範例 1: 查詢全部產品配置
List<CfgProductEntity> all = cfgProductMapper.selectAllMuchNew();

// 範例 2: 查詢單一產品配置
CfgProductEntity product = cfgProductMapper.selectByProductSn(1001);
```

**遷移後程式碼（使用 API Client）**：
```java
// 範例 1: 查詢全部產品配置
List<CfgProductResponse> all = apiClient.getCfgProducts();

// 範例 2: 查詢單一產品配置
CfgProductResponse product = apiClient.getCfgProductByProductSn(1001);
```

## 注意事項

1. **返回類型變化**: 
   - `selectAllMuchNew()`: List&lt;CfgProductEntity&gt; → List&lt;CfgProductResponse&gt;
   - `selectByProductSn()`: CfgProductEntity → CfgProductResponse (單一物件)
2. **404 處理**: 單筆查詢若找不到資料，API 返回 404 而非 null
3. **快取策略差異**:
   - 列表查詢建議快取 10-15 分鐘
   - 單筆查詢建議快取 15-30 分鐘
4. **資料關聯**: 產品配置與產品活動 (cfg-product-activities) 有多對多關係

## 相關資源

- API 規格文件: [cfg-products.md](../cfg-products.md)
- 相關 Mapper: [CfgProductActivityMapper.md](CfgProductActivityMapper.md)
