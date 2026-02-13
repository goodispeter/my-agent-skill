# Phase 3: Deep Analysis

並行 Agent 撰寫設計報告章節，返回簡要信息。

> **規範參考**: [../specs/quality-standards.md](../specs/quality-standards.md)
> **寫作風格**: [../specs/writing-style.md](../specs/writing-style.md)

## Exploration → Agent 自動分配

根據 Phase 2 生成的 exploration 文件名自動分配對應的 analysis agent。

### 映射規則

```javascript
// Exploration 角度 → Agent 映射（基於文件名識別，不讀取內容）
const EXPLORATION_TO_AGENT = {
  // Architecture Report 角度
  "layer-structure": "layers",
  "module-dependencies": "dependencies",
  "entry-points": "entrypoints",
  "data-flow": "dataflow",

  // Design Report 角度
  "design-patterns": "patterns",
  "class-relationships": "classes",
  "interface-contracts": "interfaces",
  "state-management": "state",

  // Methods Report 角度
  "core-algorithms": "algorithms",
  "critical-paths": "paths",
  "public-apis": "apis",
  "complex-logic": "logic",

  // Comprehensive 角度
  architecture: "overview",
  patterns: "patterns",
  dependencies: "dependencies",
  "integration-points": "entrypoints",
};

// 從文件名提取角度
function extractAngle(filename) {
  // exploration-layer-structure.json → layer-structure
  const match = filename.match(/exploration-(.+)\.json$/);
  return match ? match[1] : null;
}

// 分配 agent
function assignAgent(explorationFile) {
  const angle = extractAngle(path.basename(explorationFile));
  return EXPLORATION_TO_AGENT[angle] || null;
}

// Agent 配置（用於 buildAgentPrompt）
const AGENT_CONFIGS = {
  overview: {
    role: "首席系統架構師",
    task: '基於代碼庫全貌，撰寫"總體架構"章節，洞察核心價值主張和頂層技術決策',
    focus: "領域邊界與定位、架構範式、核心技術決策、頂層模塊劃分",
    constraint:
      "避免羅列目錄結構，重點闡述設計意圖，包含至少1個 Mermaid 架構圖",
  },
  layers: {
    role: "資深軟件設計師",
    task: '分析系統邏輯分層結構，撰寫"邏輯視點與分層架構"章節',
    focus: "職責分配體系、數據流向與約束、邊界隔離策略、異常處理流",
    constraint: "不要列舉具體文件名，關注層級間契約和隔離藝術",
  },
  dependencies: {
    role: "集成架構專家",
    task: '審視系統外部連接與內部耦合，撰寫"依賴管理與生態集成"章節',
    focus: "外部集成拓撲、核心依賴分析、依賴注入與控制反轉、供應鏈安全",
    constraint: "禁止簡單列出依賴配置，必須分析集成策略和風險控制模型",
  },
  dataflow: {
    role: "數據架構師",
    task: '追蹤系統數據流轉機制，撰寫"數據流與狀態管理"章節',
    focus: "數據入口與出口、數據轉換管道、持久化策略、一致性保障",
    constraint: "關注數據生命周期和形態演變，不要羅列數據庫表結構",
  },
  entrypoints: {
    role: "系統邊界分析師",
    task: '識別系統入口設計和關鍵路徑，撰寫"系統入口與調用鏈"章節',
    focus: "入口類型與職責、請求處理管道、關鍵業務路徑、異常與邊界處理",
    constraint: "關注入口設計哲學，不要逐個列舉所有端點",
  },
  patterns: {
    role: "核心開發規範制定者",
    task: '挖掘代碼中的覆用機制和標準化實踐，撰寫"設計模式與工程規範"章節',
    focus: "架構級模式、通信與並發模式、橫切關注點實現、抽象與覆用策略",
    constraint: "避免教科書式解釋，必須結合項目上下文說明應用場景",
  },
  classes: {
    role: "領域模型設計師",
    task: '分析系統類型體系和領域模型，撰寫"類型體系與領域建模"章節',
    focus: "領域模型設計、繼承與組合策略、職責分配原則、類型安全與約束",
    constraint: "關注建模思想，用 UML 類圖輔助說明核心關系",
  },
  interfaces: {
    role: "契約設計專家",
    task: '分析系統接口設計和抽象層次，撰寫"接口契約與抽象設計"章節',
    focus: "抽象層次設計、契約與實現分離、擴展點設計、版本演進策略",
    constraint: "關注接口設計哲學，不要逐個列舉接口方法簽名",
  },
  state: {
    role: "狀態管理架構師",
    task: '分析系統狀態管理機制，撰寫"狀態管理與生命周期"章節',
    focus: "狀態模型設計、狀態生命周期、並發與一致性、狀態恢覆與容錯",
    constraint: "關注狀態管理設計決策，不要列舉具體變量名",
  },
  algorithms: {
    role: "算法架構師",
    task: '分析系統核心算法設計，撰寫"核心算法與計算模型"章節',
    focus: "算法選型與權衡、計算模型設計、性能與可擴展性、正確性保障",
    constraint: "關注算法思想，用流程圖輔助說明覆雜邏輯",
  },
  paths: {
    role: "性能架構師",
    task: '分析系統關鍵執行路徑，撰寫"關鍵路徑與性能設計"章節',
    focus: "關鍵業務路徑、性能敏感區域、瓶頸識別與緩解、降級與熔斷",
    constraint: "關注路徑設計戰略考量，不要羅列所有代碼執行步驟",
  },
  apis: {
    role: "API 設計規範專家",
    task: '分析系統對外接口設計規範，撰寫"API 設計與規範"章節',
    focus: "API 設計風格、命名與結構規範、版本管理策略、錯誤處理規範",
    constraint: "關注設計規範和一致性，不要逐個列舉所有 API 端點",
  },
  logic: {
    role: "業務邏輯架構師",
    task: '分析系統業務邏輯建模，撰寫"業務邏輯與規則引擎"章節',
    focus: "業務規則建模、決策點設計、邊界條件處理、業務流程編排",
    constraint: "關注業務邏輯組織方式，不要逐行解釋代碼邏輯",
  },
};
```

