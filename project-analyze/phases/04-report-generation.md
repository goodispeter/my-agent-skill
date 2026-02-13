# Phase 4: Report Generation

生成索引式報告，通過 markdown 鏈接引用章節文件。

> **規範參考**: [../specs/quality-standards.md](../specs/quality-standards.md)

## 設計原則

1. **引用而非嵌入**：主報告通過鏈接引用章節，不覆制內容
2. **索引 + 綜述**：主報告提供導航和高階分析
3. **避免重覆**：綜述來自 consolidation，不重新生成
4. **獨立可讀**：各章節文件可單獨閱讀

## 輸入

```typescript
interface ReportInput {
  output_dir: string;
  config: AnalysisConfig;
  consolidation: {
    quality_score: QualityScore;
    issues: { errors: Issue[]; warnings: Issue[]; info: Issue[] };
    stats: Stats;
    synthesis: string; // consolidation agent 的綜合分析
    section_summaries: Array<{ file: string; summary: string }>;
  };
}
```

## 執行流程

```javascript
// 1. 質量門禁檢查
if (consolidation.issues.errors.length > 0) {
  const response = await AskUserQuestion({
    questions: [
      {
        question: `發現 ${consolidation.issues.errors.length} 個嚴重問題，如何處理？`,
        header: "質量檢查",
        multiSelect: false,
        options: [
          { label: "查看並修覆", description: "顯示問題列表，手動修覆後重試" },
          { label: "忽略繼續", description: "跳過問題檢查，繼續裝配" },
          { label: "終止", description: "停止報告生成" },
        ],
      },
    ],
  });

  if (response === "查看並修覆") {
    return { action: "fix_required", errors: consolidation.issues.errors };
  }
  if (response === "終止") {
    return { action: "abort" };
  }
}

// 2. 生成索引式報告（不讀取章節內容）
const report = generateIndexReport(config, consolidation);

// 3. 寫入最終文件
const fileName = `${config.type.toUpperCase()}-REPORT.md`;
Write(`${outputDir}/${fileName}`, report);
```

## 報告模板

### 通用結構

```markdown
# {報告標題}

> 生成日期：{date}
> 分析範圍：{scope}
> 分析深度：{depth}
> 質量評分：{overall}%

---

## 報告綜述

{consolidation.synthesis - 來自匯總 Agent 的跨章節綜合分析}

---

## 章節索引

| 章節 | 核心發現 | 詳情 |
| ---- | -------- | ---- |

{section_summaries 生成的表格行}

---

## 架構洞察

{從 consolidation 提取的跨模塊關聯分析}

---

## 建議與展望

{consolidation.recommendations - 優先級排序的綜合建議}

---

**附錄**

- [質量報告](./consolidation-summary.md)
- [章節文件目錄](./sections/)
```

### 報告標題映射

| 類型          | 標題             |
| ------------- | ---------------- |
| architecture  | 項目架構設計報告 |
| design        | 項目設計模式報告 |
| methods       | 項目核心方法報告 |
| comprehensive | 項目綜合分析報告 |

## 生成函數

```javascript
function generateIndexReport(config, consolidation) {
  const titles = {
    architecture: "項目架構設計報告",
    design: "項目設計模式報告",
    methods: "項目核心方法報告",
    comprehensive: "項目綜合分析報告",
  };

  const date = new Date().toLocaleDateString("zh-CN");

  // 章節索引表格
  const sectionTable = consolidation.section_summaries
    .map(
      (s) => `| ${s.title} | ${s.summary} | [查看詳情](./sections/${s.file}) |`,
    )
    .join("\n");

  return `# ${titles[config.type]}

> 生成日期：${date}
> 分析範圍：${config.scope}
> 分析深度：${config.depth}
> 質量評分：${consolidation.quality_score.overall}%

---

## 報告綜述

${consolidation.synthesis}

---

## 章節索引

| 章節 | 核心發現 | 詳情 |
|------|----------|------|
${sectionTable}

---

## 架構洞察

${consolidation.cross_analysis || "詳見各章節分析。"}

---

## 建議與展望

${consolidation.recommendations || "詳見質量報告中的改進建議。"}

---

**附錄**

- [質量報告](./consolidation-summary.md)
- [章節文件目錄](./sections/)
`;
}
```

## 輸出結構

```
.workflow/.scratchpad/analyze-{timestamp}/
├── sections/                    # 獨立章節（Phase 3 產出）
│   ├── section-overview.md
│   ├── section-layers.md
│   └── ...
├── consolidation-summary.md     # 質量報告（Phase 3.5 產出）
└── {TYPE}-REPORT.md             # 索引報告（本階段產出）
```

## 與 Phase 3.5 的協作

Phase 3.5 consolidation agent 需要提供：

```typescript
interface ConsolidationOutput {
  // ... 原有字段
  synthesis: string; // 跨章節綜合分析（2-3 段落）
  cross_analysis: string; // 架構級關聯洞察
  recommendations: string; // 優先級排序的建議
  section_summaries: Array<{
    file: string; // 文件名
    title: string; // 章節標題
    summary: string; // 一句話核心發現
  }>;
}
```

## 關鍵變更

| 原設計                     | 新設計                            |
| -------------------------- | --------------------------------- |
| 讀取章節內容並拼接         | 鏈接引用，不讀取內容              |
| 重新生成 Executive Summary | 直接使用 consolidation.synthesis  |
| 嵌入質量評分表格           | 鏈接引用 consolidation-summary.md |
| 主報告包含全部內容         | 主報告僅為索引 + 綜述             |
