# Spring Boot Hello World Demo

[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.3.0-brightgreen)](https://spring.io/projects/spring-boot)
[![Java](https://img.shields.io/badge/Java-17-orange)](https://openjdk.org/)
[![Maven](https://img.shields.io/badge/Maven-3.8+-blue)](https://maven.apache.org/)

一个简洁的 Spring Boot Hello World 示例项目，用于演示 Spring Boot 基础 Web 开发。

## 功能特性

- RESTful API 端点 `/hello`
- 支持可选的 `name` 参数返回个性化问候
- 使用 Spring Boot 3.3.0 + Java 17
- Maven 构建工具

## API 端点

### GET /hello
返回默认问候语。

**请求：**
```bash
curl http://localhost:8080/hello
```

**响应：**
```
Hello World
```

### GET /hello?name={name}
返回个性化问候语。

**请求：**
```bash
curl "http://localhost:8080/hello?name=Spring"
```

**响应：**
```
Hello Spring
```

## 快速开始

### 环境要求

- Java 17 或更高版本
- Maven 3.8+ 或 Gradle
- （可选）IDE：IntelliJ IDEA、VS Code、Eclipse

### 构建项目

```bash
mvn clean package
```

### 运行应用

#### 方式一：使用 Maven
```bash
mvn spring-boot:run
```

#### 方式二：运行 JAR 文件
```bash
java -jar target/demo-0.0.1-SNAPSHOT.jar
```

#### 方式三：直接运行主类
```bash
java -cp target/classes:target/dependency/* com.example.demo.DemoApplication
```

### 验证运行

应用启动后，访问：
- http://localhost:8080/hello
- http://localhost:8080/hello?name=YourName

## 项目结构

```
.
├── pom.xml                              # Maven 配置
├── src/
│   └── main/
│       ├── java/com/example/demo/
│       │   ├── DemoApplication.java      # Spring Boot 启动类
│       │   └── controller/
│       │       └── HelloController.java  # REST API 控制器
│       └── resources/
│           └── application.properties    # 应用配置
├── docs/                                 # Comet 开发流程文档
│   └── superpowers/
│       ├── specs/                        # 设计文档
│       └── plans/                        # 实施计划
└── openspec/                             # OpenSpec 变更管理
    └── changes/archive/                  # 归档的变更记录
```

## 技术栈

| 技术 | 版本 | 用途 |
|------|------|------|
| Spring Boot | 3.3.0 | 应用框架 |
| Spring Web | 3.3.0 | Web 支持 |
| Java | 17 | 编程语言 |
| Maven | 3.8+ | 构建工具 |

## 开发流程

本项目使用 [Comet](https://github.com/anthropics/claude-code) 开发流程管理：

1. **Open** - 创建变更提案和设计文档
2. **Design** - 技术设计和实施计划
3. **Build** - 代码实现
4. **Verify** - 功能验证
5. **Archive** - 归档变更记录

详见 `docs/superpowers/` 和 `openspec/changes/archive/` 目录。

## 配置说明

### 修改端口

编辑 `src/main/resources/application.properties`：

```properties
server.port=9090
```

### 其他配置

```properties
# 应用名称
spring.application.name=my-demo

# 日志级别
logging.level.com.example.demo=DEBUG
```

## 常见问题

### 端口冲突

如果 8080 端口被占用，可以修改 `application.properties`：
```properties
server.port=8081
```

### 构建失败

确保 Java 版本为 17 或更高：
```bash
java -version
```

## 许可证

MIT License - 详见 [LICENSE](LICENSE) 文件（如有）

## 贡献

欢迎提交 Issue 和 Pull Request！

## 相关链接

- [Spring Boot 官方文档](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [Spring Web 指南](https://spring.io/guides/gs/rest-service/)
- [Maven 官方文档](https://maven.apache.org/guides/)

---

**生成日期：** 2026-06-28  
**作者：** @jacarrichan
