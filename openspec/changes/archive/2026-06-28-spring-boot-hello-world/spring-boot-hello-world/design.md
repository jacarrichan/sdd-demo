# Spring Boot Hello World 设计文档

## 高层架构决策

### 技术栈选择
- **框架**: Spring Boot 3.3.x
- **语言**: Java 17 (LTS 版本，长期支持)
- **构建工具**: Maven
- **打包**: JAR (可执行)

### 项目结构
```
C:\Users\jacarrichan\demo
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── example
    │   │           └── demo
    │   │               ├── DemoApplication.java      # Spring Boot 主类
    │   │               └── controller
    │   │                   └── HelloController.java  # REST Controller
    │   └── resources
    │       └── application.properties                # 应用配置
```

### API 设计

| 端点 | 方法 | 参数 | 响应 |
|------|------|------|------|
| `/hello` | GET | 无 | `Hello World` |
| `/hello?name={name}` | GET | name (可选) | `Hello {name}` |

### 依赖项
- `spring-boot-starter-web`: Web 应用支持

### 配置
- 默认端口: 8080
- 无需额外配置
