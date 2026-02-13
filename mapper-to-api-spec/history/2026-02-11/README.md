# Mapper to API Specification - 2026-02-11

本目錄包含 `dal_products` 專案的 Mapper 轉 API 規格完整文件。

## 目錄結構

```
2026-02-11/
├── README.md                          # 本文件
├── API-SPEC.md                        # API 規格總覽（主要入口）
├── mapper-inventory.json              # Phase 1: Mapper 清單
├── _consolidation.json                # Phase 2: 合併分析結果
│
├── 資源視角 API 規格（12 個）
├── session-type.md
├── activity-rule-details.md
├── activity-rule-heads.md
├── activity-rule-rewards.md
├── cfg-activities.md
├── cfg-products.md
├── client-purchases.md
├── client-temporal-contracts.md
├── contract-change-records.md
├── cfg-contract-activities.md
├── cfg-product-activities.md
└── activity-records.md
│
└── _mappers/                          # Mapper 視角對應文件（12 個）
    ├── SessionTypeMapper.md
    ├── ActivityRuleDetailMapper.md
    ├── ActivityRuleHeadMapper.md
    ├── ActivityRuleRewardMapper.md
    ├── CfgActivityMapper.md
    ├── CfgProductMapper.md
    ├── ClientPurchaseMapper.md
    ├── ClientTemporalContractMapper.md
    ├── ContractChangeRecordMapper.md
    ├── CfgContractActivityMapper.md
    ├── CfgProductActivityMapper.md
    └── ActivityRecordMapper.md
```

## 文件說明

### 核心文件

#### [API-SPEC.md](API-SPEC.md) 
**API 規格總覽** - 主要入口文件

包含內容：
- 所有 API 端點列表
- Mapper 對應表
- 參數還原速查表
- 合併策略說明
- 通用設計規範
- 快速導航

**適合對象**: 所有開發人員、架構師、專案經理

#### [mapper-inventory.json](mapper-inventory.json)
**Mapper 清單** - Phase 1 輸出

包含內容：
- 12 個 Mapper 的完整清單
- 每個 Mapper 的方法數、複雜度分類
- 統計摘要

**適合對象**: 專案經理、技術負責人

#### [_consolidation.json](_consolidation.json)
**合併分析結果** - Phase 2 輸出

包含內容：
- 5 個合併組的詳細資訊
- 每個合併組的合併理由與優勢
- 參數對應關係

**適合對象**: API 設計師、架構師

### 資源視角 API 規格（Resource View）

位於根目錄，共 12 個文件，以資源名稱命名。

**檔案命名**: `{resource-name}.md`

**內容結構**:
- API 端點定義
- 請求/回應格式
- DTO 定義
- Mapper 方法對應表
- 程式碼遷移範例
- 使用範例
- 注意事項

**適合對象**: 
- 前端開發人員（了解如何調用 API）
- 後端開發人員（了解如何實作 API）
- QA 測試人員（了解測試案例）

**範例**: [session-type.md](session-type.md)

### Mapper 視角對應文件（Mapper View）

位於 `_mappers/` 目錄，共 12 個文件，以 Mapper 名稱命名。

**檔案命名**: `{MapperName}.md`

**內容結構**:
- Mapper 資訊
- 方法對應表（原始方法 → API 端點）
- 設計說明（為何這樣設計）
- 程式碼遷移範例
- 注意事項

**適合對象**:
- 後端開發人員（Mapper → API 遷移）
- 維護人員（追溯原始 Mapper 行為）

**範例**: [_mappers/SessionTypeMapper.md](_mappers/SessionTypeMapper.md)

## 使用指南

### 情境 1: 我想了解整體 API 設計

1. 閱讀 [API-SPEC.md](API-SPEC.md)
2. 查看「API 端點列表」和「合併策略說明」章節

### 情境 2: 我要調用某個 API

1. 從 [API-SPEC.md](API-SPEC.md) 的「快速導航」找到對應的資源文件
2. 閱讀該資源的 API 規格文件（例如：[session-type.md](session-type.md)）
3. 參考「使用範例」章節

### 情境 3: 我要遷移 Mapper 程式碼

1. 從 [API-SPEC.md](API-SPEC.md) 的「Mapper 對應表」找到 Mapper 文件
2. 閱讀 `_mappers/{MapperName}.md`（例如：[_mappers/SessionTypeMapper.md](_mappers/SessionTypeMapper.md)）
3. 參考「程式碼遷移範例」章節
4. 查看對應的 API 規格文件了解詳細的請求/回應格式

### 情境 4: 我想知道原本的 Mapper 方法對應到哪個 API

1. 開啟 [API-SPEC.md](API-SPEC.md)
2. 查看「參數還原速查表」章節
3. 找到對應的 Mapper 和方法
4. 查看「API 呼叫」欄位

### 情境 5: 我想了解為什麼某些方法被合併

1. 閱讀 [_consolidation.json](_consolidation.json)
2. 查看對應的合併組
3. 閱讀 `consolidation_reason` 和 `benefits` 欄位
4. 或查看 [API-SPEC.md](API-SPEC.md) 的「合併策略說明」章節

## 雙視角設計說明

本文件採用「雙視角設計」：

### 資源視角（Resource View）

**目的**: 從 API 使用者的角度，了解如何使用 API

**特點**:
- 以資源為中心（例如：session-types, activity-rule-details）
- 強調 API 端點、請求/回應格式
- 提供使用範例
- 適合前端開發、API 調用

### Mapper 視角（Mapper View）

**目的**: 從 Mapper 遷移的角度，了解原始方法如何對應到 API

**特點**:
- 以 Mapper 為中心（例如：SessionTypeMapper）
- 強調方法對應關係、參數轉換
- 提供遷移範例
- 適合後端開發、Mapper 遷移

### 優勢

1. **雙向查詢**: 可以從資源找 Mapper，也可以從 Mapper 找資源
2. **分離關注點**: 不同角色關注不同視角
3. **易於維護**: 各自維護，互不干擾
4. **知識傳承**: 保留 Mapper 設計脈絡

## 統計資訊

- **專案路徑**: `D:\workspace\oms\OMS_StudyProgress\dal_products`
- **掃描日期**: 2026-02-11
- **Mapper 總數**: 12
- **方法總數**: 21
- **API 端點數**: 16（合併後）
- **減少端點數**: 5（23.8% 優化）
- **合併組數**: 5

## 相關文件

- [../SKILL.md](../SKILL.md) - Mapper to API Spec Skill 定義
- [mapper-inventory.json](mapper-inventory.json) - Mapper 清單
- [_consolidation.json](_consolidation.json) - 合併分析

---

**生成時間**: 2026-02-11  
**專案**: OMS StudyProgress - dal_products  
**Skill 版本**: 1.0
