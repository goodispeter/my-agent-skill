# Spring Boot Code Review Report Template

## 目錄

- [Review 基本資訊](#-review-基本資訊)
- [問題摘要](#-問題摘要)
- [嚴重問題](#-嚴重問題-critical)
- [中等問題](#-中等問題-warning)
- [輕微問題](#-輕微問題-suggestion)
- [統計資訊](#-統計資訊)
- [優點與亮點](#-優點與亮點)
- [改善建議優先順序](#-改善建議優先順序)
- [參考資源](#-參考資源)

---

## Review 基本資訊

| 項目           | 內容                    |
| -------------- | ----------------------- |
| **審查時間**   | 2026-02-04 10:32        |
| **分支**       | feature/order-refactor  |
| **提交編號**   | a8f32cd                 |
| **審查者**     | AI Reviewer             |
| **變更檔案**   | 12 files                |
| **程式碼行數** | +342 / -128             |
| **審查範圍**   | Backend / Service Layer |

---

## 問題摘要

| 嚴重程度   | 數量   | 說明           |
| ---------- | ------ | -------------- |
| Critical   | 2      | 必須立即修正   |
| Warning    | 5      | 建議本週內修正 |
| Suggestion | 8      | 可排程改善     |
| **總計**   | **15** | -              |

---

## 嚴重問題 (Critical)

> **這些問題可能導致系統崩潰或資料錯誤，必須立即處理**

### Issue #1: NullPointer 風險

**位置**: [`UserService.java:84`](UserService.java#L84)  
**類別**: Runtime Exception  
**負責人**: @backend-team

#### 問題程式碼

```java
public String getUserName(Long userId) {
    User user = userRepository.findById(userId);
    return user.getName().toUpperCase(); // NPE 風險！
}
```

#### 問題說明

- `findById()` 可能回傳 `null`，未進行空值檢查
- `getName()` 可能為 `null`，直接調用會出錯
- 直接呼叫 `.toUpperCase()` 會拋出 `NullPointerException`
- **影響範圍**: 所有呼叫此方法的地方
- **風險等級**: High

#### 建議修正

```java
public String getUserName(Long userId) {
    return userRepository.findById(userId)
        .map(User::getName)
        .map(String::toUpperCase)
        .orElse("Unknown");
}
```

#### 行動建議

- [ ] 使用 Optional 包裝返回值
- [ ] 添加單元測試驗證空值處理
- [ ] 檢查其他類似的風險點

**參考資源**: [examples.md - NPE 風險處理](./examples.md#1-npe-風險處理)

---

### Issue #2: N+1 Query 效能問題

**位置**: [`OrderService.java:156`](OrderService.java#L156)
**類別**: Performance
**負責人**: @backend-team

#### 問題程式碼

```java
public List<OrderDTO> getOrders() {
    List<Order> orders = orderRepository.findAll();
    return orders.stream()
        .map(order -> {
            User user = userRepository.findById(order.getUserId()).get();
            // ...
        })
        .collect(Collectors.toList());
}
```

#### 問題說明

- 在 Stream 內部逐筆查詢資料庫，造成大量 SQL 查詢
- 若有 100 筆訂單，會執行 **1 + 100 次 SQL** (典型的 N+1 Query)
- 嚴重影響系統效能，特別是在資料量大時
- **預估影響**: 查詢時間可能增加 10-100 倍
- **風險等級**: High

#### 建議修正

```java
public List<OrderDTO> getOrders() {
    List<Order> orders = orderRepository.findAll();

    Set<Long> userIds = orders.stream()
        .map(Order::getUserId)
        .collect(Collectors.toSet());

    Map<Long, User> userMap = userRepository.findByIdIn(userIds).stream()
        .collect(Collectors.toMap(User::getId, user -> user));

    return orders.stream()
        .map(order -> convertToDTO(order, userMap.get(order.getUserId())))
        .collect(Collectors.toList());
}
```

#### 行動建議

- [ ] 實作批次查詢方法 `findByIdIn()`
- [ ] 進行效能測試對比
- [ ] 檢查專案中其他 N+1 Query 問題

** 參考資源**: [examples.md - N+1 Query 問題](./examples.md#8-n1-query-問題)

---

## 中等問題 (Warning)

> **這些問題影響程式碼品質和可維護性，建議盡快處理**

### Issue #3: 違反三層架構原則

**位置**: [`OrderController.java:45`](OrderController.java#L45)
**類別**: Architecture
**負責人**: @backend-team

#### 問題程式碼

```java
@PostMapping("/order")
public Result createOrder(@RequestBody OrderRequest request) {
    Order order = new Order();
    order.setUserId(request.getUserId());
    order.setTotalPrice(request.getItems().stream()
        .mapToInt(Item::getPrice).sum()); // Controller 有業務邏輯
    orderRepository.save(order); // Controller 直接操作 DB
    return Result.success(order);
}
```

#### 問題說明

- **違反職責分離**: Controller 應只處理 HTTP request/response
- **業務邏輯外洩**: 計算總價應在 Service 層處理
- **架構混亂**: 資料存取應透過 Service 層，Controller 不應直接注入 Repository
- **影響**: 降低程式碼可測試性和可維護性
- **風險等級**: Medium

#### 建議修正

```java
@PostMapping("/order")
public Result<OrderDTO> createOrder(@Valid @RequestBody OrderRequest request) {
    OrderDTO order = orderService.createOrder(request);
    return Result.success(order);
}
```

#### 行動建議

- [ ] 將業務邏輯移至 Service 層
- [ ] 移除 Controller 中的 Repository 依賴
- [ ] 添加參數驗證 `@Valid`

**參考資源**: [examples.md - 三層架構違反](./examples.md#5-三層架構違反)

---

## 輕微問題 (Suggestion)

> **這些是程式碼品質改善建議，可排程優化**

### Issue #8: 變數命名不清楚

**位置**: [`PaymentService.java:92`](PaymentService.java#L92)
**類別**: Code Quality
**負責人**: @backend-team

#### 問題程式碼

```java
public void process(Order o) {
    int a = o.getTotalPrice();
    boolean b = check(a);
    if (b) {
        save(o);
    }
}
```

#### 問題說明

- 使用單字母變數名 `o`, `a`, `b` 不具語意
- 方法名 `process`, `check`, `save` 過於籠統
- 降低程式碼可讀性和可維護性
- **風險等級**: Low

#### 建議修正

```java
public void processPayment(Order order) {
    int totalAmount = order.getTotalPrice();
    boolean isPaid = checkPaymentStatus(totalAmount);
    if (isPaid) {
        savePaymentRecord(order);
    }
}
```

#### 行動建議

- [ ] 重新命名所有單字母變數
- [ ] 使用有意義的方法名稱
- [ ] 團隊統一命名規範

**參考資源**: [examples.md - 命名不清楚](./examples.md#10-命名不清楚)

---

## 統計資訊

### 問題類型分佈

| 問題類型           | 數量 | 嚴重程度分佈 (嚴重/中等/輕微) |
| ------------------ | ---- | ----------------------------- |
| NullPointer 風險   | 3    | 2 / 1 / 0                     |
| N+1 Query 效能問題 | 2    | 1 / 1 / 0                     |
| 架構違反           | 4    | 0 / 2 / 2                     |
| 命名問題           | 8    | 0 / 0 / 8                     |
| Magic Number       | 5    | 0 / 1 / 4                     |

### 檔案問題分佈

| 檔案                 | 嚴重 | 中等 | 輕微 | 總計 |
| -------------------- | ---- | ---- | ---- | ---- |
| OrderService.java    | 2    | 3    | 3    | 8    |
| UserService.java     | 1    | 2    | 2    | 5    |
| PaymentService.java  | 0    | 1    | 3    | 4    |
| OrderController.java | 0    | 1    | 2    | 3    |

---

## 優點與亮點

> **表現優秀的部分，值得保持和推廣**

### 程式碼亮點

1. **完整的參數驗證** - [`UserController.java`](UserController.java)
   - 使用 `@Valid` 進行參數驗證
   - 統一的錯誤處理機制
   - 清晰的錯誤訊息返回

2. **良好的測試覆蓋** - [`ProductService.java`](ProductService.java)
   - 單元測試覆蓋率達 92%
   - 包含邊界條件測試
   - 使用 Mock 隔離依賴

3. **效能優化** - [`OrderRepository.java`](OrderRepository.java)
   - 有效使用資料庫索引
   - 合理的查詢批次處理
   - 避免了常見的 N+1 問題

### 最佳實踐

- 遵循 RESTful API 設計原則
- 良好的異常處理機制
- 適當的日誌記錄
- 使用 DTO 進行資料轉換

---

## 改善建議優先順序

### P0

> **影響**: 系統穩定性 / 資料安全性

| #   | 問題                      | 檔案              | 預估工時 |
| --- | ------------------------- | ----------------- | -------- |
| 1   | 修正所有 NullPointer 風險 | UserService.java  | 2h       |
| 2   | 解決 N+1 Query 效能問題   | OrderService.java | 3h       |

**總計**: 2 項 | 預估 5 小時

### P1

> **影響**: 程式碼品質 / 可維護性

| #   | 問題                       | 檔案                 | 預估工時 |
| --- | -------------------------- | -------------------- | -------- |
| 3   | 重構違反三層架構的程式碼   | OrderController.java | 4h       |
| 4   | 修正 equals 使用 == 的錯誤 | PaymentService.java  | 1h       |
| 5   | 補充缺失的異常處理         | UserService.java     | 2h       |

**總計**: 3 項 | 預估 7 小時

### P2

> **影響**: 程式碼可讀性

| #   | 問題                     | 預估工時 |
| --- | ------------------------ | -------- |
| 6   | 改善變數命名             | 3h       |
| 7   | 提取 Magic Number 為常數 | 2h       |
| 8   | 簡化巢狀 if 結構         | 2h       |

**總計**: 3 項 | 預估 7 小時

---

## 參考資源

### 內部文件

- [Code Review Checklist](./checklist.md) - 完整的檢查清單
- [Good vs Bad Examples](./examples.md) - 好壞範例對照
- [編碼規範](./coding-standards.md) - 團隊編碼標準
- [架構指南](./architecture-guide.md) - 系統架構最佳實踐

### 外部資源

- [Spring Boot Best Practices](https://spring.io/guides)
- [Effective Java (3rd Edition)](https://www.oracle.com/java/technologies/effectivejava.html)
- [Clean Code Principles](https://www.oreilly.com/library/view/clean-code-a/9780136083238/)

### 工具推薦

- **靜態分析**: SonarQube, SpotBugs, Checkstyle
- **效能分析**: JProfiler, VisualVM
- **測試工具**: JUnit 5, Mockito, Spring Test

---

## 審查團隊

| 角色          | 成員          | 職責           |
| ------------- | ------------- | -------------- |
| Lead Reviewer | AI Reviewer   | 整體審查與決策 |
| Code Reviewer | @backend-team | 程式碼品質審查 |
| Arch Reviewer | @architect    | 架構設計審查   |

---

## 後續追蹤

### 驗收標準

- [ ] 所有 P0 問題已修正並通過測試
- [ ] 所有 P1 問題已排程或完成
- [ ] 程式碼覆蓋率維持在 75% 以上
- [ ] 無新增的嚴重問題
- [ ] 通過 CI/CD 流程

### 下次審查

- **時間**: 2026-02-11 14:00
- **範圍**: P0/P1 問題修正驗證
- **參與者**: 開發團隊 + 審查團隊

---

**報告產生時間**: 2026-02-04 10:32:15
**報告版本**: v1.0
**審查工具**: AI Code Reviewer v1.0