### 自動發現與分配流程

```javascript
// 1. 發現所有 exploration 文件（僅看文件名）
const explorationFiles = bash(
  `find ${sessionFolder} -name "exploration-*.json" -type f`,
)
  .split("\n")
  .filter((f) => f.trim());

// 2. 按文件名自動分配 agent
const agentAssignments = explorationFiles
  .map((file) => {
    const angle = extractAngle(path.basename(file));
    const agentName = EXPLORATION_TO_AGENT[angle];
    return {
      exploration_file: file,
      angle: angle,
      agent: agentName,
      output_file: `section-${agentName}.md`,
    };
  })
  .filter((a) => a.agent); // 過濾未映射的角度

console.log(`
## Agent Auto-Assignment

Found ${explorationFiles.length} exploration files:
${agentAssignments.map((a) => `- ${a.angle} → ${a.agent} agent`).join("\n")}
`);
```

---

## Agent 執行前置條件

**每個 Agent 接收 exploration 文件路徑，自行讀取內容**：

```javascript
// Agent prompt 中包含文件路徑
// Agent 啟動後的操作順序：
// 1. Read exploration 文件（上下文輸入）
// 2. Read 規範文件
// 3. 執行分析任務
```

規範文件路徑（相對於 skill 根目錄）：

- `specs/quality-standards.md` - 質量標準和檢查清單
- `specs/writing-style.md` - 段落式寫作規範

---

## 通用寫作規範（所有 Agent 共用）

```
[STYLE]
- **語言規範**：使用嚴謹、專業的中文進行技術寫作。僅專業術語（如 Singleton, Middleware, ORM）保留英文原文。
- **敘述視角**：采用完全客觀的第三人稱視角（"上帝視角"）。嚴禁使用"我們"、"開發者"、"用戶"、"你"或"我"。主語應為"系統"、"模塊"、"設計"、"架構"或"該層"。
- **段落結構**：
    - 禁止使用無序列表作為主要敘述方式，必須將觀點融合在連貫的段落中。
    - 采用"論點-論據-結論"的邏輯結構。
    - 善用邏輯連接詞（"因此"、"然而"、"鑒於"、"進而"）來體現設計思路的推演過程。
- **內容深度**：
    - 抽象化：描述"做什麽"和"為什麽這麽做"，而不是"怎麽寫的"。
    - 方法論：強調設計模式、架構原則（如 SOLID、高內聚低耦合）的應用。
    - 非代碼化：除非定義關鍵接口，否則不直接引用代碼。文件引用僅作為括號內的來源標注 (參考: path/to/file)。
```

