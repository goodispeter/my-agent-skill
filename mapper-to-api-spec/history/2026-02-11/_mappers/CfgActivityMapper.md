# CfgActivityMapper 對應表

本文件記錄 `CfgActivityMapper` 所有方法與對應 API 端點的對照關係。

## Mapper 資訊

- **類別名稱**: `com.tutorabc.product.studyprogress.dal_products.DBMapper.TutorERP.CfgActivityMapper`
- **類型**: MyBatis Interface Mapper
- **方法總數**: 2
- **對應資源**: [cfg-activities.md](../cfg-activities.md)

## 方法對應表

| 原始方法 | 參數 | 返回類型 | 對應 API 端點 | HTTP 方法 | 還原方式 |
|---------|------|---------|-------------|----------|----------|
| selectAll | - | List&lt;CfgActivityEntity&gt; | `/cfg-activities` | GET | 不傳任何參數 |
| selectByPrimaryKey | Integer sn | CfgActivityEntity | `/cfg-activities/{sn}` | GET | sn 作為路徑參數 |

## 設計說明

### RESTful 設計慣例

本 Mapper 的方法對應採用 **RESTful 設計慣例**，並未合併為單一端點：

1. **selectAll()** → `GET /cfg-activities`
   - 用途：查詢資源列表
   - 返回：陣列
   - 符合 RESTful Collection 端點設計

2. **selectByPrimaryKey(Integer sn)** → `GET /cfg-activities/{sn}`
   - 用途：查詢單一資源
   - 返回：單一物件
   - 符合 RESTful Resource 端點設計

**設計理由**：
- 列表查詢與單筆查詢語意不同，應分別使用不同端點
- 符合 HTTP 語意：集合端點 vs 資源端點
- 提升 API 可讀性和語意清晰度
- 有助於實作不同的快取策略

**與合併設計的差異**：
- **合併設計**: `GET /cfg-activities?sn={sn}` (查詢參數)
- **RESTful 設計**: `GET /cfg-activities/{sn}` (路徑參數)
- RESTful 設計更符合業界標準和最佳實踐

## 遷移指南

### 程式碼遷移範例

**原始程式碼（使用 Mapper）**：
```java
// 範例 1: 查詢全部活動配置
List<CfgActivityEntity> all = cfgActivityMapper.selectAll();

// 範例 2: 查詢單一活動配置
CfgActivityEntity activity = cfgActivityMapper.selectByPrimaryKey(1001);
```

**遷移後程式碼（使用 API Client）**：
```java
// 範例 1: 查詢全部活動配置
List<CfgActivityResponse> all = apiClient.getCfgActivities();

// 範例 2: 查詢單一活動配置
CfgActivityResponse activity = apiClient.getCfgActivityById(1001);
```

## 注意事項

1. **返回類型變化**: 
   - `selectAll()`: List&lt;CfgActivityEntity&gt; → List&lt;CfgActivityResponse&gt;
   - `selectByPrimaryKey()`: CfgActivityEntity → CfgActivityResponse (單一物件)
2. **404 處理**: 單筆查詢若找不到資料，API 返回 404 而非 null
3. **快取策略差異**:
   - 列表查詢建議快取 5-10 分鐘
   - 單筆查詢建議快取 15-30 分鐘
4. **效能考量**: 列表查詢可能返回大量資料，建議前端實作分頁
5. **資料關聯**: 活動配置與活動規則、產品活動、合約活動有多對多關係

## 相關資源

- API 規格文件: [cfg-activities.md](../cfg-activities.md)
- 相關 Mapper: [CfgProductActivityMapper.md](CfgProductActivityMapper.md)
- 相關 Mapper: [CfgContractActivityMapper.md](CfgContractActivityMapper.md)
- 相關 API: [activity-rule-heads.md](../activity-rule-heads.md)
