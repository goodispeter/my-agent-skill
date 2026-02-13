# Session Type API 規格

## 資源概述

- **Resource**: `session-type`
- **Base Path**: `/api/v1/session-types`
- **資料來源**: SQL Server (TutorERP.dbo.SessionType)

## API 端點

### 1. 查詢 Session Type 列表（合併方法）

- **Method**: `GET`
- **Path**: `/session-types`
- **Query Params**:
  - `webSite` (optional, string): 網站篩選
  - `sourceSn` (optional, integer): 來源 SN 篩選
- **合併來源**:
  - `SessionTypeMapper.selectAll()` → 不傳任何參數
  - `SessionTypeMapper.selectByWebSite(String)` → 傳 `?webSite={value}`
  - `SessionTypeMapper.selectBySn(Integer)` → 傳 `?sourceSn={value}`
- **Response**: `List<SessionTypeResponse>`

**回應範例**:
```json
[
  {
    "sn": 1,
    "sessionType": "1-on-1",
    "webSite": "tutorJr",
    "sourceSn": 100,
    "description": "一對一課程"
  },
  {
    "sn": 2,
    "sessionType": "group",
    "webSite": "tutorabc",
    "sourceSn": 101,
    "description": "小班課"
  }
]
```

**Mapper 方法還原對照**:

| 原始 Mapper 方法 | 等效 API 呼叫 | 說明 |
|-----------------|-------------|------|
| `selectAll()` | `GET /session-types` | 不傳任何參數，查詢全部 |
| `selectByWebSite("tutorJr")` | `GET /session-types?webSite=tutorJr` | 只傳 webSite 參數 |
| `selectBySn(100)` | `GET /session-types?sourceSn=100` | 只傳 sourceSn 參數 |
| 組合查詢 | `GET /session-types?webSite=tutorJr&sourceSn=100` | 可同時傳入多個參數 |

## DTO 定義

### SessionTypeResponse

```json
{
  "sn": "integer - Session Type ID",
  "sessionType": "string - 課程類型代碼",
  "webSite": "string - 所屬網站",
  "sourceSn": "integer - 來源系統 SN",
  "description": "string - 課程類型說明",
  "createTime": "datetime - 建立時間（選填）",
  "updateTime": "datetime - 更新時間（選填）"
}
```

## 錯誤處理

| HTTP Status | Error Code | 說明 |
|------------|-----------|------|
| 400 | INVALID_PARAMETER | 參數格式錯誤（如 sourceSn 非數字） |
| 404 | NOT_FOUND | 查詢結果為空 |
| 500 | INTERNAL_SERVER_ERROR | 伺服器內部錯誤 |

## 使用範例

### 範例 1: 查詢全部 Session Type
```http
GET /api/v1/session-types
```

### 範例 2: 查詢特定網站的 Session Type
```http
GET /api/v1/session-types?webSite=tutorJr
```

### 範例 3: 查詢特定來源的 Session Type
```http
GET /api/v1/session-types?sourceSn=100
```

### 範例 4: 組合查詢
```http
GET /api/v1/session-types?webSite=tutorabc&sourceSn=101
```

## 注意事項

1. **參數篩選**: 所有查詢參數都是可選的，不傳參數時返回全部資料
2. **效能考量**: 建議加入分頁參數（`page`, `size`）避免一次返回過多資料
3. **快取建議**: Session Type 資料相對穩定，建議 API 端實作快取機制
