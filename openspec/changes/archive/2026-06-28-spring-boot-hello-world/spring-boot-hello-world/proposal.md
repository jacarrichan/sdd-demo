# Spring Boot Hello World Demo

## 问题背景 (Why)

需要一个基础的 Spring Boot 项目示例，用于：
- 验证开发环境配置是否正确
- 作为后续 Spring Boot 项目的模板
- 学习和演示 Spring Boot 基础功能

## 目标 (What)

创建一个可运行的 Spring Boot Hello World 应用程序：
- 使用 Spring Boot 3.x + Java 17
- 提供一个简单的 REST API 端点 `/hello`
- 返回 "Hello World" 或自定义问候消息
- 包含 Maven 构建配置
- 包含基本的项目结构

## 范围

### 包含
- 基础项目结构（src/main/java, src/main/resources）
- Spring Boot Starter Web 依赖
- 一个 REST Controller
- Maven pom.xml 配置
- application.properties 配置文件
- 可执行的 Spring Boot 主类

### 不包含
- 数据库集成
- 安全认证
- 单元测试（保持最小化）
- Docker 化部署

## 非目标

- 生产级配置
- 复杂业务逻辑
- 微服务架构
