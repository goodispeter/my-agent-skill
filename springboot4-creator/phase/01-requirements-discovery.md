# Phase 1: 需求探索

在開始生產專案之前，先收集使用者需求。

## 執行流程

### Step 1.0: 前置作業 - 環境檢查

在開始收集需求之前，必須先確認 Maven 環境配置正確。

生成的程式碼，放在 **create-history 資料夾**，路徑格式為：`YYYY-MM-DD/專案名稱`

範例：`2026-02-05/pace-calculator`

**檢查項目：**

1. 確認 Maven 使用的 JDK 版本為 JDK 25
2. 檢查系統預設 Java 版本（可選，但建議）

**執行命令：**

```bash
# 檢查 Maven 使用的 JDK
mvn -version

# 檢查系統預設 Java（選用）
java -version
```

**預期輸出：**

```
# mvn -version 輸出
Apache Maven x.x.x (...)
Maven home: ...
Java version: 25.x.x, vendor: ...
Java home: ...
Default locale: ...
OS name: ...

# java -version 輸出（可能不同）
java version "25.x.x" 或其他版本
```

**驗證重點：**

- **必須檢查**：`mvn -version` 的 `Java version` 必須顯示為 `25.x.x`
- 如果 Maven 使用的 Java 版本不是 25，需要調整 `JAVA_HOME` 環境變數指向 JDK 25 的安裝路徑
- 如果不是 `25.x.x` 則提示使用者調整開發環境，**終止這次指令**

**環境版本不一致的提醒：**

如果 `mvn -version` 顯示 Java 25，但 `java -version` 顯示其他版本（如 1.8），需提醒使用者：

- Maven 編譯和測試會正常執行（使用 JDK 25）
- 直接執行 `java -jar xxx.jar` 可能失敗（因為系統 Java 版本不同）
- 建議使用 `mvn spring-boot:run` 或 `mvn test` 來執行專案

### Step 1.1: 專案基本資訊收集

收集專案的基本資訊，包括專案名稱、package 名稱和描述。

**輸入參數：**

- `projectName`: 專案名稱（例如：pace-calculator）
- `basePackage`: Base package 名稱（例如：com.kuanyu）
- `description`: 專案描述（例如：Activity sync service）
- `serverPort`: 服務埠號（預設：8080）

**範例問題集：**

```javascript
AskUserQuestion({
  questions: [
    {
      header: "專案名稱",
      question: "請輸入專案名稱（小寫，使用 - 分隔）",
      type: "text",
      placeholder: "ex: pace-calculator",
      required: true,
    },
    {
      header: "Base Package",
      question: "Base package 名稱？",
      type: "text",
      placeholder: "ex: com.kuanyu",
      required: true,
    },
    {
      header: "專案描述",
      question: "專案描述",
      type: "text",
      placeholder: "ex: Activity sync service",
      required: true,
    },
    {
      header: "伺服器埠號",
      question: "服務埠號",
      type: "text",
      placeholder: "8080",
      default: "8080",
      required: true,
    },
  ],
});
```

**輸出：**

- 儲存專案基本資訊供後續 phase 使用

---

### Step 1.2: 資料庫配置收集

確認是否需要資料庫連線，如需要則收集資料庫連線資訊。

**輸入參數：**

- `useDatabase`: 是否使用資料庫（boolean）
- `dbUrl`: 資料庫 URL（僅 dev 環境）
- `dbUsername`: 資料庫帳號
- `dbPassword`: 資料庫密碼
- `dbType`: 資料庫類型（postgresql/mysql）

**範例問題集：**

```javascript
AskUserQuestion({
  questions: [
    {
      header: "使用資料庫",
      question: "是否需要連線至資料庫？",
      type: "select",
      options: ["是", "否"],
      required: true,
      id: "useDb",
    },
    {
      header: "資料庫類型",
      question: "資料庫類型",
      type: "select",
      options: ["PostgreSQL", "MySQL"],
      required: false,
      visibleIf: (answers) => answers.useDb === "是",
    },
    {
      header: "資料庫 URL",
      question: "資料庫連線 URL（僅 dev 環境）",
      type: "text",
      placeholder: "ex: jdbc:postgresql://localhost:5432/mydb",
      required: false,
      visibleIf: (answers) => answers.useDb === "是",
    },
    {
      header: "資料庫帳號",
      question: "資料庫連線帳號",
      type: "text",
      placeholder: "ex: postgres",
      required: false,
      visibleIf: (answers) => answers.useDb === "是",
    },
    {
      header: "資料庫密碼",
      question: "資料庫連線密碼",
      type: "password",
      required: false,
      visibleIf: (answers) => answers.useDb === "是",
    },
  ],
});
```

**輸出：**

- 資料庫配置資訊

---

### Step 1.3: 功能模組定義

定義專案需要的主要功能模組（實體）。

**輸入參數：**

- `modules`: 功能模組列表（例如：User,Product,Order）

**範例問題集：**

```javascript
AskUserQuestion({
  questions: [
    {
      header: "功能模組",
      question:
        "請列出需要的主要功能模組（用逗號分隔，例如：User,Product,Order）",
      type: "text",
      placeholder: "ex: User,Product",
      required: false,
    },
  ],
});
```

**處理邏輯：**

- 將逗號分隔的字串轉換為陣列
- 驗證模組名稱符合 Java 類別命名規範
- 儲存模組列表供 Phase 4 使用

**輸出：**

- 模組列表陣列

---

### Step 1.4: 外部 API 配置

確認是否需要呼叫外部 API，如需要則收集 API 基本資訊。

**輸入參數：**

- `useExternalApi`: 是否使用外部 API（boolean）
- `apiName`: 外部 API 名稱（例如：Strava, GitHub）
- `apiBaseUrl`: 外部 API Base URL

**範例問題集：**

```javascript
AskUserQuestion({
  questions: [
    {
      header: "外部 API",
      question: "是否需要呼叫外部 API？",
      type: "select",
      options: ["是", "否"],
      required: true,
      id: "useApi",
    },
    {
      header: "API 名稱",
      question: "外部 API 名稱（例如：Strava）",
      type: "text",
      placeholder: "ex: Strava",
      required: false,
      visibleIf: (answers) => answers.useApi === "是",
    },
    {
      header: "API Base URL",
      question: "外部 API Base URL",
      type: "text",
      placeholder: "ex: https://www.strava.com",
      required: false,
      visibleIf: (answers) => answers.useApi === "是",
    },
  ],
});
```

**輸出：**

- 外部 API 配置資訊

---

### Step 1.5: 需求總結確認

將收集到的所有需求整理並顯示給使用者確認。

**輸出格式：**

```
專案配置總結
================
專案名稱: pace-calculator
Base Package: com.kuanyu
描述: Running pace calculator
伺服器埠號: 8080

資料庫配置:
- 使用資料庫: 是
- 資料庫類型: PostgreSQL
- 連線 URL: jdbc:postgresql://localhost:5432/mydb

功能模組:
- User
- Activity

外部 API:
- 使用外部 API: 是
- API 名稱: Strava
- Base URL: https://www.strava.com
================

請確認以上配置是否正確？
```

**輸出：**

- 完整的需求配置物件，供後續 phase 使用
