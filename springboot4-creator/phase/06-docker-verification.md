# Phase 6: Docker 容器化驗證

將專案打包成 Docker 映像檔，並驗證容器運行和 API 正常運作。

## 執行流程

### Step 6.1: 生成 Dockerfile

創建多階段建構的 Dockerfile，優化映像檔大小。

**Dockerfile:**

```dockerfile
# =========================
# 第一階段：Build 階段
# =========================
FROM eclipse-temurin:25-jdk-jammy AS builder

WORKDIR /build
COPY pom.xml .

RUN apt-get update && apt-get install -y maven &&  mvn -B dependency:go-offline
COPY src ./src
RUN mvn -B clean package -DskipTests

# =========================
# 第二階段：Runtime 階段
# =========================
FROM eclipse-temurin:25-jre-jammy

RUN ln -sf /usr/share/zoneinfo/Asia/Taipei /etc/localtime && echo 'Asia/Taipei' > /etc/timezone

ENV MEMORY_MIN=512M
ENV MEMORY_MAX=1G
ENV SPRING_PROFILES_ACTIVE=dev

WORKDIR /app

COPY --from=builder /build/target/*.jar /app/app.jar

ENTRYPOINT ["sh", "-c", "java -XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0 -Xms${MEMORY_MIN} -Xmx${MEMORY_MAX} -jar /app/app.jar"]
```

**Dockerfile 說明：**

- **多階段建構**：分離建置和執行環境，減少映像檔大小
- **Builder 階段**：使用 JDK 25 編譯專案
- **Runtime 階段**：使用 JRE 25 執行應用程式
- **時區設定**：預設為 Asia/Taipei
- **記憶體配置**：可透過環境變數調整
- **Container-aware JVM**：自動偵測容器資源限制

---

### Step 6.2: 生成 .dockerignore

創建 .dockerignore 避免不必要的檔案被複製到映像檔。

**.dockerignore:**

```ignore
# Maven
target/
!target/*.jar
pom.xml.tag
pom.xml.releaseBackup
pom.xml.versionsBackup
pom.xml.next
release.properties
dependency-reduced-pom.xml
buildNumber.properties
.mvn/timing.properties
.mvn/wrapper/maven-wrapper.jar

# Logs
logs/
*.log
*.log.*
*.zip

# IDE
.idea/
.vscode/
*.iml
*.iws
*.ipr
.settings/
.classpath
.project

# OS
.DS_Store
Thumbs.db

# Git
.git/
.gitignore
.gitattributes

# Docker
Dockerfile
.dockerignore

# Documentation
README.md

# Test reports
surefire-reports/
```

---

### Step 6.3: 建構 Docker 映像檔

執行 Docker build 指令建立映像檔。

**執行指令：**

**前提**：確認你在專案根目錄下執行（包含 Dockerfile 的目錄）

```bash
cd {projectPath}
docker build -t {projectName}:latest .
```

**預期輸出：**

```
[+] Building 45.2s (14/14) FINISHED
 => [internal] load build definition from Dockerfile
 => [internal] load .dockerignore
 => [builder 1/5] FROM eclipse-temurin:25-jdk-jammy
 => [builder 2/5] WORKDIR /build
 => [builder 3/5] COPY pom.xml .
 => [builder 4/5] RUN apt-get update && apt-get install -y maven ...
 => [builder 5/5] RUN mvn -B clean package -DskipTests
 => [stage-1 1/3] FROM eclipse-temurin:25-jre-jammy
 => [stage-1 2/3] RUN ln -sf /usr/share/zoneinfo/Asia/Taipei ...
 => [stage-1 3/3] COPY --from=builder /build/target/*.jar /app/app.jar
 => exporting to image
 => => exporting layers
 => => writing image sha256:xxxxx...
 => => naming to docker.io/library/{projectName}:latest
```

**驗證映像檔建立成功：**

```bash
docker images | grep {projectName}
```

預期看到：

```
{projectName}   latest   xxxxx   1 minute ago   XXX MB
```

**潛在問題排除：**

- **Docker daemon not running**
  - 解決：啟動 Docker Desktop 或 Docker 服務
- **Build 過程中 Maven 下載依賴失敗**
  - 解決：檢查網路連線，或使用國內 Maven 鏡像
- **Out of disk space**
  - 解決：清理舊的 Docker 映像檔 `docker system prune -a`

---

### Step 6.4: 啟動 Docker 容器

執行 Docker run 指令啟動容器。

**Dev 環境執行：**

