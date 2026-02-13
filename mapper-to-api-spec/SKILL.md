---
name: mapper-to-api-spec
description: 分析 SQL Server Mapper（MyBatis/JPA 等）並自動生成對應的 RESTful API 規格文件。用於將直接資料庫存取遷移至 API 架構。觸發詞："生成 API 規格"、"Mapper 轉 API"、"資料庫遷移至 API"、"分析 Mapper"。
---

# Mapper to API Specification Generator

將 SQL Server 的 Mapper 層轉換為 RESTful API 規格的多階段分析工具。

## 核心目標

從 MyBatis Mapper、JPA Repository 或其他資料存取層自動推導：

- RESTful API 端點設計
- Request/Response DTO 結構
- API 路由與 HTTP 方法對應
- OpenAPI 3.0 規格文件

## 架構流程

```
┌──────────────────────────────────────────────────────────────┐
│  Phase 1: Mapper Discovery & Classification                  │
│  → 掃描目標路徑，識別 Mapper 類型（MyBatis XML/Java/JPA）    │
│  → 分類：CRUD / 複雜查詢 / 批次操作 / 統計聚合               │
│  → Output: history/{date}/mapper-inventory.json              │
├──────────────────────────────────────────────────────────────┤
│  Phase 2: Method Consolidation Analysis                      │
│  → 識別可合併為單一 API 的 Mapper 方法                        │
│  → 分析相似查詢邏輯（同資源、不同條件）                        │
│  → 設計統一 API + 參數組合代替多個方法                    │
│  → 記錄參數對應關係（如何還原原始方法）                    │
│  → Output: history/{date}/_consolidation.json               │
├──────────────────────────────────────────────────────────────┤
│  Phase 3: Deep Analysis (平行 Agent)                         │
│  → 每個 Mapper 啟動一個分析 Agent                            │
│  → 解析 SQL 語句、方法簽章、參數、返回類型                   │
│  → 推導資料模型與關聯關係                                    │
│  → Output: history/{date}/_analysis/{mapper-name}.json       │
├──────────────────────────────────────────────────────────────┤
│  Phase 4: API Design Generation                              │
│  → 根據 RESTful 最佳實踐設計 API 端點                        │
│  → 應用合併分析結果，減少冗餘 API                           │
│  → CRUD → GET/POST/PUT/DELETE                                │
│  → 查詢 → GET with query params                              │
│  → 批次 → POST with array payload                            │
│  → Output: history/{date}/{resource}.md (資源視角)           │
│  → Output: history/{date}/_mappers/{Mapper}.md (追溯視角)    │
├──────────────────────────────────────────────────────────────┤
│  Phase 5: Specification Assembly (KM Ready)                  │
│  → 整合所有資源 API 為完整規格文件                           │
│  → 生成 OpenAPI 3.0 YAML (技術整合用)                        │
│  → 生成 API-SPEC.md (適合貼到 KM 的完整文件)                 │
│  → 包含目錄、範例、錯誤碼、合併方法對照表                  │
│  → Output: history/{date}/openapi-spec.yaml                  │
│  → Output: history/{date}/API-SPEC.md                        │
└──────────────────────────────────────────────────────────────┘
```

## 執行階段詳細說明

### Phase 1: Mapper Discovery

**輸入**:

- Mapper 根目錄路徑
- 專案類型提示（MyBatis/JPA/Hibernate）

**執行**:

```bash
# 搜尋所有 Mapper 檔案
glob **/*Mapper.java
glob **/*Mapper.xml
glob **/*Repository.java

# 分析檔案類型
grep @Mapper
grep @Repository
grep <select|insert|update|delete>
```

**輸出**: `mapper-inventory.json`

```json
{
  "project_type": "MyBatis",
  "total_mappers": 15,
  "mappers": [
    {
      "name": "StudentMapper",
      "type": "MyBatis_XML",
      "file_path": "dal_products/StudentMapper.java",
      "xml_path": "dal_products/StudentMapper.xml",
      "category": "CRUD_AND_QUERY",
      "method_count": 8,
      "complexity": "MEDIUM"
    }
  ]
}
```

---

### Phase 2: Method Consolidation Analysis

**目標**: 識別可合併為單一 API 的 Mapper 方法，減少 API 數量並提升一致性

**合併識別規則**:

1. **相同資源的查詢變體**
   - 例：`selectByType(String type)`, `selectByStatus(String status)`, `selectByTypeAndStatus(...)`
   - 合併為：`GET /resources?type={type}&status={status}`