## Agent 配置

### Architecture Report Agents

| Agent        | 輸出文件                | 關注點                         |
| ------------ | ----------------------- | ------------------------------ |
| overview     | section-overview.md     | 頂層架構、技術決策、設計哲學   |
| layers       | section-layers.md       | 邏輯分層、職責邊界、隔離策略   |
| dependencies | section-dependencies.md | 依賴治理、集成拓撲、風險控制   |
| dataflow     | section-dataflow.md     | 數據流向、轉換機制、一致性保障 |
| entrypoints  | section-entrypoints.md  | 入口設計、調用鏈、異常傳播     |

### Design Report Agents

| Agent      | 輸出文件              | 關注點                         |
| ---------- | --------------------- | ------------------------------ |
| patterns   | section-patterns.md   | 架構模式、通信機制、橫切關注點 |
| classes    | section-classes.md    | 類型體系、繼承策略、職責劃分   |
| interfaces | section-interfaces.md | 契約設計、抽象層次、擴展機制   |
| state      | section-state.md      | 狀態模型、生命周期、並發控制   |

### Methods Report Agents

| Agent      | 輸出文件              | 關注點                             |
| ---------- | --------------------- | ---------------------------------- |
| algorithms | section-algorithms.md | 核心算法思想、覆雜度權衡、優化策略 |
| paths      | section-paths.md      | 關鍵路徑設計、性能敏感點、瓶頸分析 |
| apis       | section-apis.md       | API 設計規範、版本策略、兼容性     |
| logic      | section-logic.md      | 業務邏輯建模、決策機制、邊界處理   |

---

## Agent 返回格式

```typescript
interface AgentReturn {
  status: "completed" | "partial" | "failed";
  output_file: string;
  summary: string; // 50字以內
  cross_module_notes: string[]; // 跨模塊發現
  stats: { diagrams: number };
}
```

---

## Agent 提示詞

### Overview Agent

```javascript
Task({
  subagent_type: "cli-explore-agent",
  run_in_background: false,
  prompt: `
[SPEC]
首先讀取規範文件：
- Read: ${skillRoot}/specs/quality-standards.md
- Read: ${skillRoot}/specs/writing-style.md
嚴格遵循規範中的質量標準和段落式寫作要求。

[ROLE] 首席系統架構師

[TASK]
基於代碼庫的全貌，撰寫《系統架構設計報告》的"總體架構"章節。透過代碼表象，洞察系統的核心價值主張和頂層技術決策。
輸出: ${outDir}/sections/section-overview.md

[STYLE]
- 嚴謹專業的中文技術寫作，專業術語保留英文
- 完全客觀的第三人稱視角，嚴禁"我們"、"開發者"
- 段落式敘述，采用"論點-論據-結論"結構
- 善用邏輯連接詞體現設計推演過程
- 描述"做什麽"和"為什麽"，非"怎麽寫的"
- 不直接引用代碼，文件僅作來源標注

[FOCUS]
- 領域邊界與定位：系統旨在解決什麽核心業務問題？其在更大的技術生態中處於什麽位置？
- 架構範式：采用何種架構風格（分層、六邊形、微服務、事件驅動等）？選擇該範式的根本原因是什麽？
- 核心技術決策：關鍵技術棧的選型依據，這些選型如何支撐系統的非功能性需求（性能、擴展性、維護性）
- 頂層模塊劃分：系統在最高層級被劃分為哪些邏輯單元？它們之間的高層協作機制是怎樣的？

[CONSTRAINT]
- 避免羅列目錄結構
- 重點闡述"設計意圖"而非"現有功能"
- 包含至少1個 Mermaid 架構圖輔助說明

[RETURN JSON]
{"status":"completed","output_file":"section-overview.md","summary":"<50字>","cross_module_notes":[],"stats":{"diagrams":1}}
`,
});
```

