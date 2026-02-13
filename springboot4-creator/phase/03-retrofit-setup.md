# Phase 3: Retrofit 外部 API 配置

當 Phase 1 確認需要使用外部 API (`useExternalApi=true`) 時，執行此 Phase 配置具體的外部 API 介面。

**前置條件**：Phase 2 已生成 RetrofitConfig 基礎配置（包含 traceId 機制）

## 執行條件

- Phase 1 的 `useExternalApi` 為 `true`
- 有 `apiName` 和 `apiBaseUrl` 資訊
- Phase 2 已完成基礎 Retrofit 配置生成

## 執行流程

### Step 3.1: 取消 RetrofitConfig 中的示例註解並配置實際 API

修改 Phase 2 生成的 RetrofitConfig，取消註解並配置實際的外部 API。

**修改 config/RetrofitConfig.java：**

1. 找到註解的 EXAMPLE 區塊
2. 取消註解
3. 修改為實際的 API 名稱和 Base URL

```java
// 原本的註解區塊：
/* EXAMPLE: 外部 API 介面 Bean 範例（實際使用時取消註解並修改）
@Bean
public ExampleApi exampleApi(OkHttpClient client, GsonConverterFactory gson) {
    return new Retrofit.Builder()
            .baseUrl("https://api.example.com")
            .client(client)
            .addConverterFactory(gson)
            .build()
            .create(ExampleApi.class);
}
*/

// 修改為：
@Bean
public {ApiName}Api {apiName}Api(OkHttpClient client, GsonConverterFactory gson) {
    return new Retrofit.Builder()
            .baseUrl("{apiBaseUrl}")  // 使用實際的 API Base URL
            .client(client)
            .addConverterFactory(gson)
            .build()
            .create({ApiName}Api.class);
}
```

---

### Step 3.2: 生成外部 API 配置類別（可選）

如果需要從 application.yml 讀取 API Base URL，可創建配置類別。

**config/ExternalApiDomainConfig.java:**

```java
package {basePackage}.{appName}.config;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;

@Data
@Configuration
@ConfigurationProperties(prefix = "external.api")
public class ExternalApiDomainConfig {

    private String {apiName};
}
```

然後在 RetrofitConfig 中使用配置：

```java
// 在 RetrofitConfig 類中添加：
private final ExternalApiDomainConfig externalApiDomainConfig;

// 修改 API Bean：
@Bean
public {ApiName}Api {apiName}Api(OkHttpClient client, GsonConverterFactory gson) {
    return new Retrofit.Builder()
            .baseUrl(externalApiDomainConfig.get{ApiName}())  // 從配置讀取
            .client(client)
            .addConverterFactory(gson)
            .build()
            .create({ApiName}Api.class);
}
```

**application-dev.yml 配置：**

```yaml
external:
  api:
    { apiName }: { apiBaseUrl }
```

**application-prd.yml 配置：**

```yaml
external:
  api:
    { apiName }: ${API_BASE_URL} # 從環境變數讀取
```

**執行動作：**

1. 替換 {ApiName} 為使用者提供的 API 名稱（首字母大寫）
2. 替換 {apiName} 為小寫開頭的變數名稱
3. 替換 {apiBaseUrl} 為實際的 API Base URL

---

### Step 3.3: 生成外部 API 介面（骨架）

創建 Retrofit API 介面的基本骨架，讓使用者自行填入具體的 API endpoints。

**external/api/{ApiName}Api.java:**

```java
package {basePackage}.{appName}.external.api;

import okhttp3.ResponseBody;
import retrofit2.Call;
import retrofit2.http.*;

/**
 * {ApiName} External API Interface
 *
 * TODO: 請根據實際 API 文件填入以下內容：
 * 1. API endpoints (@GET, @POST, @PUT, @DELETE)
 * 2. 請求參數 (@Query, @Path, @Body, @Header)
 * 3. 返回類型 (建議使用對應的 Response DTO)
 *
 * 範例：
 * @GET("/api/v3/users")
 * Call<GetUsersRsp> getUsers(@Query("page") int page, @Query("size") int size);
 */
public interface {ApiName}Api {

    // TODO: 在此新增 API endpoints

}
```

