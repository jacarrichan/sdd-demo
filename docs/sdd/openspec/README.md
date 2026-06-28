# OpenSpec 工作原理详解

OpenSpec 是 Comet 双星开发流程中的 **WHAT** 系统，负责需求管理、变更提案和生命周期管理。

---

## 核心概念

OpenSpec 关注 **"做什么"** 和 **"为什么"**：
- 问题背景（Why）
- 变更目标（What）
- 范围界定（Scope）
- 变更生命周期管理

---

## 工作流程

```
┌─────────────────────────────────────────────────────────────────┐
│                      OpenSpec 工作流                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Phase 1: Open                                                  │
│  ├─ 创建 .openspec.yaml (change 元数据)                         │
│  ├─ 编写 proposal.md (Why + What)                              │
│  ├─ 编写 design.md (高层 How)                                   │
│  └─ 创建 tasks.md (任务清单)                                    │
│                                                                 │
│  ↓                                                              │
│                                                                 │
│  [Active Change]                                                │
│  ├─ 更新 tasks.md 状态                                          │
│  └─ 修改 .comet.yaml 状态                                      │
│                                                                 │
│  ↓                                                              │
│                                                                 │
│  Phase 5: Archive                                               │
│  ├─ 同步 delta spec → main spec                                │
│  ├─ 标注设计文档状态                                            │
│  └─ 移动到 archive/ 目录                                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 核心文件说明

### 1. .openspec.yaml - Change 元数据

**位置**: `openspec/changes/<name>/.openspec.yaml`

```yaml
name: spring-boot-hello-world
description: 实现一个 Spring Boot Hello World demo
author: user
created_at: 2026-06-28
capabilities:
  - spring-boot-app
```

**字段说明**:

| 字段 | 类型 | 说明 |
|------|------|------|
| `name` | string | change 的唯一标识 |
| `description` | string | 变更描述 |
| `author` | string | 创建者 |
| `created_at` | date | 创建日期 |
| `capabilities` | list | 关联的能力模块 |

---

### 2. proposal.md - 提案文档

**位置**: `openspec/changes/<name>/proposal.md`

**回答核心问题**: Why + What

```markdown
# Spring Boot Hello World Demo

## 问题背景 (Why)
- 验证开发环境配置
- 作为项目模板

## 目标 (What)
- 创建可运行的 Spring Boot 应用
- 提供 REST API 端点

## 范围
### 包含
- 基础项目结构
- Maven 配置

### 不包含
- 数据库集成
- 安全认证
```

---

### 3. design.md - 高层设计

**位置**: `openspec/changes/<name>/design.md`

**回答核心问题**: How（粗粒度）

```markdown
# 高层架构决策

## 技术栈选择
- Spring Boot 3.3.0
- Java 17
- Maven

## 项目结构
```
src/
├── main/
│   ├── java/...
│   └── resources/
```

## API 设计
| 端点 | 方法 | 响应 |
|------|------|------|
| /hello | GET | Hello World |
```

---

### 4. tasks.md - 任务清单

**位置**: `openspec/changes/<name>/tasks.md`

用于追踪各阶段任务完成状态。

```markdown
# 任务清单

## Phase 1: Open
- [x] 创建 change 目录
- [x] 编写 proposal.md

## Phase 3: Build
- [ ] 创建 pom.xml
- [ ] 编写主类
```

---

### 5. .comet.yaml - 状态追踪

**位置**: `openspec/changes/<name>/.comet.yaml`

**核心状态文件**，由 Comet 工具管理。

```yaml
workflow: full                    # 工作流类型
phase: build                      # 当前阶段
design_doc: null                  # 关联的设计文档
plan: null                        # 关联的实施计划
build_mode: null                  # 构建模式
isolation: branch                 # 隔离方式
verify_mode: full                 # 验证模式
verify_result: pending            # 验证结果
verified_at: null                 # 验证时间
archived: false                   # 是否归档
```

---

## 关键脚本解析

### comet-state.sh - 状态管理器

**功能**: 统一管理 `.comet.yaml` 状态

**子命令**:

| 子命令 | 功能 | 示例 |
|--------|------|------|
| `init` | 初始化 .comet.yaml | `bash comet-state.sh init <name> <workflow>` |
| `get` | 读取字段值 | `bash comet-state.sh get <name> <field>` |
| `set` | 更新字段值 | `bash comet-state.sh set <name> <field> <value>` |
| `check` | 阶段入口验证 | `bash comet-state.sh check <phase> <name>` |
| `scale` | 评估验证模式 | `bash comet-state.sh scale <name>` |

**初始化逻辑** (根据 workflow 设置不同默认值):

```bash
case "$workflow" in
  full)
    phase="design"        # 完整流程从 design 开始
    build_mode="null"
    ;;
  hotfix|tweak)
    phase="build"         # 快速修复从 build 开始
    build_mode="direct"
    verify_mode="light"
    ;;