2. **分頁 vs. 全部查詢**
   - 例：`selectAll()`, `selectPage(int page, int size)`
   - 合併為：`GET /resources?page={page}&size={size}` (分頁參數可選)

3. **單筆 vs. 多筆操作**
   - 例：`delete(Long id)`, `batchDelete(List<Long> ids)`
   - 合併為：`DELETE /resources` + body `{"ids": [1, 2, 3]}` (單筆傳 array 長度 1)

4. **排序變體**
   - 例：`selectOrderByName()`, `selectOrderByDate()`
   - 合併為：`GET /resources?sort=name` 或 `sort=date`

**輸出**: `_consolidation.json`

```json
{
  "consolidations": [
    {
      "merged_api": {
        "endpoint": "GET /session-types",
        "params": [
          { "name": "type", "type": "string", "required": false },
          { "name": "status", "type": "string", "required": false },
          { "name": "includeInactive", "type": "boolean", "default": false }
        ]
      },
      "source_methods": [
        {
          "mapper": "SessionTypeMapper",
          "method": "selectByType",
          "params": ["String type"],
          "equivalent_call": "GET /session-types?type={type}"
        },
        {
          "mapper": "SessionTypeMapper",
          "method": "selectByStatus",
          "params": ["String status"],
          "equivalent_call": "GET /session-types?status={status}"
        },
        {
          "mapper": "SessionTypeMapper",
          "method": "selectAllIncludeInactive",
          "params": [],
          "equivalent_call": "GET /session-types?includeInactive=true"
        }
      ],
      "consolidation_reason": "相同資源的查詢變體，只是過濾條件不同",
      "benefits": "減少 3 個 API 端點為 1 個，提升 API 一致性"
    }
  ],
  "summary": {
    "total_methods": 47,
    "after_consolidation": 28,
    "reduced_apis": 19
  }
}
```

---

### Phase 3: Deep Analysis（平行 Agent）

**每個 Mapper 啟動一個 Agent**，執行：

1. **解析方法定義**
   - 方法名稱與參數
   - 返回類型（單筆/列表/分頁）
   - 註解資訊

2. **解析 SQL 邏輯**
   - SELECT → 查詢欄位與條件
   - INSERT/UPDATE → 異動欄位
   - JOIN → 關聯資料表
   - WHERE → 查詢條件邏輯

3. **推導資料模型**
   - 主要實體類別
   - DTO 結構
   - 關聯關係（1-1, 1-N, N-N）

**輸出**: `analysis/StudentMapper.json`

```json
{
  "mapper": "StudentMapper",
  "resource_name": "student",
  "methods": [
    {
      "name": "selectById",
      "sql_type": "SELECT",
      "params": [{ "name": "id", "type": "Long" }],
      "return_type": "Student",
      "suggested_api": "GET /students/{id}"
    },
    {
      "name": "selectByCondition",
      "sql_type": "SELECT",
      "params": [{ "name": "condition", "type": "StudentQuery" }],
      "return_type": "List<Student>",
      "suggested_api": "GET /students?name={name}&grade={grade}"
    }
  ],
  "entities": {
    "Student": {
      "fields": ["id", "name", "grade", "email", "createTime"]
    }
  }
}
```

---

### Phase 4: API Design Generation

**設計原則**:

- RESTful 命名規範（複數資源名稱）
- HTTP 方法語意正確（GET 查詢、POST 新增、PUT 更新、DELETE 刪除）
- 查詢參數 vs. Path 參數
- 分頁、排序、篩選標準化

**輸出**: `api-design/student.md`

````markdown
# Student API 設計

## 資源概述

- **Resource**: `student`
- **Base Path**: `/api/v1/students`

## API 端點

### 1. 查詢單一學生

- **Method**: `GET`
- **Path**: `/students/{id}`
- **原始 Mapper**: `StudentMapper.selectById(Long id)`
- **Response**: `StudentResponse`

### 2. 條件查詢學生列表（合併多個 Mapper 方法）

- **Method**: `GET`
- **Path**: `/students`
- **Query Params**:
  - `name` (optional): 學生姓名
  - `grade` (optional): 年級
  - `status` (optional): 狀態（active/inactive）
  - `includeGraduated` (optional, boolean, default: false): 是否包含畢業生
  - `page` (default: 0)
  - `size` (default: 20)