### Layers Agent

```javascript
Task({
  subagent_type: "cli-explore-agent",
  run_in_background: false,
  prompt: `
[SPEC]
首先讀取規範文件：
- Read: ${skillRoot}/specs/quality-standards.md
- Read: ${skillRoot}/specs/writing-style.md
嚴格遵循規範中的質量標準和段落式寫作要求。

[ROLE] 資深軟件設計師

[TASK]
分析系統的邏輯分層結構，撰寫《系統架構設計報告》的"邏輯視點與分層架構"章節。重點揭示系統如何通過分層來隔離關注點。
輸出: ${outDir}/sections/section-layers.md

[STYLE]
- 嚴謹專業的中文技術寫作
- 客觀第三人稱視角，主語為"系統"、"該層"、"設計"
- 段落式敘述，禁止無序列表作為主體
- 強調方法論和架構原則的應用

[FOCUS]
- 職責分配體系：系統被劃分為哪幾個邏輯層級？每一層的核心職責和輸入輸出是什麽？
- 數據流向與約束：數據在各層之間是如何流動的？是否存在嚴格的單向依賴規則？
- 邊界隔離策略：各層之間通過何種方式解耦（接口抽象、DTO轉換、依賴注入）？如何防止下層實現細節泄露到上層？
- 異常處理流：異常信息如何在分層結構中傳遞和轉化？

[CONSTRAINT]
- 不要列舉具體的文件名列表
- 關注"層級間的契約"和"隔離的藝術"

[RETURN JSON]
{"status":"completed","output_file":"section-layers.md","summary":"<50字>","cross_module_notes":[],"stats":{}}
`,
});
```

### Dependencies Agent

```javascript
Task({
  subagent_type: "cli-explore-agent",
  run_in_background: false,
  prompt: `
[SPEC]
首先讀取規範文件：
- Read: ${skillRoot}/specs/quality-standards.md
- Read: ${skillRoot}/specs/writing-style.md
嚴格遵循規範中的質量標準和段落式寫作要求。

[ROLE] 集成架構專家

[TASK]
審視系統的外部連接與內部耦合情況，撰寫《系統架構設計報告》的"依賴管理與生態集成"章節。
輸出: ${outDir}/sections/section-dependencies.md

[STYLE]
- 嚴謹專業的中文技術寫作
- 客觀第三人稱視角
- 段落式敘述，邏輯連貫

[FOCUS]
- 外部集成拓撲：系統如何與外部世界（第三方API、數據庫、中間件）交互？采用了何種適配器或防腐層設計來隔離外部變化？
- 核心依賴分析：區分"核心業務依賴"與"基礎設施依賴"。系統對關鍵框架的依賴程度如何？是否存在被鎖定的風險？
- 依賴注入與控制反轉：系統內部模塊間的組裝方式是什麽？是否實現了依賴倒置原則以支持可測試性？
- 供應鏈安全與治理：對於覆雜的依賴樹，系統采用了何種策略來管理版本和兼容性？

[CONSTRAINT]
- 禁止簡單列出依賴配置文件的內容
- 必須分析依賴背後的"集成策略"和"風險控制模型"

[RETURN JSON]
{"status":"completed","output_file":"section-dependencies.md","summary":"<50字>","cross_module_notes":[],"stats":{}}
`,
});
```

### Patterns Agent

```javascript
Task({
  subagent_type: "cli-explore-agent",
  run_in_background: false,
  prompt: `
[SPEC]
首先讀取規範文件：
- Read: ${skillRoot}/specs/quality-standards.md
- Read: ${skillRoot}/specs/writing-style.md
嚴格遵循規範中的質量標準和段落式寫作要求。

[ROLE] 核心開發規範制定者

[TASK]
挖掘代碼中的覆用機制和標準化實踐，撰寫《系統架構設計報告》的"設計模式與工程規範"章節。
輸出: ${outDir}/sections/section-patterns.md

[STYLE]
- 嚴謹專業的中文技術寫作
- 客觀第三人稱視角
- 段落式敘述，結合項目上下文

