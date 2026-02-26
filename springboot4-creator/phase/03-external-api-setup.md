# Phase 3: 外部 API 整合與 TraceId 配置

自 Spring Boot 4 起，建議直接使用 Spring 官方內建的聲明式 HTTP API（`@HttpExchange`/`RestClient`），完全不需 Retrofit。

此外，為了支援全鏈路追蹤，請務必於 RestClient 配置 TraceId 攔截器，並於 Filter 注入 traceId 至 MDC 與 HTTP Header，參考 ASS 專案最佳實踐。

## 流程說明

1. 實作 TraceFilter，於每次請求自動產生/傳遞 traceId，並存入 MDC、回應 Header。
2. RestClient.Builder 註冊攔截器，自 MDC 取出 traceId，自動帶入外部 API 請求 Header。
3. 定義聲明式 HTTP API 介面（@HttpExchange）。
4. 註冊 RestClient Bean 並注入 Service 使用。

---

### 1. 實作 TraceFilter

於 `core/filter/TraceFilter.java`：

```java
package {basePackage}.{appName}.core.filter;

import jakarta.servlet.Filter;
import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.ServletRequest;
import jakarta.servlet.ServletResponse;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.apache.commons.lang3.StringUtils;
import org.slf4j.MDC;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

import java.io.IOException;

@Component
@Order(1)
public class TraceFilter implements Filter {
    private static final String TRACE_ID_HEADER = "X-Trace-Id";
    private static final String TRACE_ID_MDC_KEY = "traceId";

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;
        String traceId = httpRequest.getHeader(TRACE_ID_HEADER);
        if (StringUtils.isBlank(traceId)) {
            long now = System.currentTimeMillis();
            traceId = Long.toString(now);
        }
        MDC.put(TRACE_ID_MDC_KEY, traceId);
        httpResponse.setHeader(TRACE_ID_HEADER, traceId);
        try {
            chain.doFilter(request, response);
        } finally {
            MDC.remove(TRACE_ID_MDC_KEY);
        }
    }
}
```

---

### 2. RestClient Builder 註冊 TraceId 攔截器

於 `config/BaseRestClientConfig.java`：

```java
package {basePackage}.{appName}.config;

import org.apache.commons.lang3.StringUtils;
import org.slf4j.MDC;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestClient;

@Configuration
public class BaseRestClientConfig {
    @Bean
    public RestClient.Builder restClientBuilder() {
        return RestClient.builder()
            .requestInterceptor((request, body, execution) -> {
                String traceId = MDC.get("traceId");
                if (StringUtils.isNotBlank(traceId)) {
                    request.getHeaders().add("X-Trace-Id", traceId);
                }
                return execution.execute(request, body);
            });
    }
}
```

---

### 3. 定義聲明式 HTTP API 介面

於 `external/api/{ApiName}Api.java`：

```java
package {basePackage}.{appName}.external.api;

import org.springframework.web.service.annotation.GetExchange;
import org.springframework.web.service.annotation.HttpExchange;
import org.springframework.web.service.annotation.PostExchange;
import org.springframework.web.service.annotation.RequestBody;
import org.springframework.web.service.annotation.RequestParam;
import org.springframework.web.service.annotation.PathVariable;
import org.springframework.http.MediaType;

@HttpExchange("{apiBaseUrl}")
public interface {ApiName}Api {
    @GetExchange("/resource/{id}")
    {ResponseType} getResource(@PathVariable("id") String id);

    @PostExchange(value = "/resource", contentType = MediaType.APPLICATION_JSON_VALUE)
    {ResponseType} createResource(@RequestBody {RequestType} req);
    // ...可根據實際 API 定義更多方法
}
```

---

### 4. 註冊 API Bean

於 `config/ExternalApiConfig.java`：

```java
package {basePackage}.{appName}.config;

import {basePackage}.{appName}.external.api.{ApiName}Api;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestClient;

@Configuration
public class ExternalApiConfig {
    @Bean
    public {ApiName}Api {apiName}Api(RestClient.Builder restClientBuilder) {
        return restClientBuilder.baseUrl("{apiBaseUrl}").build().create({ApiName}Api.class);
    }
}
```

---

### 5. Service 層直接注入 {ApiName}Api 使用

```java
@Service
@RequiredArgsConstructor
public class ExampleService {
    private final {ApiName}Api {apiName}Api;

    public {ResponseType} callApi(String id) {
        return {apiName}Api.getResource(id);
    }
}
```

---

## 備註

- Spring Boot 4 聲明式 HTTP API 具備型別安全、AOP 攔截、測試友善等優點。
- TraceId 全鏈路追蹤建議於 Filter + RestClient 攔截器雙層實作。
- 不需 Retrofit 任何依賴與設定，請勿再出現 Retrofit 相關內容。