- **合併來源**:
  - `StudentMapper.selectByCondition(StudentQuery)` → 基本查詢
  - `StudentMapper.selectByGrade(String grade)` → 傳 `?grade={grade}`
  - `StudentMapper.selectActiveOnly()` → 傳 `?status=active`
  - `StudentMapper.selectIncludingGraduated()` → 傳 `?includeGraduated=true`
- **Response**: `PagedResponse<StudentResponse>`

**Mapper 方法還原對照**:
| 原始 Mapper 方法 | 等效 API 呼叫 |
|-----------------|-------------|
| `selectByCondition(query)` | `GET /students?name=...&grade=...` |
| `selectByGrade("五年級")` | `GET /students?grade=五年級` |
| `selectActiveOnly()` | `GET /students?status=active` |
| `selectIncludingGraduated()` | `GET /students?includeGraduated=true` |
| `selectAll()` | `GET /students` (無參數) |

### 3. 新增學生

- **Method**: `POST`
- **Path**: `/students`
- **原始 Mapper**: `StudentMapper.insert(Student)`
- **Request**: `CreateStudentRequest`
- **Response**: `StudentResponse`

## DTO 定義

### StudentResponse

```java
{
  "id": 1001,
  "name": "張三",
  "grade": "五年級",
  "email": "zhangsan@example.com",
  "createTime": "2026-02-11T10:30:00"
}
```

### CreateStudentRequest

```java
{
  "name": "李四",
  "grade": "三年級",
  "email": "lisi@example.com"
}
```
````

**同時輸出**: `_mappers/StudentMapper.md`（Mapper 追溯視角）

```markdown
# StudentMapper 對應表

本文件記錄 `StudentMapper` 所有方法與對應 API 端點的對照關係。

## Mapper 資訊

- **類別名稱**: `com.tutorabc.product.studyprogress.dal_products.DBMapper.StudentMapper`
- **類型**: MyBatis XML Mapper
- **方法總數**: 8
- **對應資源**: [student.md](student.md)

## 方法對應表

| 原始方法                 | 參數         | 返回類型            | 對應 API 端點                     | HTTP 方法 | 還原方式                     |
| ------------------------ | ------------ | ------------------- | --------------------------------- | --------- | ---------------------------- |
| selectById               | Long id      | Student             | `/students/{id}`                  | GET       | 直接對應                     |
| selectByCondition        | StudentQuery | List&lt;Student&gt; | `/students?name=...&grade=...`    | GET       | 傳入查詢參數                 |
| selectByGrade            | String grade | List&lt;Student&gt; | `/students?grade={grade}`         | GET       | 只傳 grade 參數              |
| selectActiveOnly         | -            | List&lt;Student&gt; | `/students?status=active`         | GET       | 固定傳 status=active         |
| selectIncludingGraduated | -            | List&lt;Student&gt; | `/students?includeGraduated=true` | GET       | 固定傳 includeGraduated=true |
| selectAll                | -            | List&lt;Student&gt; | `/students`                       | GET       | 不傳任何參數                 |
| insert                   | Student      | int                 | `/students`                       | POST      | 直接對應                     |
| update                   | Student      | int                 | `/students/{id}`                  | PUT       | 直接對應                     |
| deleteById               | Long id      | int                 | `/students/{id}`                  | DELETE    | 直接對應                     |

## 合併說明

### 查詢方法合併（5 個方法 → 1 個 API）

以下 5 個查詢方法已合併為單一 API 端點 `GET /students`：

1. **selectByCondition(StudentQuery)**
   - API 呼叫：`GET /students?name=張&grade=五年級`
   - 說明：傳入完整查詢條件

2. **selectByGrade(String grade)**
   - API 呼叫：`GET /students?grade=五年級`
   - 說明：只需傳入 grade 參數

3. **selectActiveOnly()**
   - API 呼叫：`GET /students?status=active`
   - 說明：固定傳入 status=active

4. **selectIncludingGraduated()**
   - API 呼叫：`GET /students?includeGraduated=true`
   - 說明：固定傳入 includeGraduated=true

5. **selectAll()**
   - API 呼叫：`GET /students`
   - 說明：不傳任何查詢參數

**合併優勢**：

- 減少 API 端點數量（5 → 1）
- 提供更靈活的查詢組合
- 符合 RESTful 設計原則
- 降低 API 維護成本

## 遷移注意事項

### 高複雜度方法

- **selectWithCourses**: 原本使用 JOIN 查詢，改為 API 後可能需要考慮：
  - 使用 `include` 參數設計
  - 或者分別呼叫 `/students/{id}` 和 `/students/{id}/courses`
  - 注意 N+1 查詢問題

### 交易邊界

- **batchInsert**: 原本在同一個 Transaction，改為 API 後需評估是否：
  - API 端提供 batch endpoint 並保證 ACID
  - 或實作補償機制處理部分失敗情境
```