[FOCUS]
- 架構級模式：識別系統中廣泛使用的架構模式（CQRS、Event Sourcing、Repository Pattern、Unit of Work）。闡述引入這些模式解決了什麽特定難題
- 通信與並發模式：分析組件間的通信機制（同步/異步、觀察者模式、發布訂閱）以及並發控制策略
- 橫切關注點實現：系統如何統一處理日志、鑒權、緩存、事務管理等橫切邏輯（AOP、中間件管道、裝飾器）？
- 抽象與覆用策略：分析基類、泛型、工具類的設計思想，系統如何通過抽象來減少重覆代碼並提高一致性？

[CONSTRAINT]
- 避免教科書式地解釋設計模式定義，必須結合當前項目上下文說明其應用場景
- 關注"解決類問題的通用機制"

[RETURN JSON]
{"status":"completed","output_file":"section-patterns.md","summary":"<50字>","cross_module_notes":[],"stats":{}}
`,
});
```

### DataFlow Agent

```javascript
Task({
  subagent_type: "cli-explore-agent",
  run_in_background: false,
  prompt: `
[SPEC]
首先讀取規範文件：
- Read: ${skillRoot}/specs/quality-standards.md
- Read: ${skillRoot}/specs/writing-style.md
嚴格遵循規範中的質量標準和段落式寫作要求。

[ROLE] 數據架構師

[TASK]
追蹤系統的數據流轉機制，撰寫《系統架構設計報告》的"數據流與狀態管理"章節。
輸出: ${outDir}/sections/section-dataflow.md

[STYLE]
- 嚴謹專業的中文技術寫作
- 客觀第三人稱視角
- 段落式敘述

[FOCUS]
- 數據入口與出口：數據從何處進入系統，最終流向何處？邊界處的數據校驗和轉換策略是什麽？
- 數據轉換管道：數據在各層/模塊間經歷了怎樣的形態變化？DTO、Entity、VO 等數據對象的職責邊界如何劃分？
- 持久化策略：系統如何設計數據存儲方案？采用了何種 ORM 策略或數據訪問模式？
- 一致性保障：系統如何處理事務邊界？分布式場景下如何保證數據一致性？

[CONSTRAINT]
- 關注數據的"生命周期"和"形態演變"
- 不要羅列數據庫表結構

[RETURN JSON]
{"status":"completed","output_file":"section-dataflow.md","summary":"<50字>","cross_module_notes":[],"stats":{}}
`,
});
```

### EntryPoints Agent

```javascript
Task({
  subagent_type: "cli-explore-agent",
  run_in_background: false,
  prompt: `
[SPEC]
首先讀取規範文件：
- Read: ${skillRoot}/specs/quality-standards.md
- Read: ${skillRoot}/specs/writing-style.md
嚴格遵循規範中的質量標準和段落式寫作要求。

[ROLE] 系統邊界分析師

[TASK]
識別系統的入口設計和關鍵路徑，撰寫《系統架構設計報告》的"系統入口與調用鏈"章節。
輸出: ${outDir}/sections/section-entrypoints.md

[STYLE]
- 嚴謹專業的中文技術寫作
- 客觀第三人稱視角
- 段落式敘述

[FOCUS]
- 入口類型與職責：系統提供了哪些類型的入口（REST API、CLI、消息隊列消費者、定時任務）？各入口的設計目的和適用場景是什麽？
- 請求處理管道：從入口到核心邏輯，請求經過了怎樣的處理管道？中間件/攔截器的編排邏輯是什麽？
- 關鍵業務路徑：最重要的幾條業務流程的調用鏈是怎樣的？關鍵節點的設計考量是什麽？
- 異常與邊界處理：系統如何統一處理異常？異常信息如何傳播和轉化？

[CONSTRAINT]
- 關注"入口的設計哲學"而非 API 清單
- 不要逐個列舉所有端點

[RETURN JSON]
{"status":"completed","output_file":"section-entrypoints.md","summary":"<50字>","cross_module_notes":[],"stats":{}}
`,
});
```

### Classes Agent

```javascript
Task({
  subagent_type: "cli-explore-agent",
  run_in_background: false,
  prompt: `
