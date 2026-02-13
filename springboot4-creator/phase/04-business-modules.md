# Phase 4: 業務模組程式碼生成

根據 Phase 1 收集的功能模組列表，為每個模組生成三層架構的基本骨架（不含 CRUD）。

## 執行條件

- Phase 1 的 `modules` 列表不為空

## 執行流程

### Step 4.1: 為每個模組生成 DTO

為每個模組創建 Request 和 Response DTO 的骨架。

**dto/request/{Entity}Req.java:**

```java
package {basePackage}.{appName}.dto.request;

import lombok.Data;

/**
 * {Entity} Request DTO
 *
 * TODO: 根據業務需求定義請求欄位
 *
 * 範例：
 * private String name;
 * private String email;
 */
@Data
public class {Entity}Req {

    // TODO: 在此新增請求欄位

}
```

**dto/response/{Entity}Rsp.java:**

```java
package {basePackage}.{appName}.dto.response;

import lombok.Data;

/**
 * {Entity} Response DTO
 *
 * TODO: 根據業務需求定義回應欄位
 *
 * 範例：
 * private Long id;
 * private String name;
 * private String email;
 * private LocalDateTime createdAt;
 */
@Data
public class {Entity}Rsp {

    // TODO: 在此新增回應欄位

}
```

---

### Step 4.2: 為每個模組生成 Domain Entity（如果使用資料庫）

僅在 `useDatabase=true` 時執行。

**domain/{Entity}.java:**

```java
package {basePackage}.{appName}.domain;

import jakarta.persistence.*;
import lombok.Getter;
import lombok.Setter;

/**
 * {Entity} Entity
 *
 * TODO: 根據資料庫表結構定義欄位和關係
 *
 * 範例：
 * @Column(name = "name", nullable = false, length = 100)
 * private String name;
 *
 * @Column(name = "email", unique = true)
 * private String email;
 */
@Entity
@Table(name = "{entity_table_name}")
@Getter
@Setter
public class {Entity} {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    // TODO: 在此新增欄位
}
```

**執行動作：**

- {entity_table_name} 使用蛇形命名法（例如：User -> user, ProductOrder -> product_order）

---

### Step 4.3: 為每個模組生成 Repository（如果使用資料庫）

僅在 `useDatabase=true` 時執行。

**repository/{Entity}Repository.java:**

```java
package {basePackage}.{appName}.repository;

import {basePackage}.{appName}.domain.{Entity};
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

/**
 * {Entity} Repository
 *
 * TODO: 根據查詢需求新增自定義查詢方法
 *
 * 範例：
 * Optional<{Entity}> findByEmail(String email);
 * List<{Entity}> findByNameContaining(String keyword);
 */
@Repository
public interface {Entity}Repository extends JpaRepository<{Entity}, Long> {

    // TODO: 在此新增自定義查詢方法

}
```

---

### Step 4.4: 為每個模組生成 Service 介面和實作

生成 Service 層的介面和實作類別骨架。

**service/{Entity}Service.java:**

```java
package {basePackage}.{appName}.service;

import {basePackage}.{appName}.dto.request.{Entity}Req;
import {basePackage}.{appName}.dto.response.{Entity}Rsp;

/**
 * {Entity} Service Interface
 *
 * TODO: 根據業務需求定義服務方法
 *
 * 範例：
 * {Entity}Rsp create({Entity}Req req);
 * {Entity}Rsp getById(Long id);
 * List<{Entity}Rsp> list();
 */
public interface {Entity}Service {

    // TODO: 在此新增業務方法

}
```

**service/impl/{Entity}ServiceImpl.java:**

```java
package {basePackage}.{appName}.service.impl;

import {basePackage}.{appName}.service.{Entity}Service;
// 如果有 repository，加入以下 import
// import {basePackage}.{appName}.repository.{Entity}Repository;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

/**
 * {Entity} Service Implementation
 *
 * TODO: 實作 {Entity}Service 介面中定義的方法
 */
@Slf4j
@Service
@RequiredArgsConstructor
public class {Entity}ServiceImpl implements {Entity}Service {

    // 如果有 repository，加入以下依賴
    // private final {Entity}Repository {entity}Repository;

    // TODO: 在此實作業務邏輯

}
```

---

### Step 4.5: 為每個模組生成 Controller

生成 REST API Controller 骨架。

**controller/{Entity}Controller.java:**

```java
package {basePackage}.{appName}.controller;

import {basePackage}.{appName}.core.annotation.ApiLog;
import {basePackage}.{appName}.core.web.ApiResponse;
import {basePackage}.{appName}.core.web.BaseResponse;
import {basePackage}.{appName}.dto.request.{Entity}Req;
import {basePackage}.{appName}.dto.response.{Entity}Rsp;
import {basePackage}.{appName}.service.{Entity}Service;
import io.swagger.v3.oas.annotations.tags.Tag;
import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

/**
 * {Entity} Controller
 *
 * TODO: 根據業務需求新增 API endpoints
 *
 * 範例：
 * @ApiLog
 * @PostMapping
 * public ResponseEntity<BaseResponse<{Entity}Rsp>> create(@RequestBody {Entity}Req req) {
 *     {Entity}Rsp result = {entity}Service.create(req);
 *     return ApiResponse.ok(result);
 * }
 */
@Tag(name = "{Entity}", description = "{Entity} API")
@RestController
@RequiredArgsConstructor
@RequestMapping("/api/{entity-lowercase}")
public class {Entity}Controller {

    private final {Entity}Service {entity}Service;

    // TODO: 在此新增 API endpoints

}
```

**執行動作：**

- {entity-lowercase} 使用小寫加短橫線（例如：ProductOrder -> product-order）

---

## 執行邏輯

1. 遍歷 Phase 1 的 `modules` 列表
2. 對每個模組執行 Step 4.1 - 4.5
3. 根據 `useDatabase` 決定是否生成 Domain 和 Repository
4. 所有生成的類別都是骨架，包含 TODO 註解引導使用者填入具體邏輯

## 輸出

- 為每個模組生成完整的三層架構骨架
- 包含 Controller、Service、Repository（條件性）、Domain（條件性）、DTO