esac
```

**验证逻辑**:
- `workflow`: full / hotfix / tweak
- `phase`: design / build / verify / archive
- `verify_result`: pending / pass / fail
- `archived`: true / false

---

### comet-archive.sh - 归档器

**功能**: 自动化归档流程

**执行步骤**:

1. **读取 .comet.yaml** - 提取 design_doc 和 plan 路径
2. **验证入口状态** - 确保 phase=archive, verify_result=pass, archived≠true
3. **检查归档目标** - 确保 archive/ 目录不存在
4. **同步 Delta Spec** - 将 `specs/<capability>/spec.md` 复制到主 spec
5. **标注设计文档** - 在 YAML frontmatter 添加 `archived-with`
6. **标注计划文档** - 同上
7. **移动 change 目录** - 从 `changes/<name>` 移动到 `changes/archive/<name>`
8. **更新 archived 状态** - 设置 `archived: true`

**关键代码**:

```bash
# 同步 delta specs
sync_delta_specs() {
  for delta_spec_dir in "$delta_root"/*/; do
    capability=$(basename "$delta_spec_dir")
    delta_spec="$delta_spec_dir/spec.md"
    main_spec="openspec/specs/$capability/spec.md"
    cp "$delta_spec" "$main_spec"
  done
}

# 标注 frontmatter
annotate_frontmatter() {
  # 在 --- 和 --- 之间添加 archived-with 字段
  awk -v archive="$ARCHIVE_NAME" '
    /^archived-with:/ { next }  # 跳过已存在的字段
    /^---/ && NR>1 {
      print "archived-with: " archive
      print; next
    }
    { print }
  '
}
```

---

## 文件结构总览

```
openspec/
├── config.yaml                    # 全局配置（可选）
│
├── changes/                       # 活跃 changes
│   ├── <name>/                    # 单个 change 目录
│   │   ├── .openspec.yaml         # change 元数据
│   │   ├── .comet.yaml            # Comet 状态追踪
│   │   ├── proposal.md            # 提案（Why + What）
│   │   ├── design.md              # 高层设计
│   │   ├── tasks.md               # 任务清单
│   │   └── specs/                 # delta 能力规格
│   │       └── <capability>/
│   │           └── spec.md
│   │
│   └── archive/                   # 归档目录
│       └── YYYY-MM-DD-<name>/
│           └── <change-files>/    # 归档的 change
│
└── specs/                         # 主能力规格
    └── <capability>/
        └── spec.md                # 从 delta 同步的主 spec
```

---

## 与其他组件的关系

```
OpenSpec (WHAT)                    Superpowers (HOW)
───────────────────────────────────────────────────────────

openspec/changes/<name>/
├── proposal.md ───────┐
├── design.md (高层) ────┤
└── .openspec.yaml       │          docs/superpowers/
                         │          ├── specs/
openspec/changes/<name>/ │          │   └── <date>-topic-design.md
└── .comet.yaml          │          └── plans/
    ├── design_doc ──────┘          │   └── <date>-feature.md
    │  = specs/YYYY-MM-DD-topic-design.md
    │
    └── plan
       = plans/YYYY-MM-DD-feature.md
       (文件头: change: <name>)
```

**OpenSpec 产出 → Superpowers 输入**:
- `proposal.md` → Design Doc 的需求来源
- `design.md` (高层) → Design Doc 的技术细化基础
- `.comet.yaml` → Design Doc 和 Plan 的关联点

---

## 使用场景

### 场景 1: 创建新变更

```bash
# 1. 创建 change 目录
mkdir -p openspec/changes/my-feature

# 2. 编写 proposal.md 和 design.md
# 3. 初始化 .comet.yaml
bash comet-state.sh init my-feature full

# 4. 进入 Phase 2 (Design)
```

### 场景 2: 查看变更状态

```bash
# 获取当前阶段
bash comet-state.sh get my-feature phase
# 输出: design

# 获取验证结果
bash comet-state.sh get my-feature verify_result
# 输出: pending
```

### 场景 3: 归档变更

```bash
# 1. 更新状态
bash comet-state.sh set my-feature phase archive
bash comet-state.sh set my-feature verify_result pass
bash comet-state.sh set my-feature archived true

# 2. 执行归档
bash comet-archive.sh my-feature
```

---

## 最佳实践

1. **提案先行** - 先写 proposal.md 明确 Why 和 What
2. **任务追踪** - 及时更新 tasks.md 中的 checkbox
3. **状态同步** - 文件状态与 .comet.yaml 保持一致
4. **归档闭环** - 完成归档后 design doc 和 plan 标注 archived-with
5. **Delta Spec** - 在 changes/<name>/specs/ 下维护能力规格变更

---

**相关文档**: [Superpowers 工作原理](../superpower/README.md)
