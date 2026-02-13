# Phase 1: 需求探索

在開始分析之前，先收集使用者需求。

## 執行流程

### Step 1: 選擇報告類型

```javascript
AskUserQuestion({
  questions: [
    {
      question: "你希望產出哪種類型的專案分析報告？",
      header: "報告類型",
      multiSelect: false,
      options: [
        {
          label: "架構（建議）",
          description: "系統結構、模組關係、分層分析、依賴圖",
        },
        {
          label: "設計",
          description: "設計模式、類別關係、元件互動、抽象化分析",
        },
        {
          label: "方法",
          description: "關鍵演算法、重要程式路徑、核心函式說明與範例",
        },
        {
          label: "完整分析",
          description: "整合以上所有內容，提供完整專案分析",
        },
      ],
    },
  ],
});
```

### Step 2: 選擇深度等級

```javascript
AskUserQuestion({
  questions: [
    {
      question: "你需要哪種分析深度？",
      header: "深度",
      multiSelect: false,
      options: [
        { label: "概覽", description: "高層次理解，適合新手或快速上手" },
        { label: "詳細", description: "深入分析並包含程式碼範例" },
        { label: "深度解析", description: "完整且詳盡的實作層級分析" },
      ],
    },
  ],
});
```

### Step 3: 定義分析範圍

```javascript
AskUserQuestion({
  questions: [
    {
      question: "分析範圍應涵蓋哪些部分？",
      header: "範圍",
      multiSelect: false,
      options: [
        { label: "整個專案", description: "分析完整程式碼庫" },
        { label: "指定模組", description: "專注於特定模組或目錄" },
        { label: "自訂路徑", description: "指定自訂的路徑規則" },
      ],
    },
  ],
});
```

## 分析重點對照表

| 報告類型 | 分析重點                                 |
| -------- | ---------------------------------------- |
| 架構     | 分層結構、模組依賴、進入點、資料流程     |
| 設計     | 設計模式、類別關係、介面契約、狀態管理   |
| 方法     | 核心演算法、關鍵流程、公開 API、複雜邏輯 |
| 分析     | 整合以上全部內容                         |

## 輸出

將設定儲存至 `analysis-config.json`:

```json
{
  "type": "architecture|design|methods|comprehensive",
  "depth": "overview|detailed|deep-dive",
  "scope": "**/*|src/**/*|custom",
  "focus_areas": ["..."]
}
```