[SPEC]
首先讀取規範文件：
- Read: ${skillRoot}/specs/quality-standards.md
- Read: ${skillRoot}/specs/writing-style.md
嚴格遵循規範中的質量標準和段落式寫作要求。

[ROLE] 領域模型設計師

[TASK]
分析系統的類型體系和領域模型，撰寫《系統架構設計報告》的"類型體系與領域建模"章節。
輸出: ${outDir}/sections/section-classes.md

[STYLE]
- 嚴謹專業的中文技術寫作
- 客觀第三人稱視角
- 段落式敘述

[FOCUS]
- 領域模型設計：系統的核心領域概念有哪些？它們之間的關系如何建模（聚合、實體、值對象）？
- 繼承與組合策略：系統傾向於使用繼承還是組合？基類/接口的設計意圖是什麽？
- 職責分配原則：類的職責劃分遵循了什麽原則？是否體現了單一職責原則？
- 類型安全與約束：系統如何利用類型系統來表達業務約束和不變量？

[CONSTRAINT]
- 關注"建模思想"而非類的屬性列表
- 用 UML 類圖輔助說明核心關系

[RETURN JSON]
{"status":"completed","output_file":"section-classes.md","summary":"<50字>","cross_module_notes":[],"stats":{}}
`,
});
```

### Interfaces Agent

```javascript
Task({
  subagent_type: "cli-explore-agent",
  run_in_background: false,
  prompt: `
[SPEC]
首先讀取規範文件：
- Read: ${skillRoot}/specs/quality-standards.md
- Read: ${skillRoot}/specs/writing-style.md
嚴格遵循規範中的質量標準和段落式寫作要求。

[ROLE] 契約設計專家

[TASK]
分析系統的接口設計和抽象層次，撰寫《系統架構設計報告》的"接口契約與抽象設計"章節。
輸出: ${outDir}/sections/section-interfaces.md

[STYLE]
- 嚴謹專業的中文技術寫作
- 客觀第三人稱視角
- 段落式敘述

[FOCUS]
- 抽象層次設計：系統定義了哪些核心接口/抽象類？這些抽象的設計意圖和職責邊界是什麽？
- 契約與實現分離：接口如何隔離契約與實現？多態機制如何被運用？
- 擴展點設計：系統預留了哪些擴展點？如何在不修改核心代碼的情況下擴展功能？
- 版本演進策略：接口如何支持版本演進？向後兼容性如何保障？

[CONSTRAINT]
- 關注"接口的設計哲學"
- 不要逐個列舉接口方法簽名

[RETURN JSON]
{"status":"completed","output_file":"section-interfaces.md","summary":"<50字>","cross_module_notes":[],"stats":{}}
`,
});
```

### State Agent

```javascript
Task({
  subagent_type: "cli-explore-agent",
  run_in_background: false,
  prompt: `
[SPEC]
首先讀取規範文件：
- Read: ${skillRoot}/specs/quality-standards.md
- Read: ${skillRoot}/specs/writing-style.md
嚴格遵循規範中的質量標準和段落式寫作要求。

[ROLE] 狀態管理架構師

[TASK]
分析系統的狀態管理機制，撰寫《系統架構設計報告》的"狀態管理與生命周期"章節。
輸出: ${outDir}/sections/section-state.md

[STYLE]
- 嚴謹專業的中文技術寫作
- 客觀第三人稱視角
- 段落式敘述

[FOCUS]
- 狀態模型設計：系統需要管理哪些類型的狀態（會話狀態、應用狀態、領域狀態）？狀態的存儲位置和作用域是什麽？
- 狀態生命周期：狀態如何創建、更新、銷毀？生命周期管理的機制是什麽？
- 並發與一致性：多線程/多實例場景下，狀態如何保持一致？采用了何種並發控制策略？
- 狀態恢覆與容錯：系統如何處理狀態丟失或損壞？是否有狀態恢覆機制？

[CONSTRAINT]
- 關注"狀態管理的設計決策"
- 不要列舉具體的變量名

[RETURN JSON]
{"status":"completed","output_file":"section-state.md","summary":"<50字>","cross_module_notes":[],"stats":{}}
`,
});
```

### Algorithms Agent

```javascript
Task({
  subagent_type: "cli-explore-agent",
  run_in_background: false,
  prompt: `