---

### Phase 5: Specification Assembly（KM Ready）

**目標**: 整合所有 API 規格為適合貼到 KM 的完整文件

**輸出 1**: `API-SPEC.md`（完整規格文件，可直接貼到 KM）

````markdown
# Product Study Progress API 規格文件

## 文件資訊

- **版本**: v1.0.0
- **產生日期**: 2026-02-11
- **Base URL**: `https://api.example.com/v1`
- **來源**: 由 SQL Server Mapper 轉換而來

## 目錄

- [1. Student API](#1-student-api)
- [2. Course API](#2-course-api)
- [3. Enrollment API](#3-enrollment-api)
- [4. 通用規範](#4-通用規範)
- [5. 錯誤處理](#5-錯誤處理)
- [6. 附錄：Mapper 對應表](#6-附錄mapper-對應表)

---

## 1. Student API

### 1.1 查詢單一學生

**端點**: `GET /students/{id}`

**路徑參數**:

- `id` (required, integer): 學生 ID

**回應範例**:

```json
{
  "id": 1001,
  "name": "張三",
  "grade": "五年級",
  "email": "zhangsan@example.com",
  "createTime": "2026-02-11T10:30:00"
}
```

**原始 Mapper**: `StudentMapper.selectById(Long id)`

---

### 1.2 條件查詢學生列表

**端點**: `GET /students`

**查詢參數**:

- `name` (optional, string): 學生姓名（模糊搜尋）
- `grade` (optional, string): 年級
- `page` (optional, integer, default: 0): 頁碼
- `size` (optional, integer, default: 20): 每頁筆數

**回應範例**:

```json
{
  "content": [
    {
      "id": 1001,
      "name": "張三",
      "grade": "五年級",
      "email": "zhangsan@example.com"
    }
  ],
  "totalElements": 100,
  "totalPages": 5,
  "page": 0,
  "size": 20
}
```

**原始 Mapper**: `StudentMapper.selectByCondition(StudentQuery query)`

---

## 4. 通用規範

### 4.1 認證與授權

所有 API 請求需在 Header 加入 Bearer Token：

```
Authorization: Bearer {access_token}
```

### 4.2 分頁規範

使用 `page` 和 `size` 參數，頁碼從 0 開始。

### 4.3 日期時間格式

統一使用 ISO 8601 格式：`YYYY-MM-DDTHH:mm:ss`

---

## 5. 錯誤處理

### 5.1 錯誤格式

```json
{
  "error": {
    "code": "STUDENT_NOT_FOUND",
    "message": "找不到指定的學生",
    "timestamp": "2026-02-11T10:30:00"
  }
}
```

### 5.2 常見錯誤碼

| HTTP 狀態碼 | 錯誤碼                | 說明           |
| ----------- | --------------------- | -------------- |
| 400         | INVALID_PARAMETER     | 參數格式錯誤   |
| 401         | UNAUTHORIZED          | 未授權         |
| 404         | RESOURCE_NOT_FOUND    | 資源不存在     |
| 500         | INTERNAL_SERVER_ERROR | 伺服器內部錯誤 |

---

## 6. 附錄：Mapper 對應表

提供給開發人員遷移時參考。

### 6.1 完整對應表

| 原始 Mapper   | 原始方法                 | 對應 API 端點                   | HTTP 方法 | 還原方式                   |
| ------------- | ------------------------ | ------------------------------- | --------- | -------------------------- |
| StudentMapper | selectById               | /students/{id}                  | GET       | 直接對應                   |
| StudentMapper | selectByCondition        | /students?name=...&grade=...    | GET       | 傳入查詢參數               |
| StudentMapper | selectByGrade            | /students?grade={grade}         | GET       | 只傳 grade                 |
| StudentMapper | selectActiveOnly         | /students?status=active         | GET       | 固定 status=active         |
| StudentMapper | selectIncludingGraduated | /students?includeGraduated=true | GET       | 固定 includeGraduated=true |
| StudentMapper | selectAll                | /students                       | GET       | 不傳參數                   |
| StudentMapper | insert                   | /students                       | POST      | 直接對應                   |
| CourseMapper  | selectById               | /courses/{id}                   | GET       | 直接對應                   |

### 6.2 合併方法說明

以下 Mapper 方法已合併為單一 API 端點，通過不同參數組合實現原有功能：

#### Student 查詢方法合併（5 → 1）

**合併後 API**: `GET /students`

| 原始方法                   | 等效 API 呼叫                                 | 說明                         |
| -------------------------- | --------------------------------------------- | ---------------------------- |
| selectAll()                | `GET /students`                               | 不傳任何參數                 |
| selectByGrade(grade)       | `GET /students?grade=五年級`                  | 只傳 grade 參數              |
| selectActiveOnly()         | `GET /students?status=active`                 | 固定傳 status=active         |
| selectIncludingGraduated() | `GET /students?includeGraduated=true`         | 固定傳 includeGraduated=true |
| selectByCondition(query)   | `GET /students?name=...&grade=...&status=...` | 傳入多個查詢條件             |

**合併優勢**: 減少 API 端點、提供更靈活的查詢組合、符合 RESTful 設計原則

### 6.3 詳細對應文件

詳細的 Mapper 方法對應、參數說明、使用範例，請參閱 `_mappers/` 目錄下的個別文件。
````

**輸出 2**: `openapi-spec.yaml`（技術整合用，可導入 Postman/Swagger）

```yaml
openapi: 3.0.3
info:
  title: Product Study Progress API
  version: 1.0.0
  description: 由 SQL Server Mapper 轉換而來的 RESTful API

servers:
  - url: https://api.example.com/v1
    description: Production
  - url: https://api-dev.example.com/v1
    description: Development

paths:
  /students/{id}:
    get:
      summary: 查詢單一學生
      tags: [Student]
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: integer
            format: int64
      responses:
        "200":
          description: Success
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/StudentResponse"
        "404":
          description: Student not found

  /students:
    get:
      summary: 條件查詢學生列表
      tags: [Student]
      parameters:
        - name: name
          in: query
          schema:
            type: string
        - name: grade
          in: query
          schema:
            type: string
        - name: page
          in: query
          schema:
            type: integer
            default: 0
        - name: size
          in: query
          schema:
            type: integer
            default: 20
      responses:
        "200":
          description: Success
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/PagedStudentResponse"

components:
  schemas:
    StudentResponse:
      type: object
      properties:
        id:
          type: integer
          format: int64
        name:
          type: string
        grade:
          type: string
        email:
          type: string
        createTime:
          type: string
          format: date-time

    PagedStudentResponse:
      type: object
      properties:
        content:
          type: array
          items:
            $ref: "#/components/schemas/StudentResponse"
        totalElements:
          type: integer
        totalPages:
          type: integer
        page:
          type: integer
        size:
          type: integer
```

---

## Skill 執行範例

### 使用者觸發

```
請幫我分析 D:\workspace\oms\OMS_StudyProgress\dal_products\src\main\java\com\tutorabc\product\studyprogress\dal_products\DBMapper
這個路徑的所有 Mapper，並生成對應的 API 規格
```

### Agent 回應

```
Phase 1: 掃描 Mapper 檔案...
   發現 15 個 Mapper (8 個 MyBatis, 7 個 JPA Repository)
   共 47 個方法

Phase 2: 分析方法合併機會...
   識別到 6 組可合併的方法
   合併前：47 個方法 → 47 個 API
   合併後：47 個方法 → 28 個 API
   減少 19 個 API 端點

   範例合併：
   - SessionTypeMapper.selectByType/selectByStatus/selectAll → GET /session-types?type=...&status=...
   - StudentMapper.selectByGrade/selectActiveOnly/selectAll → GET /students?grade=...&status=...

Phase 3: 深度分析中...（平行執行 15 個 Agent）
   └─ StudentMapper: 8 methods → 3 API endpoints (合併 5 個查詢方法)
   └─ CourseMapper: 12 methods → 8 API endpoints (合併 4 個查詢方法)
   └─ SessionTypeMapper: 3 methods → 1 API endpoint (完全合併)
   └─ ...

Phase 4: 生成 API 設計文件...
   資源視角: student.md, course.md, enrollment.md, session-type.md
   Mapper 視角: _mappers/StudentMapper.md, _mappers/CourseMapper.md, ...
   （含合併方法還原對照表）

Phase 5: 整合完整規格文件...
   API-SPEC.md (完整規格文件，可直接貼到 KM)
   - 包含合併方法說明與參數對照
   openapi-spec.yaml (技術整合用，可導入 Postman/Swagger UI)

所有檔案已輸出至 d:\workspace\side\my-agent-skills\mapper-to-api-spec\history\2026-02-11\
```

---

## KM 文件特色

生成的 `API-SPEC.md` 包含：

1. **完整目錄**: 可快速跳轉到各資源 API
2. **清楚範例**: 每個端點都有 Request/Response 範例
3. **通用規範**: 認證、分頁、日期格式等統一說明
4. **錯誤處理**: 完整的錯誤碼與格式說明
5. **Mapper 對應表**: 附錄提供開發人員遷移參考
6. **原始來源標註**: 每個 API 標註對應的原始 Mapper 方法
7. **合併方法說明**: 標註哪些 Mapper 方法已合併，以及如何用參數還原原始行為

---

## 擴充功能（可選）

### 1. 自動生成 Client 程式碼

根據 OpenAPI spec 自動生成：

- Retrofit Interface
- Request/Response DTO
- API Client Implementation

### 2. Postman Collection

- 從 OpenAPI spec 產生 Postman Collection
- 包含所有端點與範例資料
- 可直接匯入測試

---

## 品質標準

### API 設計品質

- 符合 RESTful 規範（資源導向、HTTP 語意正確）
- 統一的命名規範（camelCase/snake_case）
- 完整的錯誤處理（4xx/5xx 錯誤碼）
- 分頁、排序、過濾標準化

### 文件完整性（KM Ready）

- 完整目錄與錨點連結，方便瀏覽
- 每個 API 端點有清楚的說明與範例
- Request/Response 有實際的 JSON 範例
- 列出必填/選填參數與預設值
- 統一的通用規範（認證、分頁、日期格式）
- 完整的錯誤碼對照表
- Mapper 對應表附錄供開發人員參考

### 技術整合性

- OpenAPI 3.0 規格可匯入 Postman/Swagger
- 可用於自動產生 Client 程式碼
- 支援 API 版本控管

---

## 輸出結構

```
mapper-to-api-spec/
├── history/                                # 歷史記錄（按日期分類）
│   ├── 2026-02-11/                         # 日期格式: YYYY-MM-DD
│   │   ├── API-SPEC.md                     # 🎯 完整 API 規格（可貼到 KM）
│   │   ├── openapi-spec.yaml               # Phase 5: OpenAPI 3.0（技術整合用）
│   │   ├── mapper-inventory.json           # Phase 1: Mapper 清單
│   │   ├── _consolidation.json             # Phase 2: 合併分析結果
│   │   ├── _analysis/                      # Phase 3: 詳細分析（內部使用）
│   │   │   ├── StudentMapper.json
│   │   │   ├── CourseMapper.json
│   │   │   └── ...
│   │   ├── _mappers/                       # Phase 4: Mapper 追溯對應表
│   │   │   ├── StudentMapper.md            # StudentMapper 所有方法的 API 對應
│   │   │   ├── CourseMapper.md             # CourseMapper 所有方法的 API 對應
│   │   │   └── ...
│   │   ├── student.md                      # Phase 4: Student API 規格（資源視角）
│   │   ├── course.md                       # Phase 4: Course API 規格（資源視角）
│   │   └── enrollment.md                   # Phase 4: Enrollment API 規格
│   └── 2026-02-10/                         # 其他日期的分析記錄
│       └── ...
├── SKILL.md                                # 本文件
└── phases/                                 # 階段執行指引
    ├── 01-discovery.md
    ├── 02-consolidation.md
    ├── 03-analysis.md
    ├── 04-api-design.md
    └── 05-spec-assembly.md
```

### 檔案說明

**核心產出**：

- **API-SPEC.md**: 🎯 完整的 API 規格文件，包含所有資源的端點、範例、錯誤碼、Mapper 對應表（含合併方法說明）。**可直接複製貼到 KM 系統**。
- **openapi-spec.yaml**: OpenAPI 3.0 規格，可匯入 Postman、Swagger UI 或用於程式碼產生器。

**輔助檔案**：

- **資源視角** (`student.md`, `course.md`, ...): 單一資源的完整 API 規格，適合單獨查閱。
- **Mapper 視角** (`_mappers/StudentMapper.md`, ...): Mapper 方法與 API 的對照表，**包含合併方法的參數還原說明**，適合開發人員遷移時快速定位。
- **\_consolidation.json**: 記錄哪些 Mapper 方法被合併、合併理由、參數對應關係。
- **\_analysis/**: 內部分析 JSON 資料，供進階使用者或工具整合時參考。
