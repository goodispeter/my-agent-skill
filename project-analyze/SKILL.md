---
name: project-analyze
description: Multi-phase iterative project analysis with Mermaid diagrams. Generates architecture reports, design reports, method analysis reports. Use when analyzing codebases, understanding project structure, reviewing architecture, exploring design patterns, or documenting system components. Triggers on "analyze project", "architecture report", "design analysis", "code structure", "system overview".
allowed-tools: Task, AskUserQuestion, Read, Bash, Glob, Grep, Write
---

# Project Analysis Skill

Generate comprehensive project analysis reports through multi-phase iterative workflow.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│  Context-Optimized Architecture                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Phase 1: Requirements      → analysis-config.json              │
│           ↓                                                      │
│  Phase 2: Exploration       → 初步探索，確定範圍                  │
│           ↓                                                      │
│  Phase 3: Parallel Agents   → sections/section-*.md (直接寫MD)   │
│               ↓ 返回簡要JSON                                     │
│  Phase 3.5: Consolidation   → consolidation-summary.md          │
│           Agent                 ↓ 返回質量評分+問題列表          │
│           ↓                                                      │
│  Phase 4: Assembly          → 合並MD + 質量附錄                  │
│           ↓                                                      │
│  Phase 5: Refinement        → 最終報告                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Key Design Principles

1. **Agent 直接輸出 MD**: 避免 JSON → MD 轉換的上下文開銷
2. **簡要返回**: Agent 只返回路徑+摘要，不返回完整內容
3. **匯總 Agent**: 獨立 Agent 負責跨章節問題檢測和質量評分
4. **引用合並**: Phase 4 讀取文件合並，不在上下文中傳遞
5. **段落式描述**: 禁止清單羅列，層層遞進，客觀學術表達

## Execution Flow

```
┌─────────────────────────────────────────────────────────────────┐
│  Phase 1: Requirements Discovery                                │
│  → Read: phases/01-requirements-discovery.md                    │
│  → Collect: report type, depth level, scope, focus areas        │
│  → Output: analysis-config.json                                 │
├─────────────────────────────────────────────────────────────────┤
│  Phase 2: Project Exploration                                   │
│  → Read: phases/02-project-exploration.md                       │
│  → Launch: parallel exploration agents                          │
│  → Output: exploration context for Phase 3                      │
├─────────────────────────────────────────────────────────────────┤
│  Phase 3: Deep Analysis (Parallel Agents)                       │
│  → Read: phases/03-deep-analysis.md                             │
│  → Reference: specs/quality-standards.md                        │
│  → Each Agent: 分析代碼 → 直接寫 sections/section-*.md          │
│  → Return: {"status", "output_file", "summary", "cross_notes"}  │
├─────────────────────────────────────────────────────────────────┤
│  Phase 3.5: Consolidation (New!)                                │
│  → Read: phases/03.5-consolidation.md                           │
│  → Input: Agent 返回的簡要信息 + cross_module_notes             │
│  → Analyze: 一致性/完整性/關聯性/質量檢查                        │
│  → Output: consolidation-summary.md                             │
│  → Return: {"quality_score", "issues", "stats"}                 │
├─────────────────────────────────────────────────────────────────┤
│  Phase 4: Report Generation                                     │
│  → Read: phases/04-report-generation.md                         │
│  → Check: 如有 errors，提示用戶處理                              │
│  → Merge: Executive Summary + sections/*.md + 質量附錄          │
│  → Output: {TYPE}-REPORT.md                                     │
├─────────────────────────────────────────────────────────────────┤
│  Phase 5: Iterative Refinement                                  │
│  → Read: phases/05-iterative-refinement.md                      │
│  → Reference: specs/quality-standards.md                        │
│  → Loop: 發現問題 → 提問 → 修覆 → 重新檢查                       │
└─────────────────────────────────────────────────────────────────┘
```

## Report Types

| Type            | Output                  | Agents | Focus                                   |
| --------------- | ----------------------- | ------ | --------------------------------------- |
| `architecture`  | ARCHITECTURE-REPORT.md  | 5      | System structure, modules, dependencies |
| `design`        | DESIGN-REPORT.md        | 4      | Patterns, classes, interfaces           |
| `methods`       | METHODS-REPORT.md       | 4      | Algorithms, critical paths, APIs        |
| `comprehensive` | COMPREHENSIVE-REPORT.md | All    | All above combined                      |

## Agent Configuration by Report Type

### Architecture Report

| Agent        | Output File             | Section             |
| ------------ | ----------------------- | ------------------- |
| overview     | section-overview.md     | System Overview     |
| layers       | section-layers.md       | Layer Analysis      |
| dependencies | section-dependencies.md | Module Dependencies |
| dataflow     | section-dataflow.md     | Data Flow           |
| entrypoints  | section-entrypoints.md  | Entry Points        |

### Design Report

| Agent      | Output File           | Section             |
| ---------- | --------------------- | ------------------- |
| patterns   | section-patterns.md   | Design Patterns     |
| classes    | section-classes.md    | Class Relationships |
| interfaces | section-interfaces.md | Interface Contracts |
| state      | section-state.md      | State Management    |

### Methods Report

| Agent      | Output File           | Section              |
| ---------- | --------------------- | -------------------- |
| algorithms | section-algorithms.md | Core Algorithms      |
| paths      | section-paths.md      | Critical Code Paths  |
| apis       | section-apis.md       | Public API Reference |
| logic      | section-logic.md      | Complex Logic        |

## Directory Setup

```javascript
// 生成時間戳目錄名
const timestamp = new Date().toISOString().slice(0, 19).replace(/[-:T]/g, "");
const dir = `.workflow/.scratchpad/analyze-${timestamp}`;

// Windows (cmd)
Bash(`mkdir "${dir}\\sections"`);
Bash(`mkdir "${dir}\\iterations"`);

// Unix/macOS
// Bash(`mkdir -p "${dir}/sections" "${dir}/iterations"`);
```

## Output Structure

```
.workflow/.scratchpad/analyze-{timestamp}/
├── analysis-config.json          # Phase 1
├── sections/                     # Phase 3 (Agent 直接寫入)
│   ├── section-overview.md
│   ├── section-layers.md
│   ├── section-dependencies.md
│   └── ...
├── consolidation-summary.md      # Phase 3.5
├── {TYPE}-REPORT.md              # Final Output
└── iterations/                   # Phase 5
    ├── v1.md
    └── v2.md
```

## Reference Documents

| Document                                                                   | Purpose                             |
| -------------------------------------------------------------------------- | ----------------------------------- |
| [phases/01-requirements-discovery.md](phases/01-requirements-discovery.md) | User interaction, config collection |
| [phases/02-project-exploration.md](phases/02-project-exploration.md)       | Initial exploration                 |
| [phases/03-deep-analysis.md](phases/03-deep-analysis.md)                   | Parallel agent analysis             |
| [phases/03.5-consolidation.md](phases/03.5-consolidation.md)               | Cross-section consolidation         |
| [phases/04-report-generation.md](phases/04-report-generation.md)           | Report assembly                     |
| [phases/05-iterative-refinement.md](phases/05-iterative-refinement.md)     | Quality refinement                  |
| [specs/quality-standards.md](specs/quality-standards.md)                   | Quality gates, standards            |
| [specs/writing-style.md](specs/writing-style.md)                           | 段落式學術寫作規範                  |
| [../\_shared/mermaid-utils.md](../_shared/mermaid-utils.md)                | Shared Mermaid utilities            |
