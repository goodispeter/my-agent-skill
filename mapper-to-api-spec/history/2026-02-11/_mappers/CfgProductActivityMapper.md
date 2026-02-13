# CfgProductActivityMapper 對應表

本文件記錄 `CfgProductActivityMapper` 所有方法與對應 API 端點的對照關係。

## Mapper 資訊

- **類別名稱**: `com.tutorabc.product.studyprogress.dal_products.DBMapper.TutorERP.CfgProductActivityMapper`
- **類型**: MyBatis Interface Mapper
- **方法總數**: 1
- **對應資源**: [cfg-product-activities.md](../cfg-product-activities.md)

## 方法對應表

| 原始方法 | 參數 | 返回類型 | 對應 API 端點 | HTTP 方法 | 還原方式 |
|---------|------|---------|-------------|----------|----------|
| selectByExample | CfgProductActivityEntityExample example | List&lt;CfgProductActivityEntity&gt; | `/cfg-product-activities/query` | POST | Request Body 傳入查詢條件物件 |

## 設計說明

### 條件查詢設計

由於本 Mapper 使用 MyBatis Example 物件進行條件查詢，對應為 POST API 端點：

**selectByExample(CfgProductActivityEntityExample example)** → `POST /cfg-product-activities/query`
- 用途：根據多種條件組合查詢產品活動配置
- 返回：陣列
- 參數結構：複雜查詢條件物件（支援多欄位篩選）
- HTTP 方法：POST（因為查詢條件複雜，使用 Request Body）

**設計理由**：
- MyBatis Example 物件支援複雜條件組合（AND/OR、比較運算等）
- POST Request Body 適合傳遞複雜資料結構
- 提供彈性的查詢能力，滿足不同業務場景
- 符合 RESTful 查詢資源的設計慣例

## Example 物件對應

### MyBatis Example 條件對應 API 查詢參數

| MyBatis Example 條件 | API 查詢參數 | 說明 |
|---------------------|-------------|------|
| `criteria.andProductSnEqualTo(1001)` | `productSn: 1001` | 產品 SN 等於 |
| `criteria.andActivitySnEqualTo(2001)` | `activitySn: 2001` | 活動 SN 等於 |
| `criteria.andStatusEqualTo("ACTIVE")` | `status: "ACTIVE"` | 狀態等於 |
| `criteria.andIsActiveEqualTo(true)` | `isActive: true` | 是否啟用等於 |
| `criteria.andStartDateGreaterThanOrEqualTo("2024-01-01")` | `startDateFrom: "2024-01-01"` | 開始日期 >= |
| `criteria.andStartDateLessThanOrEqualTo("2024-12-31")` | `startDateTo: "2024-12-31"` | 開始日期 <= |

## 遷移指南

### 程式碼遷移範例

**原始程式碼（使用 Mapper）**：
```java
// 範例 1: 單一條件查詢
CfgProductActivityEntityExample example1 = new CfgProductActivityEntityExample();
example1.createCriteria().andProductSnEqualTo(1001);
List<CfgProductActivityEntity> result1 = 
    cfgProductActivityMapper.selectByExample(example1);

// 範例 2: 多條件組合查詢
CfgProductActivityEntityExample example2 = new CfgProductActivityEntityExample();
CfgProductActivityEntityExample.Criteria criteria = example2.createCriteria();
criteria.andProductSnEqualTo(1001);
criteria.andActivitySnEqualTo(2001);
criteria.andStatusEqualTo("ACTIVE");
criteria.andIsActiveEqualTo(true);
List<CfgProductActivityEntity> result2 = 
    cfgProductActivityMapper.selectByExample(example2);

// 範例 3: 日期範圍查詢
CfgProductActivityEntityExample example3 = new CfgProductActivityEntityExample();
CfgProductActivityEntityExample.Criteria criteria3 = example3.createCriteria();
criteria3.andStartDateGreaterThanOrEqualTo("2024-01-01");
criteria3.andStartDateLessThanOrEqualTo("2024-12-31");
List<CfgProductActivityEntity> result3 = 
    cfgProductActivityMapper.selectByExample(example3);
```

**遷移後程式碼（使用 API Client）**：
```java
// 範例 1: 單一條件查詢
CfgProductActivityQueryRequest request1 = CfgProductActivityQueryRequest.builder()
    .productSn(1001)
    .build();
List<CfgProductActivityResponse> result1 = 
    apiClient.queryCfgProductActivities(request1);

// 範例 2: 多條件組合查詢
CfgProductActivityQueryRequest request2 = CfgProductActivityQueryRequest.builder()
    .productSn(1001)
    .activitySn(2001)
    .status("ACTIVE")
    .isActive(true)
    .build();
List<CfgProductActivityResponse> result2 = 
    apiClient.queryCfgProductActivities(request2);

// 範例 3: 日期範圍查詢
CfgProductActivityQueryRequest request3 = CfgProductActivityQueryRequest.builder()
    .startDateFrom("2024-01-01")
    .startDateTo("2024-12-31")
    .build();
List<CfgProductActivityResponse> result3 = 
    apiClient.queryCfgProductActivities(request3);
```

## 注意事項

1. **返回類型變化**: Mapper 返回 `List<CfgProductActivityEntity>`，API 返回 `List<CfgProductActivityResponse>`
2. **查詢參數簡化**: API 提供簡化的查詢參數，而非完整的 MyBatis Example 物件
3. **彈性查詢**: 所有查詢參數皆為可選，可自由組合
4. **空條件處理**: 不傳任何條件則返回全部資料（建議 API 實作分頁或限制返回筆數）
5. **快取建議**: 產品活動配置相對穩定，建議實作 10-15 分鐘的本地快取
6. **效能考量**: 複雜條件查詢可能較慢，建議後端實作查詢結果快取
7. **分頁建議**: 若資料量大，建議 API 支援分頁參數

## 相關資源

- API 規格文件: [cfg-product-activities.md](../cfg-product-activities.md)
- 相關 Mapper: [CfgProductMapper.md](CfgProductMapper.md)
- 相關 Mapper: [CfgActivityMapper.md](CfgActivityMapper.md)