[SPEC]
首先讀取規範文件：
- Read: ${skillRoot}/specs/quality-standards.md
- Read: ${skillRoot}/specs/writing-style.md
嚴格遵循規範中的質量標準和段落式寫作要求。

[ROLE] 算法架構師

[TASK]
分析系統的核心算法設計，撰寫《系統架構設計報告》的"核心算法與計算模型"章節。
輸出: ${outDir}/sections/section-algorithms.md

[STYLE]
- 嚴謹專業的中文技術寫作
- 客觀第三人稱視角
- 段落式敘述

[FOCUS]
- 算法選型與權衡：系統的核心業務邏輯采用了哪些關鍵算法？選擇這些算法的考量因素是什麽（時間覆雜度、空間覆雜度、可維護性）？
- 計算模型設計：覆雜計算如何被分解和組織？是否采用了流水線、Map-Reduce 等計算模式？
- 性能與可擴展性：算法設計如何考慮性能和可擴展性？是否有針對大數據量的優化策略？
- 正確性保障：關鍵算法的正確性如何保障？是否有邊界條件的特殊處理？

[CONSTRAINT]
- 關注"算法思想"而非具體實現代碼
- 用流程圖輔助說明覆雜邏輯

[RETURN JSON]
{"status":"completed","output_file":"section-algorithms.md","summary":"<50字>","cross_module_notes":[],"stats":{}}
`,
});
```

### Paths Agent

```javascript
Task({
  subagent_type: "cli-explore-agent",
  run_in_background: false,
  prompt: `
[SPEC]
首先讀取規範文件：
- Read: ${skillRoot}/specs/quality-standards.md
- Read: ${skillRoot}/specs/writing-style.md
嚴格遵循規範中的質量標準和段落式寫作要求。

[ROLE] 性能架構師

[TASK]
分析系統的關鍵執行路徑，撰寫《系統架構設計報告》的"關鍵路徑與性能設計"章節。
輸出: ${outDir}/sections/section-paths.md

[STYLE]
- 嚴謹專業的中文技術寫作
- 客觀第三人稱視角
- 段落式敘述

[FOCUS]
- 關鍵業務路徑：系統中最重要的幾條業務執行路徑是什麽？這些路徑的設計目標和約束是什麽？
- 性能敏感區域：哪些環節是性能敏感的？系統采用了何種優化策略（緩存、異步、批處理）？
- 瓶頸識別與緩解：潛在的性能瓶頸在哪里？設計中是否預留了擴展空間？
- 降級與熔斷：在高負載或故障場景下，系統如何保護關鍵路徑？

[CONSTRAINT]
- 關注"路徑設計的戰略考量"
- 不要羅列所有代碼執行步驟

[RETURN JSON]
{"status":"completed","output_file":"section-paths.md","summary":"<50字>","cross_module_notes":[],"stats":{}}
`,
});
```

### APIs Agent

```javascript
Task({
  subagent_type: "cli-explore-agent",
  run_in_background: false,
  prompt: `
[SPEC]
首先讀取規範文件：
- Read: ${skillRoot}/specs/quality-standards.md
- Read: ${skillRoot}/specs/writing-style.md
嚴格遵循規範中的質量標準和段落式寫作要求。

[ROLE] API 設計規範專家

[TASK]
分析系統的對外接口設計規範，撰寫《系統架構設計報告》的"API 設計與規範"章節。
輸出: ${outDir}/sections/section-apis.md

[STYLE]
- 嚴謹專業的中文技術寫作
- 客觀第三人稱視角
- 段落式敘述

[FOCUS]
- API 設計風格：系統采用了何種 API 設計風格（RESTful、GraphQL、RPC）？選擇該風格的原因是什麽？
- 命名與結構規範：API 的命名、路徑結構、參數設計遵循了什麽規範？是否有一致性保障機制？
- 版本管理策略：API 如何支持版本演進？向後兼容性策略是什麽？
- 錯誤處理規範：API 錯誤響應的設計規範是什麽？錯誤碼體系如何組織？

[CONSTRAINT]
- 關注"設計規範和一致性"
- 不要逐個列舉所有 API 端點

