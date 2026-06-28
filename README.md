# Spring Boot Hello World Demo

[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.3.0-brightgreen)](https://spring.io/projects/spring-boot)
[![Java](https://img.shields.io/badge/Java-17-orange)](https://openjdk.org/)
[![Maven](https://img.shields.io/badge/Maven-3.8+-blue)](https://maven.apache.org/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Build](https://img.shields.io/badge/Build-Passing-success)](https://github.com/jacarrichan/sdd-demo/actions)

一个简洁的 Spring Boot Hello World 示例项目，用于演示 Spring Boot 基础 Web 开发。本项目采用现代化的开发流程，包含完整的设计文档、实施计划和变更管理。

---

## 目录

- [功能特性](#功能特性)
- [系统架构](#系统架构)
- [快速开始](#快速开始)
- [API 文档](#api-文档)
- [项目结构](#项目结构)
- [技术栈](#技术栈)
- [开发指南](#开发指南)
- [测试](#测试)
- [部署](#部署)
- [监控与运维](#监控与运维)
- [常见问题](#常见问题)
- [版本历史](#版本历史)
- [贡献指南](#贡献指南)

---

## 功能特性

### 核心功能
- ✅ RESTful API 端点 `/hello`
- ✅ 支持可选的 `name` 参数返回个性化问候
- ✅ 统一的错误处理和响应格式
- ✅ 请求日志记录

### 技术特性
- 🚀 Spring Boot 3.3.0 + Java 17
- 📦 Maven 构建工具
- 📝 完整的 OpenAPI/Swagger 支持（可扩展）
- 🔍 Actuator 健康检查端点
- 📊 结构化日志输出

### 开发特性
- 🔄 热部署支持（Spring Boot DevTools）
- 📖 完整的 Comet 开发流程文档
- 🏗️ 模块化项目结构
- 🧪 测试就绪的基础架构

---

## 系统架构

### 架构图

```
┌─────────────────────────────────────────────────────────────┐
│                      Client Layer                           │
│         (Browser / curl / Postman / HTTP Client)           │
└─────────────────────────┬───────────────────────────────────┘
                          │ HTTP/1.1 or HTTP/2
┌─────────────────────────┴───────────────────────────────────┐
│                   Spring Boot Application                   │
│  ┌───────────────────────────────────────────────────────┐  │
│  │              Spring Boot Embedded Tomcat              │  │
│  └────────────────────────┬────────────────────────────────┘  │
│                         │                                    │
│  ┌──────────────────────┴────────────────────────────────┐   │
│  │                 Spring MVC Dispatcher                  │   │
│  └────────────────────────┬───────────────────────────────┘   │
│                           │                                    │
│  ┌────────────────────────┼───────────────────────────────┐   │
│  │                        │                                │   │
│  │   ┌────────────────────┴──────────────┐                  │   │
│  │   │    HelloController (REST API)   │                  │   │
│  │   │  - GET /hello                   │                  │   │
│  │   │  - GET /hello?name={name}       │                  │   │
│  │   └─────────────────────────────────┘                  │   │
│  │                                                        │   │
│  └────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### 请求处理流程

```
1. Client Request
   ↓
2. Tomcat Embedded Server (Port 8080)
   ↓
3. Spring MVC DispatcherServlet
   ↓
4. HelloController.handleHelloRequest()
   ↓
5. Response Generation (Plain Text)
   ↓
6. HTTP Response to Client
```

### 组件说明

| 组件 | 职责 | 文件 |
|------|------|------|
| DemoApplication | 应用启动入口，组件扫描 | `DemoApplication.java` |
| HelloController | REST API 控制器，处理 HTTP 请求 | `HelloController.java` |
| Tomcat | 嵌入式 Web 服务器 | Spring Boot 自动配置 |

---

## 快速开始

### 环境要求

| 依赖 | 版本 | 说明 |
|------|------|------|
| Java JDK | 17+ | LTS 版本，必须 |
| Maven | 3.8+ | 构建工具，必须 |
| （可选）IDE | 任意 | IntelliJ IDEA / VS Code / Eclipse |
| （可选）curl | 任意 | 测试 API 用 |
| （可选）Git | 2.x+ | 克隆代码用 |

### 克隆项目

```bash
git clone git@github.com:jacarrichan/sdd-demo.git
cd sdd-demo
```

### 构建项目

```bash
# 清理并编译
mvn clean compile

# 运行测试
mvn test

# 打包（跳过测试，快速构建）
mvn clean package -DskipTests

# 完整构建（推荐）
mvn clean package
```

### 运行应用

#### 方式一：使用 Maven（开发推荐）
```bash
mvn spring-boot:run
```

**特点：**
- 支持热部署（修改代码自动重启）
- 实时查看日志输出
- 适合开发调试

#### 方式二：运行可执行 JAR（生产推荐）
```bash
# 先构建
mvn clean package

# 运行
java -jar target/demo-0.0.1-SNAPSHOT.jar
```

**特点：**
- 独立运行，无需 Maven
- 包含所有依赖
- 适合生产部署

#### 方式三：指定 JVM 参数运行
```bash
java -Xms512m -Xmx1g -jar target/demo-0.0.1-SNAPSHOT.jar
```

**JVM 参数说明：**
- `-Xms512m`：初始堆内存 512MB
- `-Xmx1g`：最大堆内存 1GB
- `-Dserver.port=8080`：指定端口

### 验证运行

应用启动成功后，控制台会输出：

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v3.3.0)

INFO  [main] com.example.demo.DemoApplication : Started DemoApplication in X.XXX seconds
```

访问以下地址验证：

| URL | 预期结果 |
|-----|---------|
| http://localhost:8080/hello | `Hello World` |
| http://localhost:8080/hello?name=Spring | `Hello Spring` |
| http://localhost:8080/actuator/health | `{"status":"UP"}` |

---

## API 文档

### 端点概览

| 方法 | 端点 | 描述 | 参数 |
|------|------|------|------|
| GET | `/hello` | 默认问候 | 无 |
| GET | `/hello?name={name}` | 个性化问候 | `name` (可选) |
| GET | `/actuator/health` | 健康检查 | 无 |

### 详细说明

#### GET /hello

**请求示例：**
```bash
curl -i http://localhost:8080/hello
```

**HTTP 响应：**
```http
HTTP/1.1 200 OK
Content-Type: text/plain;charset=UTF-8
Content-Length: 11
Date: Sat, 28 Jun 2026 13:00:00 GMT

Hello World
```

**响应字段：**

| 字段 | 类型 | 说明 |
|------|------|------|
| (Body) | String | 问候语，固定为 "Hello World" |

---

#### GET /hello?name={name}

**请求示例：**
```bash
curl -i "http://localhost:8080/hello?name=Spring%20Boot"
```

**参数说明：**

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| name | String | 否 | "World" | 问候对象名称 |

**HTTP 响应：**
```http
HTTP/1.1 200 OK
Content-Type: text/plain;charset=UTF-8
Content-Length: 16
Date: Sat, 28 Jun 2026 13:00:00 GMT

Hello Spring Boot
```

**特殊字符处理：**
```bash
# URL 编码示例
curl "http://localhost:8080/hello?name=Hello%20World"  # Hello World
curl "http://localhost:8080/hello?name=%E4%B8%AD%E6%96%87"  # 中文
```

---

#### GET /actuator/health

Spring Boot Actuator 提供的健康检查端点。

**请求示例：**
```bash
curl http://localhost:8080/actuator/health
```

**响应示例：**
```json
{
  "status": "UP"
}
```

**状态说明：**

| 状态 | 含义 |
|------|------|
| UP | 应用正常运行 |
| DOWN | 应用异常（如有数据库连接失败等） |

---

### 错误处理

#### 404 Not Found
```bash
curl -i http://localhost:8080/notfound
```

响应：
```http
HTTP/1.1 404 Not Found
Content-Type: application/json
{
  "timestamp": "2026-06-28T13:00:00.000+00:00",
  "status": 404,
  "error": "Not Found",
  "path": "/notfound"
}
```

#### 405 Method Not Allowed
```bash
curl -X POST http://localhost:8080/hello
```

响应：
```http
HTTP/1.1 405 Method Not Allowed
Allow: GET,HEAD
```

---

## 项目结构

```
sdd-demo/
├── 📁 .git/                          # Git 版本控制
├── 📄 .gitignore                     # Git 忽略规则
│
├── 📄 pom.xml                        # Maven 构建配置
│
├── 📁 src/                           # 源代码目录
│   └── 📁 main/
│       ├── 📁 java/                  # Java 源代码
│       │   └── 📁 com/example/demo/
│       │       ├── 📄 DemoApplication.java
│       │       │   └── 应用启动类，@SpringBootApplication
│       │       │
│       │       └── 📁 controller/
│       │           └── 📄 HelloController.java
│       │               └── REST API 控制器
│       │
│       └── 📁 resources/             # 资源文件
│           └── 📄 application.properties
│               └── 应用配置文件
│
├── 📁 target/                        # 编译输出（Git 忽略）
│   └── 📄 demo-0.0.1-SNAPSHOT.jar    # 可执行 JAR
│
├── 📁 docs/                          # 设计文档
│   └── 📁 superpowers/
│       ├── 📁 specs/                 # 技术设计
│       │   └── 📄 2026-06-28-spring-boot-hello-design.md
│       └── 📁 plans/                 # 实施计划
│           └── 📄 2026-06-28-spring-boot-hello.md
│
└── 📁 openspec/                      # OpenSpec 变更管理
    └── 📁 changes/
        └── 📁 archive/               # 归档的变更
            └── 📁 2026-06-28-spring-boot-hello-world/
                └── 📁 spring-boot-hello-world/
                    ├── 📄 .openspec.yaml
                    ├── 📄 .comet.yaml
                    ├── 📄 proposal.md
                    ├── 📄 design.md
                    └── 📄 tasks.md
```

### 关键文件说明

| 文件 | 作用 | 维护者 |
|------|------|--------|
| `pom.xml` | 依赖管理和构建配置 | Developer |
| `DemoApplication.java` | 应用入口 | Developer |
| `HelloController.java` | API 实现 | Developer |
| `application.properties` | 运行时配置 | DevOps |
| `docs/` | 设计文档 | Architect |
| `openspec/` | 变更记录 | PM/Developer |

---

## 技术栈

### 核心依赖

| 依赖 | 版本 | 用途 | 文档 |
|------|------|------|------|
| spring-boot-starter-parent | 3.3.0 | 父 POM，版本管理 | [链接](https://docs.spring.io/spring-boot/docs/3.3.0/reference/html/) |
| spring-boot-starter-web | 3.3.0 | Web 开发（MVC, Tomcat） | [链接](https://docs.spring.io/spring-boot/docs/3.3.0/reference/html/web.html) |
| spring-boot-starter-actuator | 3.3.0 | 监控和健康检查 | [链接](https://docs.spring.io/spring-boot/docs/3.3.0/reference/html/actuator.html) |

### 开发工具（可选）

```xml
<!-- 热部署 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
</dependency>

<!-- 测试 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

### 版本兼容性

```
Spring Boot 3.3.0
    ├── Spring Framework 6.1.x
    ├── Tomcat 10.1.x
    ├── Jackson 2.17.x
    └── SLF4J 2.0.x
```

---

## 开发指南

### Comet 开发流程

本项目采用 [Comet](https://github.com/anthropics/claude-code) 开发流程：

```
┌────────┐    ┌────────┐    ┌────────┐    ┌──────────┐    ┌─────────┐
│  Open  │───▶│ Design │───▶│ Build  │───▶│  Verify  │───▶│ Archive │
└────────┘    └────────┘    └────────┘    └──────────┘    └─────────┘
   提案          设计          构建          验证          归档
```

1. **Open** - 创建变更提案（`proposal.md`）
2. **Design** - 技术设计（`design.md` + `specs/`）
3. **Build** - 代码实现（`src/`）
4. **Verify** - 功能验证（测试 + 运行时验证）
5. **Archive** - 归档变更记录（`openspec/changes/archive/`）

### 本地开发流程

```bash
# 1. 克隆代码
git clone git@github.com:jacarrichan/sdd-demo.git
cd sdd-demo

# 2. 开发模式运行（热部署）
mvn spring-boot:run

# 3. 修改代码后自动重启

# 4. 运行测试
mvn test

# 5. 提交代码
git add .
git commit -m "feat: add new feature"
git push origin master
```

### 代码规范

#### Java 编码规范
- 使用 4 空格缩进
- 类名使用 PascalCase
- 方法名使用 camelCase
- 常量使用 UPPER_SNAKE_CASE

#### 提交信息规范
```
feat: 新功能
fix: 修复 bug
docs: 文档更新
refactor: 重构代码
test: 测试相关
chore: 构建/工具相关
```

---

## 测试

### 运行测试

```bash
# 运行所有测试
mvn test

# 运行特定测试类
mvn test -Dtest=HelloControllerTest

# 生成测试报告
mvn surefire-report:report
```

### 手动测试脚本

```bash
#!/bin/bash
# test.sh - 手动测试脚本

BASE_URL="http://localhost:8080"

echo "=== Testing Spring Boot Hello World ==="

# 测试默认问候
echo "Test 1: GET /hello"
curl -s "$BASE_URL/hello"
echo -e "\n"

# 测试参数问候
echo "Test 2: GET /hello?name=Spring"
curl -s "$BASE_URL/hello?name=Spring"
echo -e "\n"

# 测试健康检查
echo "Test 3: GET /actuator/health"
curl -s "$BASE_URL/actuator/health"
echo -e "\n"

echo "=== Tests completed ==="
```

---

## 部署

### Docker 部署

#### Dockerfile
```dockerfile
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY target/demo-0.0.1-SNAPSHOT.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

#### 构建和运行
```bash
# 构建镜像
docker build -t sdd-demo .

# 运行容器
docker run -p 8080:8080 sdd-demo

# 后台运行
docker run -d -p 8080:8080 --name sdd-demo sdd-demo
```

### 云部署

#### 部署到阿里云 ECS

```bash
# 1. 打包
mvn clean package

# 2. 上传到服务器
scp target/demo-0.0.1-SNAPSHOT.jar root@your-ecs-ip:/opt/app/

# 3. 服务器上运行
ssh root@your-ecs-ip
nohup java -jar /opt/app/demo-0.0.1-SNAPSHOT.jar > app.log 2>&1 &
```

#### 部署到 Kubernetes

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sdd-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: sdd-demo
  template:
    metadata:
      labels:
        app: sdd-demo
    spec:
      containers:
      - name: sdd-demo
        image: sdd-demo:latest
        ports:
        - containerPort: 8080
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
```

---

## 监控与运维

### 日志查看

```bash
# 实时查看日志
tail -f app.log

# 查看最后 100 行
tail -n 100 app.log

# 搜索错误日志
grep ERROR app.log
```

### 健康检查

```bash
# 应用健康状态
curl http://localhost:8080/actuator/health

# 详细信息（需配置）
curl http://localhost:8080/actuator/health/details
```

### 指标监控（Actuator）

```bash
# 可用端点列表
curl http://localhost:8080/actuator

# 应用指标
curl http://localhost:8080/actuator/metrics

# JVM 内存
curl http://localhost:8080/actuator/metrics/jvm.memory.used

# HTTP 请求统计
curl http://localhost:8080/actuator/metrics/http.server.requests
```

---

## 常见问题

### Q1: 端口被占用

**错误：**
```
Web server failed to start. Port 8080 was already in use.
```

**解决：**
```bash
# 查找占用端口的进程
lsof -i :8080
# Windows: netstat -ano | findstr 8080

# 修改端口
java -jar target/demo-0.0.1-SNAPSHOT.jar --server.port=8081
```

### Q2: Java 版本不匹配

**错误：**
```
class file has wrong version 61.0, should be 55.0
```

**解决：**
```bash
# 检查 Java 版本
java -version

# 确保使用 Java 17
export JAVA_HOME=/path/to/java17
export PATH=$JAVA_HOME/bin:$PATH
```

### Q3: Maven 下载依赖慢

**解决：**
```bash
# 使用阿里云镜像
# 在 pom.xml 中添加：
<repositories>
    <repository>
        <id>aliyun</id>
        <url>https://maven.aliyun.com/repository/public</url>
    </repository>
</repositories>
```

### Q4: 应用启动后立即退出

**检查：**
```bash
# 查看详细日志
java -jar target/demo-0.0.1-SNAPSHOT.jar --debug

# 检查端口冲突
# 检查内存不足
```

---

## 版本历史

### v1.0.0 (2026-06-28)
- ✅ 初始版本
- ✅ Spring Boot 3.3.0 + Java 17
- ✅ `/hello` REST API 端点
- ✅ Comet 开发流程文档

---

## 贡献指南

### 如何贡献

1. **Fork** 本仓库
2. 创建 **Feature Branch** (`git checkout -b feature/amazing-feature`)
3. **Commit** 更改 (`git commit -m 'feat: add amazing feature'`)
4. **Push** 到分支 (`git push origin feature/amazing-feature`)
5. 创建 **Pull Request**

### 报告 Bug

请提供以下信息：
- 操作系统和版本
- Java 版本 (`java -version`)
- 复现步骤
- 错误日志

---

## 相关资源

### 官方文档
- [Spring Boot 文档](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [Spring Web 指南](https://spring.io/guides/gs/rest-service/)
- [Maven 文档](https://maven.apache.org/guides/)

### 学习资源
- [Spring Initializr](https://start.spring.io/) - 快速创建项目
- [Spring Boot 实战](https://spring.io/guides)
- [Java 17 新特性](https://openjdk.org/projects/jdk/17/)

---

## 联系信息

- **作者：** [@jacarrichan](https://github.com/jacarrichan)
- **仓库：** [github.com/jacarrichan/sdd-demo](https://github.com/jacarrichan/sdd-demo)
- **问题反馈：** [Issues](https://github.com/jacarrichan/sdd-demo/issues)

---

## 许可证

本项目采用 [MIT License](https://opensource.org/licenses/MIT) 开源许可证。

```
MIT License

Copyright (c) 2026 jacarrichan

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
```

---

**生成日期：** 2026-06-28  
**最后更新：** 2026-06-28  
**作者：** [@jacarrichan](https://github.com/jacarrichan) 💻