---

### Step 3.4: 生成 BaseApiClient

創建 BaseApiClient 基礎類別，封裝 Retrofit Call 的執行邏輯和錯誤處理。

**external/client/BaseApiClient.java:**

```java
package {basePackage}.{appName}.external.client;

import okhttp3.ResponseBody;
import retrofit2.Response;

import java.util.concurrent.Callable;

/**
 * Base API Client for external API calls
 * 提供統一的 API 呼叫錯誤處理
 */
public abstract class BaseApiClient {

    protected <T> T call(Callable<Response<T>> action) {
        try {
            Response<T> rsp = action.call();
            if (rsp.isSuccessful()) {
                return rsp.body();
            }
            String error;
            try (ResponseBody errorBody = rsp.errorBody()) {
                error = errorBody != null ? errorBody.string() : "";
            }
            throw new RuntimeException("External API error: " + rsp.code() + " - " + error);

        } catch (Exception e) {
            throw new RuntimeException("External API call failed", e);
        }
    }
}
```

---

### Step 3.5: 生成 API Client 介面和實作（骨架）

創建具體的 API Client 介面和實作類別骨架。

**external/client/{ApiName}ApiClient.java:**

```java
package {basePackage}.{appName}.external.client;

/**
 * {ApiName} API Client Interface
 *
 * TODO: 請定義需要呼叫的外部 API 方法
 *
 * 範例：
 * GetUsersRsp getUsers(int page, int size);
 */
public interface {ApiName}ApiClient {

    // TODO: 在此新增 API client 方法

}
```

**external/client/impl/{ApiName}ApiClientImpl.java:**

```java
package {basePackage}.{appName}.external.client.impl;

import {basePackage}.{appName}.external.api.{ApiName}Api;
import {basePackage}.{appName}.external.client.BaseApiClient;
import {basePackage}.{appName}.external.client.{ApiName}ApiClient;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

/**
 * {ApiName} API Client Implementation
 *
 * TODO: 實作 {ApiName}ApiClient 介面中定義的方法
 *
 * 範例：
 * @Override
 * public GetUsersRsp getUsers(int page, int size) {
 *     return call(() -> {apiName}Api.getUsers(page, size).execute());
 * }
 */
@Slf4j
@Component
@RequiredArgsConstructor
public class {ApiName}ApiClientImpl extends BaseApiClient implements {ApiName}ApiClient {

    private final {ApiName}Api {apiName}Api;

    // TODO: 在此實作 API 呼叫方法

}
```

---

### Step 3.6: 創建 Request/Response DTO 資料夾

創建 external 下的 request 和 response 資料夾，並加入 README 說明。

**external/request/README.md:**

````markdown
# External API Request DTOs

此資料夾存放呼叫外部 API 時使用的請求 DTO。

## 命名規範

- 格式：`{Action}{Entity}Req.java`
- 範例：`GetUserReq.java`, `CreateOrderReq.java`

## 範例

```java
package {basePackage}.{appName}.external.request;

import lombok.Data;

@Data
public class GetTokenReq {
    private String clientId;
    private String clientSecret;
    private String grantType;
    private String refreshToken;
}
```
````

````

**external/response/README.md:**
```markdown
# External API Response DTOs

此資料夾存放外部 API 回應的 DTO。

## 命名規範

- 格式：`{Action}{Entity}Rsp.java` 或 `{Entity}Rsp.java`
- 範例：`GetUserRsp.java`, `TokenRsp.java`

## 範例

```java
package {basePackage}.{appName}.external.response;

import lombok.Data;

@Data
public class GetTokenRsp {
    private String accessToken;
    private String tokenType;
    private Long expiresIn;
    private String refreshToken;
}
````

```

---

## 輸出

- Retrofit 配置完成
- 外部 API 的基本骨架已建立
- 使用者可以根據實際 API 文件填入具體的 endpoints 和方法
```
