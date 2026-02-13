# SessionTypeMapper 對應表

本文件記錄 `SessionTypeMapper` 所有方法與對應 API 端點的對照關係。

## Mapper 資訊

- **類別名稱**: `com.tutorabc.product.studyprogress.dal_products.DBMapper.TutorERP.SessionTypeMapper`
- **類型**: MyBatis Interface Mapper
- **方法總數**: 3
- **對應資源**: [session-type.md](../session-type.md)

## 方法對應表

| 原始方法 | 參數 | 返回類型 | 對應 API 端點 | HTTP 方法 | 還原方式 |
|---------|------|---------|-------------|----------|----------|
| selectAll | - | List&lt;SessionTypeEntity&gt; | `/session-types` | GET | 不傳任何參數 |
| selectByWebSite | String webSite | List&lt;SessionTypeEntity&gt; | `/session-types?webSite={webSite}` | GET | 傳入 webSite 參數 |
| selectBySn | Integer sourceSn | List&lt;SessionTypeEntity&gt; | `/session-types?sourceSn={sourceSn}` | GET | 傳入 sourceSn 參數 |

## 合併說明

### 查詢方法合併（3 個方法 → 1 個 API）

以下 3 個查詢方法已合併為單一 API 端點 `GET /session-types`：

1. **selectAll()**
   - API 呼叫：`GET /session-types`
   - 說明：不傳任何參數，返回全部資料

2. **selectByWebSite(String webSite)**
   - API 呼叫：`GET /session-types?webSite=tutorJr`
   - 說明：傳入 webSite 參數進行篩選
   - 範例：`GET /session-types?webSite=tutorabc`

3. **selectBySn(Integer sourceSn)**
   - API 呼叫：`GET /session-types?sourceSn=100`
   - 說明：傳入 sourceSn 參數進行篩選
   - 範例：`GET /session-types?sourceSn=101`

**組合查詢**：
- API 支援同時傳入多個參數
- 範例：`GET /session-types?webSite=tutorJr&sourceSn=100`

**合併優勢**：
- 減少 3 個 API 端點為 1 個
- 提供更靈活的查詢組合
- 符合 RESTful 設計原則
- 降低 API 維護成本

## 遷移指南

### 程式碼遷移範例

**原始程式碼（使用 Mapper）**：
```java
// 範例 1: 查詢全部
List<SessionTypeEntity> all = sessionTypeMapper.selectAll();

// 範例 2: 依網站查詢
List<SessionTypeEntity> byWebSite = sessionTypeMapper.selectByWebSite("tutorJr");

// 範例 3: 依來源 SN 查詢
List<SessionTypeEntity> bySn = sessionTypeMapper.selectBySn(100);
```

**遷移後程式碼（使用 API Client）**：
```java
// 範例 1: 查詢全部
List<SessionTypeResponse> all = apiClient.getSessionTypes();

// 範例 2: 依網站查詢
List<SessionTypeResponse> byWebSite = apiClient.getSessionTypes("tutorJr", null);

// 範例 3: 依來源 SN 查詢
List<SessionTypeResponse> bySn = apiClient.getSessionTypes(null, 100);

// 範例 4: 組合查詢（新功能）
List<SessionTypeResponse> combined = apiClient.getSessionTypes("tutorJr", 100);
```

## 注意事項

1. **返回類型變化**: Mapper 返回 `SessionTypeEntity`，API 返回 `SessionTypeResponse`（DTO）
2. **快取策略**: Session Type 資料相對穩定，建議實作本地快取
3. **錯誤處理**: API 調用失敗時需要實作 fallback 機制
4. **效能影響**: 從直接資料庫查詢改為 HTTP API 調用，會增加網路延遲

## 相關資源

- API 規格文件: [session-type.md](../session-type.md)
- OpenAPI 定義: [openapi-spec.yaml](../openapi-spec.yaml)
