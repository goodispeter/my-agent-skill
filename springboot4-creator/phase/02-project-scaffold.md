# Phase 2: 專案骨架生成

生成 Spring Boot 4 專案的基礎架構，包括資料夾結構、配置檔案和核心框架程式碼。

**重要提醒**：

- README.md 在 Phase 5 生成，此階段不生成
- HelloController 在 Phase 5 生成並測試，此階段不生成
- 僅生成基礎框架和核心程式碼

## 執行流程

### Step 2.1: 創建專案資料夾結構

根據 Phase 1 收集的資訊，創建完整的 Maven 專案結構。

**資料夾結構：**

```
{projectName}/
├── .gitignore
├── pom.xml
├── mvnw
├── mvnw.cmd
├── .mvn/
│   └── wrapper/
│       ├── maven-wrapper.jar
│       └── maven-wrapper.properties
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── {basePackage}/{appName}/
│   │   │       ├── {AppName}Application.java
│   │   │       ├── config/
│   │   │       ├── controller/
│   │   │       ├── service/
│   │   │       ├── repository/         # 僅在 useDatabase=true 時創建
│   │   │       ├── domain/             # 僅在 useDatabase=true 時創建
│   │   │       ├── dto/
│   │   │       │   ├── request/
│   │   │       │   └── response/
│   │   │       ├── external/           # 僅在 useExternalApi=true 時創建
│   │   │       │   ├── api/
│   │   │       │   ├── client/
│   │   │       │   ├── request/
│   │   │       │   └── response/
│   │   │       ├── core/
│   │   │       │   ├── annotation/
│   │   │       │   ├── aop/
│   │   │       │   └── web/
│   │   │       ├── enums/
│   │   │       └── util/
│   │   └── resources/
│   │       ├── application.yml
│   │       ├── application-dev.yml
│   │       ├── application-prd.yml
│   │       ├── logback-spring.xml
│   │       ├── static/
│   │       └── templates/
│   └── test/
│       └── java/
│           └── {basePackage}/{appName}/
│               ├── {AppName}ApplicationTests.java
│               └── controller/
└── logs/                              # 日誌目錄（在 .gitignore 中）
```

**執行動作：**

1. 創建所有必要的資料夾
2. 根據條件決定是否創建 repository、domain、external 資料夾

---

### Step 2.2: 生成 pom.xml

生成包含所有必要依賴的 Maven 配置檔案。
非常重要: **不更動依賴版本，所有版本已指定好如 pom.xml**

**pom.xml 內容：**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>4.0.1</version>
        <relativePath/>
    </parent>

    <groupId>{basePackage}</groupId>
    <artifactId>{projectName}</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>{projectName}</name>
    <description>{description}</description>

    <properties>
        <java.version>25</java.version>
        <maven.compiler.source>25</maven.compiler.source>
        <maven.compiler.target>25</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <!-- Spring Boot Web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Spring Boot AOP -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aspectj</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
        </dependency>

        <!-- Database (僅在 useDatabase=true 時加入) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <!-- PostgreSQL 或 MySQL Driver 根據 dbType 決定 -->
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
        </dependency>
        <dependency>
            <groupId>io.hypersistence</groupId>
            <artifactId>hypersistence-utils-hibernate-71</artifactId>
            <version>3.14.1</version>
        </dependency>

        <!-- Validation -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>

        <!-- Swagger  -->
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
            <version>3.0.0</version>
        </dependency>

        <!-- Commons Collections -->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-collections4</artifactId>
            <version>4.4</version>
        </dependency>

        <!-- Lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <scope>provided</scope>
        </dependency>

        <!-- Testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webmvc-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <annotationProcessorPaths>
                        <path>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </path>
                    </annotationProcessorPaths>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

**執行動作：**

1. 替換 {basePackage}, {projectName}, {description} 為實際值
2. 根據 useDatabase 和 useExternalApi 決定是否包含相關依賴
3. 根據 dbType 選擇對應的資料庫驅動
4. **關鍵：保留所有明確指定的版本號**
   - `spring-boot-starter-aop` 必須使用 `<version>4.0.0-M2</version>`（這是關鍵！忽略會導致編譯失敗）
   - Swagger 使用 `${springdoc.version}` 引用 properties 中的版本
   - Lombok 使用 `${lombok.version}` 並設定 `<scope>provided</scope>`
   - 所有第三方套件（如 Commons Collections）都必須保留明確版本號

---

### Step 2.3: 生成配置檔案

創建 application.yml、logback 等配置檔案。

**application.yml:**

```yaml
spring:
  application:
    name: { projectName }
  profiles:
    active: dev
  jpa: # 僅在 useDatabase=true 時加入
    open-in-view: false

server:
  port: { serverPort }
```

**application-dev.yml:**