```bash
docker run -d -p {serverPort}:{serverPort} -e SPRING_PROFILES_ACTIVE=dev --name {projectName}-dev {projectName}:latest
```

**參數說明：**

- `-d`：背景執行
- `-p {serverPort}:{serverPort}`：映射端口
- `-e SPRING_PROFILES_ACTIVE=dev`：指定環境為 dev
- `--name {projectName}-dev`：容器名稱
- `{projectName}:latest`：映像檔名稱

**檢查容器狀態：**

```bash
docker ps | grep {projectName}
```

預期輸出：

```
CONTAINER ID   IMAGE                    STATUS         PORTS                      NAMES
xxxxx          {projectName}:latest     Up 10 seconds  0.0.0.0:{serverPort}->{serverPort}   {projectName}-dev
```

**查看容器日誌：**

```bash
docker logs {projectName}-dev
```

**預期看到 Spring Boot 啟動成功日誌：**

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/

 :: Spring Boot ::                (v4.0.1)

... Application startup completed in X.XXX seconds ...
```

**潛在問題排除：**

- **Port already in use**
  - 解決：更換端口或停止佔用端口的服務
- **Container exited immediately**
  - 解決：查看日誌 `docker logs {projectName}-dev`
- **Out of memory**
  - 解決：增加記憶體配置 `docker run ... -e MEMORY_MAX=2G ...`

---

### Step 6.5: 測試容器內的 Hello World API

使用 curl 或瀏覽器測試 API 是否正常運作。

**測試 1：基本 Hello Endpoint**

```bash
curl http://localhost:{serverPort}/api/hello
```

**預期回應：**

```json
{
  "code": 200,
  "message": "OK",
  "success": true,
  "data": {
    "message": "Hello from {projectName}!",
    "timestamp": "2026-02-12T..."
  }
}
```

**測試 2：帶名稱參數的 Hello Endpoint**

```bash
curl http://localhost:{serverPort}/api/hello/Docker
```

**預期回應：**

```json
{
  "code": 200,
  "message": "OK",
  "success": true,
  "data": {
    "message": "Hello, Docker!",
    "timestamp": "2026-02-12T..."
  }
}
```

**測試 3：Swagger UI**

開啟瀏覽器訪問：

```
http://localhost:{serverPort}/swagger-ui.html
```

預期看到完整的 API 文件介面。

---

### Step 6.6: 驗證結果與清理

完成測試後，驗證結果並清理容器。

**驗證清單：**

- Docker 映像檔建立成功
- 容器成功啟動（狀態為 Up）
- Spring Boot 應用程式正常啟動
- Hello World API 回應正確
- API 回應格式符合 BaseResponse 規範
- Swagger UI 可正常訪問
- 容器日誌無錯誤訊息

**停止並移除容器：**

```bash
# 停止容器
docker stop {projectName}-dev

# 移除容器
docker rm {projectName}-dev
```

**（可選）移除映像檔：**

```bash
docker rmi {projectName}:latest
```

---

### Step 6.7: 輸出完成訊息

顯示 Docker 驗證完成訊息。

**完成訊息：**

```
Docker 容器化驗證完成！

Docker 映像檔資訊：
==================
映像檔名稱: {projectName}:latest
映像檔大小: ~XXX MB
==================

驗證結果：
- Dockerfile 建立成功
- .dockerignore 建立成功
- Docker 映像檔建構成功
- 容器成功啟動
- Spring Boot 應用程式正常運行
- Hello World API 測試通過
- Swagger UI 正常訪問

容器管理指令：
==================

建構映像檔：
   docker build -t {projectName}:latest .

啟動容器（Dev 環境）：
   docker run -d -p {serverPort}:{serverPort} -e SPRING_PROFILES_ACTIVE=dev --name {projectName}-dev {projectName}:latest

啟動容器（Prd 環境）：
   docker run -d -p {serverPort}:{serverPort} -e SPRING_PROFILES_ACTIVE=prd -e MEMORY_MIN=1G -e MEMORY_MAX=2G --name {projectName}-prd {projectName}:latest

查看容器狀態：
   docker ps | grep {projectName}

查看容器日誌：
   docker logs {projectName}-dev

停止容器：
   docker stop {projectName}-dev

移除容器：
   docker rm {projectName}-dev

==================

專案已完整生成並通過所有驗證！

你可以開始開發業務功能了！
詳細說明請參閱 README.md
```

---

## 輸出

- Dockerfile（多階段建構）
- .dockerignore（排除不必要檔案）
- Docker 映像檔（已建構並測試）
- 容器運行驗證報告
- Docker 使用指南
