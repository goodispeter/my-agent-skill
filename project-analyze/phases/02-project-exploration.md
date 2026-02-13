# Phase 2: 專案探索

根據報告類型與任務情境，啟動平行探索代理。

## 執行流程

### Step 1: 智慧角度選擇

```javascript
// 根據報告類型的角度預設（改編自 lite-plan.md）
const ANGLE_PRESETS = {
  architecture: [
    "layer-structure",
    "module-dependencies",
    "entry-points",
    "data-flow",
  ],
  design: [
    "design-patterns",
    "class-relationships",
    "interface-contracts",
    "state-management",
  ],
  methods: [
    "core-algorithms",
    "critical-paths",
    "public-apis",
    "complex-logic",
  ],
  comprehensive: [
    "architecture",
    "patterns",
    "dependencies",
    "integration-points",
  ],
};

// 根據深度決定角度數量
const angleCount = {
  shallow: 2,
  standard: 3,
  deep: 4,
};

function selectAngles(reportType, depth) {
  const preset = ANGLE_PRESETS[reportType] || ANGLE_PRESETS.comprehensive;
  const count = angleCount[depth] || 3;
  return preset.slice(0, count);
}

const selectedAngles = selectAngles(config.type, config.depth);

console.log(`
## 探索計畫

報告類型: ${config.type}
深度: ${config.depth}
選擇角度: ${selectedAngles.join(", ")}

啟動  ${selectedAngles.length} 個平行探索任務...
`);
```

### Step 2: 啟動平行代理（直接輸出）

**⚠️ 關鍵**: 代理需直接寫入輸出檔案，不需要額外彙整。

```javascript
// 使用預先分配的角度啟動代理
const explorationTasks = selectedAngles.map((angle, index) =>
  Task({
    subagent_type: "cli-explore-agent",
    run_in_background: false, // ⚠️ 強制：必須等待結果
    description: `Explore: ${angle}`,
    prompt: `
## 探索目標
為 ${config.type} 專案分析報告執行 **${angle}** 面向探索。

## 指派內容
- **探索角度**: ${angle}
- **報告類型**: ${config.type}
- **深度**: ${config.depth}
- **範圍**: ${config.scope}
- **探索序號**: ${index + 1} / ${selectedAngles.length}
- **輸出檔案**: ${sessionFolder}/exploration-${angle}.json

## 強制第一步（代理必須執行）
**你（cli-explore-agent）必須依序執行：**
1. 執行：ccw tool exec get_modules_by_depth '{}'（專案結構）
2. 執行：rg -l "{relevant_keyword}" --type ts（定位相關檔案）
3. 從 ${angle} 角度分析專案

## 探索策略（聚焦 ${angle}）

**步驟 1：結構掃描**（Bash）
- get_modules_by_depth.sh → 找出與 ${angle} 相關模組
- find/rg → 尋找相關檔案
- 從 ${angle} 視角分析 import/依賴關係

**步驟 2：語意分析**（Gemini/Qwen CLI）
- 現有程式如何處理 ${angle}？
- 使用了哪些模式來實作 ${angle}？
- 找出與 ${angle} 相關的關鍵架構決策

**步驟 3：直接寫入輸出**
- 整理 ${angle} 發現為 JSON
- 寫入上方指定輸出路徑

## 預期輸出格式

**檔案**: ${sessionFolder}/exploration-${angle}.json

\`\`\`json
{
  "angle": "${angle}",
  "findings": {
    "structure": [
      { "component": "...", "type": "module|layer|service", "description": "..." }
    ],
    "patterns": [
      { "name": "...", "usage": "...", "files": ["path1", "path2"] }
    ],
    "relationships": [
      { "from": "...", "to": "...", "type": "depends|imports|calls", "strength": "high|medium|low" }
    ],
    "key_files": [
      { "path": "src/file.ts", "relevance": 0.85, "rationale": "Core ${angle} logic" }
    ]
  },
  "insights": [
    { "observation": "...", "impact": "high|medium|low", "recommendation": "..." }
  ],
  "_metadata": {
    "exploration_angle": "${angle}",
    "exploration_index": ${index + 1},
    "report_type": "${config.type}",
    "timestamp": "ISO8601"
  }
}
\`\`\`

## 成功標準
- [ ] 已執行 get_modules_by_depth.sh
- [ ] 至少找到 3 個與 ${angle} 相關的檔案並說明原因
- [ ] 模式可實際應用（有程式碼範例，不是泛泛建議）
- [ ] 關係包含具體檔案參考
- [ ] JSON 輸出寫入 ${sessionFolder}/exploration-${angle}.json
- [ ] 回傳：2–3 句 ${angle} 發現摘要
`,
  }),
);

// Execute all exploration tasks in parallel
```

## 輸出結果

探索完成後的 Session 資料夾結構：

```
${sessionFolder}/
├── exploration-{angle1}.json      # Agent 1 direct output
├── exploration-{angle2}.json      # Agent 2 direct output
├── exploration-{angle3}.json      # Agent 3 direct output (if applicable)
└── exploration-{angle4}.json      # Agent 4 direct output (if applicable)
```

## 後續使用（階段 3 分析輸入）

後續分析階段 必須讀取探索輸出作為輸入：

```javascript
// 依角度名稱讀取探索檔案
const explorationData = {};
selectedAngles.forEach((angle) => {
  const filePath = `${sessionFolder}/exploration-${angle}.json`;
  explorationData[angle] = JSON.parse(Read(filePath));
});

// 傳給分析代理
Task({
  subagent_type: "analysis-agent",
  prompt: `
## 分析輸入

### 各角度探索資料
${Object.entries(explorationData)
  .map(
    ([angle, data]) => `
#### ${angle}
${JSON.stringify(data, null, 2)}
`,
  )
  .join("\n")}

## 分析任務
整合所有探索角度的發現...
`,
});
```