```yaml
spring:
  datasource:           # 僅在 useDatabase=true 時加入
    url: {dbUrl}
    username: {dbUsername}
    password: {dbPassword}
    hikari:
      maximum-pool-size: 10
      minimum-idle: 5
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000
      pool-name: {AppName}HikariPool
  jpa:                  # 僅在 useDatabase=true 時加入
    hibernate:
      ddl-auto: validate
    show-sql: true
    properties:
      hibernate.format_sql: true
      hibernate.jdbc.fetch_size: 500
      hibernate.jdbc.batch_size: 50
      hibernate.order_inserts: true
      hibernate.order_updates: true

springdoc:
  api-docs:
    enabled: true
  swagger-ui:
    enabled: true
```

**application-prd.yml:**

```yaml
spring:
  datasource: # 僅在 useDatabase=true 時加入
    url: ${DB_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
    hikari:
      maximum-pool-size: 20
      minimum-idle: 10
  jpa: # 僅在 useDatabase=true 時加入
    hibernate:
      ddl-auto: validate
    show-sql: false

springdoc:
  api-docs:
    enabled: false
  swagger-ui:
    enabled: false
```

**logback-spring.xml:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <springProfile name="default">
        <property name="LOG_PATH" value="logs"/>
    </springProfile>
    <springProfile name="dev">
        <property name="LOG_PATH" value="logs/dev"/>
    </springProfile>
    <springProfile name="prod">
        <property name="LOG_PATH" value="logs/prod"/>
    </springProfile>
    <property name="LOG_FILE_NAME" value="{projectName}"/>
    <property name="PATTERN" value="%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread][%X{traceId}] %-5level %logger{36} - %msg%n"/>

    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${PATTERN}</pattern>
        </encoder>
    </appender>

    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_PATH}/${LOG_FILE_NAME}.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_PATH}/${LOG_FILE_NAME}.%d{yyyy-MM-dd}.log.zip</fileNamePattern>
            <maxHistory>30</maxHistory>
            <totalSizeCap>1GB</totalSizeCap>
        </rollingPolicy>
        <encoder>
            <pattern>${PATTERN}</pattern>
        </encoder>
    </appender>

    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
    </root>
</configuration>
```

**.gitignore:**

```gitignore
HELP.md
target/
.mvn/wrapper/maven-wrapper.jar
!**/src/main/**/target/
!**/src/test/**/target/

### LOG ###
*.log
logs/

### STS ###
.apt_generated
.classpath
.factorypath
.project
.settings
.springBeans
.sts4-cache

### IntelliJ IDEA ###
.idea
*.iws
*.iml
*.ipr

### NetBeans ###
/nbproject/private/
/nbbuild/
/dist/
/nbdist/
/.nb-gradle/
build/
!**/src/main/**/build/
!**/src/test/**/build/

### VS Code ###
.vscode/
```

---

### Step 2.4: 生成核心框架程式碼

創建 AOP、annotation 和 web response 等核心程式碼。

**{AppName}Application.java:**

```java
package {basePackage}.{appName};

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class {AppName}Application {

    public static void main(String[] args) {
        SpringApplication.run({AppName}Application.class, args);
    }
}
```

**core/annotation/ApiLog.java:**

```java
package {basePackage}.{appName}.core.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface ApiLog {
}
```

**core/aop/ApiLogAspect.java:**

```java
package {basePackage}.{appName}.core.aop;

import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;
import org.springframework.web.multipart.MultipartFile;
import tools.jackson.databind.ObjectMapper;

import java.util.Arrays;

@Aspect
@Component
@Order(0)
@Slf4j
@RequiredArgsConstructor
public class ApiLogAspect {

    private final ObjectMapper objectMapper;

    @Around("@annotation({basePackage}.{appName}.core.annotation.ApiLog)")
    public Object logApi(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();

        ServletRequestAttributes attrs = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attrs != null ? attrs.getRequest() : null;

        String method = request != null ? request.getMethod() : "N/A";
        String uri = request != null ? request.getRequestURI() : "N/A";

        Object[] args = filterArgs(pjp.getArgs());
        log.info("[API START] {} {}, Request: {}", method, uri, toJson(args));

        Object result = null;
        try {
            result = pjp.proceed();
            return result;
        } catch (Throwable ex) {
            log.error("[API EXCEPTION] {} {} - {}", method, uri, ex.getMessage(), ex);
            throw ex;
        } finally {
            long cost = System.currentTimeMillis() - start;
            log.info("[API END] {} {} ({} ms), Response: {}", method, uri, cost, toJson(result));
        }
    }

    private Object[] filterArgs(Object[] args) {
        if (args == null) {
            return new Object[0];
        }
        return Arrays.stream(args)
                .filter(arg -> !(arg instanceof HttpServletRequest)
                    && !(arg instanceof HttpServletResponse)
                    && !(arg instanceof MultipartFile))
                .toArray();
    }

