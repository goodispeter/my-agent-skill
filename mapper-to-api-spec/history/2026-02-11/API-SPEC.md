# API 規格總覽

**專案**: OMS StudyProgress - dal_products  
**生成日期**: 2026-02-11  
**Mapper 總數**: 12  
**API 端點總數**: 16（已合併 5 個）

## 目錄

1. [概述](#概述)
2. [API 端點列表](#api-端點列表)
3. [Mapper 對應表](#mapper-對應表)
4. [合併策略說明](#合併策略說明)
5. [通用設計規範](#通用設計規範)
6. [快速導航](#快速導航)

## 概述

本文件整合了 dal_products 專案中所有 Mapper 轉換為 RESTful API 的規格說明。共涵蓋 12 個 Mapper，21 個方法，經過合併優化後產生 16 個 API 端點。

### 統計摘要

- **原始 Mapper 方法數**: 21
- **合併後 API 數**: 16
- **減少端點數**: 5（23.8% 優化）
- **合併組數**: 5

## API 端點列表

### 1. Session Types

| 端點 | 方法 | 說明 | 文件 |
|------|------|------|------|
| `/api/v1/session-types` | GET | 查詢會話類型（支援 webSite, sourceSn 篩選） | [session-type.md](session-type.md) |

**合併**: 3 個 Mapper 方法 → 1 個 API

### 2. Activity Rules

| 端點 | 方法 | 說明 | 文件 |
|------|------|------|------|
| `/api/v1/activity-rule-details` | GET | 查詢活動規則明細（支援 levelNo 篩選） | [activity-rule-details.md](activity-rule-details.md) |
| `/api/v1/activity-rule-heads` | GET | 查詢活動規則主檔（支援 activitySn 篩選） | [activity-rule-heads.md](activity-rule-heads.md) |
| `/api/v1/activity-rule-rewards` | GET | 查詢活動規則獎勵（支援 levelNo 篩選） | [activity-rule-rewards.md](activity-rule-rewards.md) |

**合併**: 每個端點各合併 2 個 Mapper 方法

### 3. Configuration Management

| 端點 | 方法 | 說明 | 文件 |
|------|------|------|------|
| `/api/v1/cfg-activities` | GET | 查詢全部活動配置 | [cfg-activities.md](cfg-activities.md) |
| `/api/v1/cfg-activities/{sn}` | GET | 查詢單一活動配置 | [cfg-activities.md](cfg-activities.md) |
| `/api/v1/cfg-products` | GET | 查詢全部產品配置 | [cfg-products.md](cfg-products.md) |
| `/api/v1/cfg-products/{productSn}` | GET | 查詢單一產品配置 | [cfg-products.md](cfg-products.md) |
| `/api/v1/cfg-product-activities/query` | POST | 條件查詢產品活動配置 | [cfg-product-activities.md](cfg-product-activities.md) |
| `/api/v1/cfg-contract-activities/batch-query` | POST | 批次查詢合約活動配置 | [cfg-contract-activities.md](cfg-contract-activities.md) |

**設計**: 採用 RESTful 慣例，列表與單筆查詢分離

### 4. Client & Contract Management

| 端點 | 方法 | 說明 | 文件 |
|------|------|------|------|
| `/api/v1/client-purchases` | GET | 查詢客戶購買記錄（必填 clientSn） | [client-purchases.md](client-purchases.md) |
| `/api/v1/client-temporal-contracts` | GET | 查詢客戶臨時合約列表（必填 clientSn） | [client-temporal-contracts.md](client-temporal-contracts.md) |
| `/api/v1/client-temporal-contracts/{sn}` | GET | 查詢單一臨時合約 | [client-temporal-contracts.md](client-temporal-contracts.md) |
| `/api/v1/contract-change-records` | GET | 查詢合約變更記錄（必填 accountSn，可選 changeTypeSn） | [contract-change-records.md](contract-change-records.md) |

### 5. Activity Records

| 端點 | 方法 | 說明 | 文件 |
|------|------|------|------|
| `/api/v1/activity-records` | GET | 範圍查詢活動記錄（必填 from, to） | [activity-records.md](activity-records.md) |
| `/api/v1/activity-records/calculate-results` | PUT | 批次更新計算結果 | [activity-records.md](activity-records.md) |

**特點**: 包含查詢與更新操作

## Mapper 對應表

### 完整對應關係

| Mapper | 方法數 | 對應 API 數 | Mapper 文件 | API 文件 |
|--------|--------|------------|------------|----------|
| SessionTypeMapper | 3 | 1 | [SessionTypeMapper.md](_mappers/SessionTypeMapper.md) | [session-type.md](session-type.md) |
| ActivityRuleDetailMapper | 2 | 1 | [ActivityRuleDetailMapper.md](_mappers/ActivityRuleDetailMapper.md) | [activity-rule-details.md](activity-rule-details.md) |
| ActivityRuleHeadMapper | 2 | 1 | [ActivityRuleHeadMapper.md](_mappers/ActivityRuleHeadMapper.md) | [activity-rule-heads.md](activity-rule-heads.md) |
| ActivityRuleRewardMapper | 2 | 1 | [ActivityRuleRewardMapper.md](_mappers/ActivityRuleRewardMapper.md) | [activity-rule-rewards.md](activity-rule-rewards.md) |
| CfgActivityMapper | 2 | 2 | [CfgActivityMapper.md](_mappers/CfgActivityMapper.md) | [cfg-activities.md](cfg-activities.md) |
| CfgProductMapper | 2 | 2 | [CfgProductMapper.md](_mappers/CfgProductMapper.md) | [cfg-products.md](cfg-products.md) |
| ClientPurchaseMapper | 1 | 1 | [ClientPurchaseMapper.md](_mappers/ClientPurchaseMapper.md) | [client-purchases.md](client-purchases.md) |
| ClientTemporalContractMapper | 2 | 2 | [ClientTemporalContractMapper.md](_mappers/ClientTemporalContractMapper.md) | [client-temporal-contracts.md](client-temporal-contracts.md) |
| ContractChangeRecordMapper | 1 | 1 | [ContractChangeRecordMapper.md](_mappers/ContractChangeRecordMapper.md) | [contract-change-records.md](contract-change-records.md) |
| CfgContractActivityMapper | 1 | 1 | [CfgContractActivityMapper.md](_mappers/CfgContractActivityMapper.md) | [cfg-contract-activities.md](cfg-contract-activities.md) |
| CfgProductActivityMapper | 1 | 1 | [CfgProductActivityMapper.md](_mappers/CfgProductActivityMapper.md) | [cfg-product-activities.md](cfg-product-activities.md) |
| ActivityRecordMapper | 2 | 2 | [ActivityRecordMapper.md](_mappers/ActivityRecordMapper.md) | [activity-records.md](activity-records.md) |

### 參數還原速查表

#### SessionTypeMapper

| 原始方法 | API 呼叫 |
|---------|---------|
| `selectAll()` | `GET /session-types` |
| `selectByWebSite(String webSite)` | `GET /session-types?webSite={webSite}` |
| `selectBySn(Integer sourceSn)` | `GET /session-types?sourceSn={sourceSn}` |

#### ActivityRuleDetailMapper

| 原始方法 | API 呼叫 |
|---------|---------|
| `selectAll()` | `GET /activity-rule-details` |
| `selectByLevelNo(String levelNo)` | `GET /activity-rule-details?levelNo={levelNo}` |

#### ActivityRuleHeadMapper

| 原始方法 | API 呼叫 |
|---------|---------|
| `selectAll()` | `GET /activity-rule-heads` |
| `selectByActivitySn(int activitySn)` | `GET /activity-rule-heads?activitySn={activitySn}` |

#### ActivityRuleRewardMapper

| 原始方法 | API 呼叫 |
|---------|---------|
| `selectAll()` | `GET /activity-rule-rewards` |
| `selectBylevelNo(String levelNo)` | `GET /activity-rule-rewards?levelNo={levelNo}` |

#### CfgActivityMapper

| 原始方法 | API 呼叫 |
|---------|---------|
| `selectAll()` | `GET /cfg-activities` |
| `selectByPrimaryKey(Integer sn)` | `GET /cfg-activities/{sn}` |

#### CfgProductMapper

| 原始方法 | API 呼叫 |
|---------|---------|
| `selectAllMuchNew()` | `GET /cfg-products` |
| `selectByProductSn(int productSn)` | `GET /cfg-products/{productSn}` |

#### ClientPurchaseMapper

| 原始方法 | API 呼叫 |
|---------|---------|
| `selectByClientSn(Integer clientSn)` | `GET /client-purchases?clientSn={clientSn}` |

#### ClientTemporalContractMapper

| 原始方法 | API 呼叫 |
|---------|---------|
| `selectByClientSn(Integer clientSn)` | `GET /client-temporal-contracts?clientSn={clientSn}` |
| `selectByPrimaryKey(Integer sn)` | `GET /client-temporal-contracts/{sn}` |

#### ContractChangeRecordMapper

| 原始方法 | API 呼叫 |
|---------|---------|
| `selectByAccountSn(Integer accountSn, Integer changeTypeSn)` | `GET /contract-change-records?accountSn={accountSn}&changeTypeSn={changeTypeSn}` |

#### CfgContractActivityMapper

| 原始方法 | API 呼叫 |
|---------|---------|
| `selectByContractSns(List contractSns)` | `POST /cfg-contract-activities/batch-query` |

#### CfgProductActivityMapper

| 原始方法 | API 呼叫 |
|---------|---------|
| `selectByExample(CfgProductActivityEntityExample example)` | `POST /cfg-product-activities/query` |

#### ActivityRecordMapper

| 原始方法 | API 呼叫 |
|---------|---------|
| `selectbySn(Integer from, Integer to)` | `GET /activity-records?from={from}&to={to}` |
| `updateCalculateResult(List<ActivityRecordEntity> records)` | `PUT /activity-records/calculate-results` |

## 合併策略說明

### 合併規則

#### 規則 1: 查詢參數合併

**適用場景**: 相同資源的查詢變體，只是過濾條件不同

**範例**:
- SessionTypeMapper: `selectAll()` + `selectByWebSite()` + `selectBySn()` → `GET /session-types?webSite=...&sourceSn=...`
- ActivityRuleDetailMapper: `selectAll()` + `selectByLevelNo()` → `GET /activity-rule-details?levelNo=...`

**優勢**:
- 減少 API 端點數量
- 提供更靈活的查詢組合
- 符合 RESTful 設計原則

#### 規則 2: RESTful 資源設計

**適用場景**: 列表查詢與單筆查詢

**範例**:
- CfgActivityMapper: `selectAll()` → `GET /cfg-activities`, `selectByPrimaryKey()` → `GET /cfg-activities/{sn}`
- CfgProductMapper: `selectAllMuchNew()` → `GET /cfg-products`, `selectByProductSn()` → `GET /cfg-products/{productSn}`

**設計理由**:
- 列表與單筆查詢語意不同
- 符合 HTTP 語意：集合端點 vs 資源端點
- 提升 API 可讀性

#### 規則 3: 批次操作使用 POST

**適用場景**: 參數為 List 或複雜物件

**範例**:
- CfgContractActivityMapper: `selectByContractSns(List)` → `POST /cfg-contract-activities/batch-query`
- CfgProductActivityMapper: `selectByExample(Example)` → `POST /cfg-product-activities/query`

**設計理由**:
- 避免 GET URL 長度限制
- POST Request Body 支援複雜資料結構

### 未合併案例

以下 Mapper 方法因語意差異未進行合併：

1. **CfgActivityMapper / CfgProductMapper**: 列表與單筆查詢分離，符合 RESTful 慣例
2. **ClientTemporalContractMapper**: 提供客戶視角與合約視角兩種查詢
3. **ActivityRecordMapper**: 查詢與更新操作語意不同

## 通用設計規範

### HTTP 方法使用

| 方法 | 用途 | 範例 |
|------|------|------|
| GET | 查詢資源 | `GET /session-types`, `GET /cfg-activities/{sn}` |
| POST | 批次查詢、複雜條件查詢 | `POST /cfg-product-activities/query` |
| PUT | 更新資源 | `PUT /activity-records/calculate-results` |

### 回應格式

**成功回應** (HTTP 200):
```json
{
  "success": true,
  "data": [...],
  "total": 10,
  "timestamp": "2026-02-11T10:00:00Z"
}
```

**錯誤回應** (HTTP 4xx/5xx):
```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "錯誤訊息",
    "details": "詳細說明"
  },
  "timestamp": "2026-02-11T10:00:00Z"
}
```

### 命名慣例

- **端點**: kebab-case（例如：`/session-types`, `/activity-rule-details`）
- **參數**: camelCase（例如：`webSite`, `activitySn`）
- **DTO**: PascalCase + Suffix（例如：`SessionTypeResponse`, `ActivityRuleDetailResponse`）

### 快取建議

| 資源類型 | 建議快取時間 | 說明 |
|---------|-------------|------|
| 配置類（cfg-*） | 10-15 分鐘 | 變動頻率低 |
| 規則類（activity-rule-*） | 5-10 分鐘 | 相對穩定 |
| 記錄類（*-records） | 3-5 分鐘或不快取 | 可能頻繁變動 |
| 稽核類（contract-change-records） | 10-15 分鐘 | 只增不改 |

### 分頁建議

對於可能返回大量資料的 API，建議實作分頁功能：

```
GET /api/v1/resource?page=1&pageSize=20
```

**建議實作分頁的端點**:
- `/cfg-activities`
- `/cfg-products`
- `/client-purchases`
- `/activity-records`

## 快速導航

### 依功能分類

**活動管理**:
- [session-type.md](session-type.md) - 會話類型
- [activity-rule-details.md](activity-rule-details.md) - 活動規則明細
- [activity-rule-heads.md](activity-rule-heads.md) - 活動規則主檔
- [activity-rule-rewards.md](activity-rule-rewards.md) - 活動規則獎勵
- [activity-records.md](activity-records.md) - 活動記錄

**配置管理**:
- [cfg-activities.md](cfg-activities.md) - 活動配置
- [cfg-products.md](cfg-products.md) - 產品配置
- [cfg-product-activities.md](cfg-product-activities.md) - 產品活動配置
- [cfg-contract-activities.md](cfg-contract-activities.md) - 合約活動配置

**客戶與合約**:
- [client-purchases.md](client-purchases.md) - 客戶購買記錄
- [client-temporal-contracts.md](client-temporal-contracts.md) - 客戶臨時合約
- [contract-change-records.md](contract-change-records.md) - 合約變更記錄

### Mapper 文件

所有 Mapper 對應文件位於 [_mappers/](_mappers/) 目錄下。

---

**文件版本**: 1.0  
**最後更新**: 2026-02-11  
**維護者**: dal_products API 遷移專案團隊