[RETURN JSON]
{"status":"completed","output_file":"section-apis.md","summary":"<50字>","cross_module_notes":[],"stats":{}}
`,
});
```

### Logic Agent

```javascript
Task({
  subagent_type: "cli-explore-agent",
  run_in_background: false,
  prompt: `
[SPEC]
首先讀取規範文件：
- Read: ${skillRoot}/specs/quality-standards.md
- Read: ${skillRoot}/specs/writing-style.md
嚴格遵循規範中的質量標準和段落式寫作要求。

[ROLE] 業務邏輯架構師

[TASK]
分析系統的業務邏輯建模，撰寫《系統架構設計報告》的"業務邏輯與規則引擎"章節。
輸出: ${outDir}/sections/section-logic.md

[STYLE]
- 嚴謹專業的中文技術寫作
- 客觀第三人稱視角
- 段落式敘述

[FOCUS]
- 業務規則建模：核心業務規則如何被表達和組織？是否采用了規則引擎或策略模式？
- 決策點設計：系統中的關鍵決策點有哪些？決策邏輯如何被封裝和測試？
- 邊界條件處理：系統如何處理邊界條件和異常情況？是否有防御性編程措施？
- 業務流程編排：覆雜業務流程如何被編排？是否采用了工作流引擎或狀態機？

[CONSTRAINT]
- 關注"業務邏輯的組織方式"
- 不要逐行解釋代碼邏輯

[RETURN JSON]
{"status":"completed","output_file":"section-logic.md","summary":"<50字>","cross_module_notes":[],"stats":{}}
`,
});
```

---

## 執行流程

```javascript
// 1. 發現 exploration 文件並自動分配 agent
const explorationFiles = bash(
  `find ${sessionFolder} -name "exploration-*.json" -type f`,
)
  .split("\n")
  .filter((f) => f.trim());

const agentAssignments = explorationFiles
  .map((file) => {
    const angle = extractAngle(path.basename(file));
    const agentName = EXPLORATION_TO_AGENT[angle];
    return { exploration_file: file, angle, agent: agentName };
  })
  .filter((a) => a.agent);

// 2. 準備目錄
Bash(`mkdir "${outputDir}\\sections"`);

// 3. 並行啟動所有 Agent（傳遞 exploration 文件路徑）
const results = await Promise.all(
  agentAssignments.map((assignment) =>
    Task({
      subagent_type: "cli-explore-agent",
      run_in_background: false,
      description: `Analyze: ${assignment.agent}`,
      prompt: buildAgentPrompt(assignment, config, outputDir),
    }),
  ),
);

// 4. 收集簡要返回信息
const summaries = results.map((r) => JSON.parse(r));

// 5. 傳遞給 Phase 3.5 匯總 Agent
return {
  summaries,
  cross_notes: summaries.flatMap((s) => s.cross_module_notes),
};
```

### Agent Prompt 構建

```javascript
function buildAgentPrompt(assignment, config, outputDir) {
  const agentConfig = AGENT_CONFIGS[assignment.agent];
  return `
[CONTEXT]
**Exploration 文件**: ${assignment.exploration_file}
首先讀取此文件獲取 ${assignment.angle} 探索結果作為分析上下文。

[SPEC]
讀取規範文件：
- Read: ${skillRoot}/specs/quality-standards.md
- Read: ${skillRoot}/specs/writing-style.md

[ROLE] ${agentConfig.role}

[TASK]
${agentConfig.task}
輸出: ${outputDir}/sections/section-${assignment.agent}.md

[STYLE]
- 嚴謹專業的中文技術寫作，專業術語保留英文
- 完全客觀的第三人稱視角，嚴禁"我們"、"開發者"
- 段落式敘述，采用"論點-論據-結論"結構
- 善用邏輯連接詞體現設計推演過程

[FOCUS]
${agentConfig.focus}

[CONSTRAINT]
${agentConfig.constraint}

[RETURN JSON]
{"status":"completed","output_file":"section-${assignment.agent}.md","summary":"<50字>","cross_module_notes":[],"stats":{}}
`;
}
```

## Output

各 Agent 寫入 `sections/section-xxx.md`，返回簡要 JSON 供 Phase 3.5 匯總。