    private String toJson(Object obj) {
        if (obj == null) {
            return "";
        }
        try {
            return objectMapper.writeValueAsString(obj);
        } catch (Exception e) {
            return String.valueOf(obj);
        }
    }
}
```

**core/web/BaseResponse.java:**

```java
package {basePackage}.{appName}.core.web;

import lombok.Getter;
import lombok.Setter;

import java.io.Serial;
import java.io.Serializable;

@Getter
@Setter
public class BaseResponse<T> implements Serializable {

    @Serial
    private static final long serialVersionUID = -1L;

    private int code;
    private T data;
    private boolean success;
    private String message;

    public static BaseResponse<Void> success() {
        return success(null);
    }

    public static <T> BaseResponse<T> success(T data) {
        BaseResponse<T> res = new BaseResponse<>();
        res.setCode(200);
        res.setSuccess(true);
        res.setData(data);
        res.setMessage("OK");
        return res;
    }

    public static BaseResponse<Void> fail(int code, String message) {
        BaseResponse<Void> res = new BaseResponse<>();
        res.setCode(code);
        res.setSuccess(false);
        res.setMessage(message);
        return res;
    }
}
```

**core/web/ApiResponse.java:**

```java
package {basePackage}.{appName}.core.web;

import org.springframework.http.ResponseEntity;

public final class ApiResponse {

    private ApiResponse() {
    }

    public static <T> ResponseEntity<BaseResponse<T>> ok(T data) {
        return ResponseEntity.ok(BaseResponse.success(data));
    }

    public static ResponseEntity<BaseResponse<Void>> ok() {
        return ok(null);
    }
}
```

---

### Step 2.5: 生成 TraceId 追蹤機制

實作全鏈路請求追蹤功能，透過 Filter 和 MDC 機制在每個 HTTP 請求中注入唯一的 TraceId。

**core/filter/TraceFilter.java:**

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
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;

        // 從請求標頭取得 traceId，若無則自動生成
        String traceId = httpRequest.getHeader(TRACE_ID_HEADER);
        if (StringUtils.isBlank(traceId)) {
            long now = System.currentTimeMillis();
            traceId = Long.toString(now);
        }

        // 將 traceId 存入 MDC，供 logback 使用
        MDC.put(TRACE_ID_MDC_KEY, traceId);

        // 將 traceId 加入回應標頭
        httpResponse.setHeader(TRACE_ID_HEADER, traceId);

        try {
            chain.doFilter(request, response);
        } finally {
            // 請求完成後清除 MDC，避免記憶體洩漏
            MDC.remove(TRACE_ID_MDC_KEY);
        }
    }
}
```

**執行動作：**

1. 創建 `core/filter` 資料夾
2. 生成 `TraceFilter.java`
3. 替換 `{basePackage}`, `{appName}` 為實際值

**TraceId 機制說明：**

- **X-Trace-Id Header**：客戶端可透過此標頭傳入 traceId，若未傳入則自動生成
- **MDC (Mapped Diagnostic Context)**：SLF4J 提供的執行緒安全容器，可在同一請求的所有日誌中注入 traceId
- **Order(1)**：確保此 Filter 在其他 Filter 之前執行，優先注入 traceId
- **finally block**：請求結束後必須清除 MDC，避免在使用執行緒池時造成 traceId 污染

**logback-spring.xml 已包含 traceId 支援：**

在 Step 2.3 生成的 logback 配置中，PATTERN 已包含 `%X{traceId}`：

```xml
<property name="PATTERN" value="%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread][%X{traceId}] %-5level %logger{36} - %msg%n"/>
```

**日誌輸出範例：**

```
2026-02-13 09:45:12.345 [http-nio-8080-exec-1][1739413512345] INFO  c.e.demo.controller.HelloController - Processing request
2026-02-13 09:45:12.456 [http-nio-8080-exec-1][1739413512345] INFO  c.e.demo.service.HelloService - Business logic executed
```

---

### Step 2.6: 生成測試基礎程式碼

創建基本的測試類別。

**注意**：HelloController 和 HelloControllerTest 在 Phase 5 才生成，此階段只生成 ApplicationTests。

**{AppName}ApplicationTests.java:**

```java
package {basePackage}.{appName};

import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
class {AppName}ApplicationTests {

    @Test
    void contextLoads() {
    }
}
```

**輸出：**

- 完整的 Spring Boot 4 專案骨架
- 所有必要的配置檔案和核心程式碼
- **不包含**：README.md（Phase 5）、HelloController（Phase 5）

**後續動作建議：**

完成此 Phase 後，建議執行 `mvn clean compile` 驗證專案可以編譯成功。
