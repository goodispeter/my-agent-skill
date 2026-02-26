# Spring Boot 4 Creator Agent Skill

自動生成 Spring Boot 4.0.1 專案的 AI Agent Skill，基於 ASS (Activity Sync Service) 專案模板。

## 功能特色

- **Spring Boot 4.0.1** + Java 25
- **三層式架構** (Controller → Service → Repository)
- **Spring Boot 4 聲明式 HTTP API**（條件性生成）
- **Logback 日誌配置**（多環境支援）
- **AOP 日誌切面** (@ApiLog annotation)
- **統一回應格式** (BaseResponse)
- **Swagger UI** 固定整合
- **JPA + PostgreSQL/MySQL**（條件性生成）
- **完整測試架構**
- **Docker 容器化支援**（多階段建構）
- **自動生成 Hello World API 並測試驗證**
- **自動打包成 ZIP 檔**（輸出到根目錄，方便分享）

## 技術棧

### 固定技術

- **Framework**: Spring Boot 4.0.1
- **Java**: 25
- **Build Tool**: Maven
- **AOP**: spring-boot-starter-aspectj
- **API 文件**: Swagger 3.0.0 (springdoc-openapi)
- **日誌**: Logback
- **工具**: Lombok, Commons Collections

### 條件性技術

- **HTTP Client**: Spring Boot 4 內建聲明式 HTTP API (RestClient/HttpExchange)
- **Database**: JPA + Hibernate (當需要資料庫時)
  - PostgreSQL 或 MySQL Driver
  - Hypersistence Utils
- **Validation**: Spring Boot Validation

## 生成的專案結構

```
{projectName}/
├── pom.xml                           # Maven 配置
├── .gitignore
├── Dockerfile                        # Docker 多階段建構配置
├── .dockerignore                     # Docker 忽略檔案
├── README.md                         # 專案說明文件
├── src/
│   ├── main/
│   │   ├── java/{basePackage}/{appName}/
│   │   │   ├── {AppName}Application.java
│   │   │   ├── config/              # 配置類別
│   │   │   │   └── RestClientConfig.java        (條件性)
│   │   │   ├── controller/          # REST API 控制器
│   │   │   │   ├── HelloController.java         (自動生成)
│   │   │   │   └── {Entity}Controller.java      (模組骨架)
│   │   │   ├── service/             # 業務邏輯層
│   │   │   │   ├── {Entity}Service.java
│   │   │   │   └── impl/{Entity}ServiceImpl.java
│   │   │   ├── repository/          # 資料訪問層 (條件性)
│   │   │   │   └── {Entity}Repository.java
│   │   │   ├── domain/              # JPA 實體 (條件性)
│   │   │   │   └── {Entity}.java
│   │   │   ├── dto/                 # 資料傳輸物件
│   │   │   │   ├── request/{Entity}Req.java
│   │   │   │   └── response/{Entity}Rsp.java
│   │   │   ├── external/            # 外部 API (條件性)
│   │   │   │   ├── api/{ApiName}Api.java
│   │   │   │   ├── client/
│   │   │   │   │   ├── BaseApiClient.java
│   │   │   │   │   ├── {ApiName}ApiClient.java
│   │   │   │   │   └── impl/{ApiName}ApiClientImpl.java
│   │   │   │   ├── request/
│   │   │   │   └── response/
│   │   │   ├── core/                # 核心功能
│   │   │   │   ├── annotation/
│   │   │   │   │   └── ApiLog.java
│   │   │   │   ├── aop/
│   │   │   │   │   └── ApiLogAspect.java
│   │   │   │   └── web/
│   │   │   │       ├── BaseResponse.java
│   │   │   │       └── ApiResponse.java
│   │   │   ├── enums/
│   │   │   └── util/
│   │   └── resources/
│   │       ├── application.yml
│   │       ├── application-dev.yml
│   │       ├── application-prd.yml
│   │       ├── logback-spring.xml
│   │       ├── static/
│   │       └── templates/
│   └── test/
│       └── java/{basePackage}/{appName}/
│           └── {AppName}ApplicationTests.java
└── logs/                            # 日誌目錄 (gitignore)
```

## 執行流程

此 Skill 分為 **6 個 Phases**：

| Phase       | 說明                                   | 詳細文件                                                             |
| ----------- | -------------------------------------- | -------------------------------------------------------------------- |
| **Phase 1** | 需求探索                               | [01-requirements-discovery.md](phase/01-requirements-discovery.md)   |
| **Phase 2** | 專案骨架生成                           | [02-project-scaffold.md](phase/02-project-scaffold.md)               |
| **Phase 3** | 外部 API 整合與 TraceId 配置（條件性） | [03-external-api-setup.md](phase/03-external-api-setup.md)           |
| **Phase 4** | 業務模組程式碼生成                     | [04-business-modules.md](phase/04-business-modules.md)               |
| **Phase 5** | 驗證與完成                             | [05-verification-completion.md](phase/05-verification-completion.md) |
| **Phase 6** | Docker 容器化驗證                      | [06-docker-verification.md](phase/06-docker-verification.md)         |

---

## 使用方式

### 前置需求

- JDK 25
- Maven 3.x
- Docker（用於容器化部署）

### 執行 Skill

直接對 AI Agent 說：

```
請使用 Spring Boot 4 Creator Skill 幫我建立一個新專案
```

AI 會引導你完成需求探索、專案生成、驗證測試和 Docker 容器化部署。

### 生成成果

- 可立即編譯運行的 Spring Boot 4 專案
- Hello World API（已測試通過）
- 完整的三層架構骨架（含 TODO 註解）
- Swagger UI 整合
- Docker 容器化支援
- README 使用說明

## 參考專案

此 Skill 基於 ASS (Activity Sync Service) 專案模板：

- 位置：`D:\workspace\side\ass`
- 包含完整的 Spring Boot 4 + 聲明式 HTTP API + JPA 架構

---

**Generated on**: 2026-02-12  
**Version**: 1.1.0  
**Author**: KuanYu Pan
