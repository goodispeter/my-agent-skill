# Spring Boot Code Review Checklist

## 必檢項目（Critical - 可自動化檢測）

### 1.Coding Style & 專案架構

- [ ] Coding style 是否符合專案架構規範
- [ ] 命名規範是否一致（camelCase / PascalCase）
- [ ] package 結構是否合理

### 2.NPE 風險（NullPointerException）

- [ ] 物件使用前是否有 null check
- [ ] List 操作前是否檢查 null 或 isEmpty()
- [ ] Map 操作前是否檢查 null 或 isEmpty()
- [ ] DTO 欄位是否可能為 null
- [ ] Optional 是否亂用 `.get()` 而不檢查 `.isPresent()`

### 3.比較運算錯誤

- [ ] equals() 用 `==` 比較
- [ ] String 用 `==` 比較
- [ ] BigDecimal 用 `==` 比較
- [ ] 應使用 `.equals()` 或 `Objects.equals()` 的地方

### 4.基本語法檢查

- [ ] switch 是否有 default 分支
- [ ] magic number（魔法數字）- 應提取為常數
- [ ] 空的 catch block
- [ ] 資源未關閉（Connection / Stream / File）

### 5.業務邏輯驗證

- [ ] 業務邏輯是否符合需求？
- [ ] edge case 有處理嗎？（邊界條件）
- [ ] 空資料會怎樣？（empty list / null / ""）
- [ ] DB 查不到會怎樣？（404 / 預設值 / 拋錯）
- [ ] 併發情況下的資料一致性

---

## 三層式架構原則（Architecture）

### Controller 層

- [ ] Controller 只有 request/response 處理
- [ ] 不寫業務邏輯（business logic）
- [ ] 參數驗證（@Valid / @Validated）
- [ ] 統一回傳格式（ResponseEntity / Result）

### Service 層

- [ ] Service 才有 business logic
- [ ] 不直接操作 SQL
- [ ] Transaction 邊界正確（@Transactional）
- [ ] 異常處理明確

### DAO / Repository 層

- [ ] DAO 只做 DB 操作
- [ ] 不包含業務邏輯
- [ ] SQL 寫在 Mapper XML 或 Repository

### Cross-Layer 檢查

- [ ] 沒有 cross-layer 呼叫（controller → dao ）
- [ ] 沒有把 SQL 寫在 Service

---

## 設計原則（SOLID & Clean Code）

### 單一職責原則 (SRP)

- [ ] 一個方法只做一件事
- [ ] 方法太長 > 50 行 && 可以復用？（如有復用性建議拆分，無復用性**不建議拆分**）
- [ ] class 太肥 > 500 行？（建議重構）
- [ ] util 濫用？（是否該用 Service）
- [ ] static 方法太多？（考慮依賴注入）

### 命名規範

- [ ] 變數命名清楚（不要 a, b, c, temp, data）
- [ ] method 名稱是動詞開頭（get / set / create / update / delete / calculate）
- [ ] boolean 命名 is / has / can / should
- [ ] 常數全大寫（CONSTANT_VALUE）

### 可讀性優化

- [ ] 避免巢狀 if > 3 層
- [ ] early return 代替大 if-else
- [ ] 複雜條件提取為 boolean 變數
- [ ] switch 優於多個 if-else

---

## 效能問題（Performance）

### Database 查詢

- [ ] for 迴圈內查 DB （N+1 query）
- [ ] 重複查同一筆資料
- [ ] Stream 裡面呼叫外部 service 或 DB
- [ ] 無 limit 的查詢（應加分頁）
- [ ] 沒 index 的 where 條件
- [ ] SELECT \* 濫用（應指定欄位）

### API 呼叫

- [ ] 同步呼叫外部 API 造成 blocking（考慮非同步）
- [ ] 沒有 timeout 設定
- [ ] 沒有 retry 機制
- [ ] 沒有 fallback 處理

### 併發控制

- [ ] 不必要的 synchronized（效能瓶頸）
- [ ] Lock 範圍過大
- [ ] 沒有考慮 deadlock 風險

---

## 安全性檢查（Security）

- [ ] SQL Injection 風險（使用 PreparedStatement / MyBatis）
- [ ] XSS 攻擊（輸出需轉義）
- [ ] 敏感資料是否加密（密碼 / token）
- [ ] Log 中不要印出密碼 / token
- [ ] API 權限控制（@PreAuthorize）

---

## 測試覆蓋

- [ ] 核心邏輯是否有單元測試
- [ ] 邊界條件是否有測試
- [ ] 異常情況是否有測試

---

## 文件與註解

- [ ] public 方法是否有 JavaDoc
- [ ] 複雜邏輯是否有註解說明
- [ ] TODO / FIXME 是否該處理
- [ ] 註解是否與 code 同步更新
