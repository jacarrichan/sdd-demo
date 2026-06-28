# Superpowers 工作原理详解

Superpowers 是 Comet 双星开发流程中的 **HOW** 系统，负责技术设计、计划制定和执行实现。

---

## 核心概念

Superpowers 关注 **"怎么做"** 和 **"技术实现细节"**：
- 技术选型与架构决策
- 详细设计方案（RFC）
- 实施计划与任务分解
- 代码执行与构建

---

## 工作流程

```
┌─────────────────────────────────────────────────────────────────┐
│                    Superpowers 工作流                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Phase 2: Design                                                │
│  ├─ 创建 Design Doc (技术 RFC)                                 │
│  │   docs/superpowers/specs/YYYY-MM-DD-topic-design.md         │
│  ├─ 技术决策 (技术栈、架构、API)                                │
│  ├─ Delta Spec 细化                                             │
│  └─ 验证标准定义                                                │
│                                                                 │
│  ↓                                                              │
│                                                                 │
│  Phase 3: Build                                                 │
│  ├─ 创建 Plan (实施计划)                                        │
│  │   docs/superpowers/plans/YYYY-MM-DD-feature.md            │
│  ├─ 任务分解与策略选择                                          │
│  │   • subagent-driven-development                            │
│  │   • executing-plans                                         │
│  │   • direct                                                  │
│  └─ 代码实现与提交                                              │
│                                                                 │
│  ↓                                                              │
│                                                                 │
│  Phase 4: Verify                                                │
│  ├─ 功能验证                                                    │
│  ├─ 测试执行                                                    │
│  └─ 分支处理决策                                                │
│                                                                 │
│  ↓                                                              │
│                                                                 │
│  Phase 5: Archive                                               │
│  ├─ Design Doc 标注 archived-with                             │
│  └─ Plan 标注 archived-with                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 核心文件说明

### 1. Design Doc - 技术设计文档

**位置**: `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`

**作用**: 技术 RFC (Request for Comments)，详细技术设计文档

**文件头元数据**:
```yaml
---
关联 Change: spring-boot-hello-world
创建日期: 2026-06-28
状态: design → build → verify → archive
---
```

**核心内容**:

| 章节 | 内容 |
|------|------|
| 技术决策 | 框架、语言、构建工具选型及理由 |
| 项目结构 | 目录结构、模块划分 |
| 依赖清单 | 完整依赖表（名称、版本、用途） |
| API 规格 | 端点、方法、参数、响应格式 |
| 配置 | 应用配置、环境变量 |
| 验证标准 | 验收条件、测试要求 |

**示例**:
```markdown
# Spring Boot Hello World 技术设计

**关联 Change**: spring-boot-hello-world

## 1. 技术决策

### Spring Boot 版本
- **选择**: Spring Boot 3.3.0
- **理由**: 最新稳定版本，支持 Java 17+

## 2. 项目结构
```
src/
├── main/java/...
└── resources/
```

## 3. 依赖清单

| 依赖 | 版本 | 用途 |
|------|------|------|
| spring-boot-starter-web | 3.3.0 | Web 支持 |

## 4. API 规格

### GET /hello
- **描述**: 返回问候语
- **响应**: `text/plain` - "Hello World"

## 5. 验证标准
1. 项目可成功构建
2. 应用可成功启动
3. GET /hello 返回正确响应
```

---

### 2. Plan - 实施计划

**位置**: `docs/superpowers/plans/YYYY-MM-DD-<feature>.md`

**作用**: 具体实施计划，任务分解和执行策略

**文件头元数据**:
```yaml
---
change: spring-boot-hello-world
design-doc: docs/superpowers/specs/2026-06-28-spring-boot-hello-design.md
创建日期: 2026-06-28
---
```

**核心内容**:

| 章节 | 内容 |
|------|------|
| 任务列表 | 编号、描述、验收标准、文件 |
| 执行策略 | subagent-driven / executing-plans / direct |
| 验证计划 | 测试步骤、预期结果 |

**任务格式**:
```markdown
### Task 1: 创建 Maven 配置
**描述**: 创建 pom.xml，配置 Spring Boot 父 POM
**验收**: pom.xml 存在且格式正确
**文件**: `pom.xml`

### Task 2: 创建项目结构
**描述**: 创建 Java 包目录结构
**验收**: 目录存在
**命令**: `mkdir -p src/main/java/...`
```

**执行策略**:

| 策略 | 适用场景 | 说明 |
|------|----------|------|
| `subagent-driven-development` | 复杂任务、多文件修改 | 使用子 Agent 并行执行 |
| `executing-plans` | 有详细计划的多步骤任务 | 按计划逐步执行 |
| `direct` | 简单任务、单文件修改 | 直接修改，无需复杂协调 |

---

## 关键脚本解析

### comet-guard.sh - 阶段守卫

**功能**: 验证阶段退出条件，确保质量门槛

**工作原理**:

```
┌────────────────────────────────────────────────────────────┐
│                    Guard 检查流程                         │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  1. Preflight 检查                                        │
│     └─ 验证 change 目录存在                                │
│     └─ 验证 .comet.yaml 存在                               │
│     └─ 验证 phase 字段匹配预期                             │
│     └─ 运行 YAML schema 验证                               │
│                                                            │
│  2. Phase-specific 检查                                    │
│     ├─ guard_open: proposal/design/tasks.md 存在           │
│     ├─ guard_design: Design Doc 存在                       │
│     ├─ guard_build: 所有任务完成、Maven 编译通过           │
│     ├─ guard_verify: verify_result=pass                   │
│     └─ guard_archive: archived=true                       │
│                                                            │
│  3. 结果处理                                               │
│     ├─ 检查通过 → 允许进入下一阶段                         │
│     │              (可选: --apply 自动更新状态)             │
│     └─ 检查失败 → 阻断，输出失败原因                         │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

**检查函数**:

```bash
# 检查所有任务是否完成
tasks_all_done() {
  local tasks="$CHANGE_DIR/tasks.md"
  [ -f "$tasks" ] || return 1
  grep -q '\- \[x\]' "$tasks" || return 1    # 有完成的任务
  ! grep -q '\- \[ \]' "$tasks"             # 没有未完成的任务
}

# 检查 Maven 编译
maven_compiles() {
  mvn compile -q 2>/dev/null
}

# 检查验证结果
verify_result_is_pass() {
  local result=$(yaml_field_value "verify_result")
  [ "$result" = "pass" ]
}
```

**使用方式**:

```bash
# 检查当前阶段是否可以退出
bash comet-guard.sh <change-name> <current-phase>

# 检查通过并自动更新状态
bash comet-guard.sh <change-name> <current-phase> --apply
```

---

### comet-yaml-validate.sh - 配置验证器

**功能**: 验证 `.comet.yaml` 格式和字段值

**验证内容**:

1. **必需字段检查**:
   - workflow, phase, design_doc, plan
   - build_mode, isolation, verify_mode
   - verify_result, verified_at, archived

2. **枚举值验证**:
   - `workflow`: full / hotfix / tweak
   - `phase`: design / build / verify / archive
   - `build_mode`: subagent-driven-development / executing-plans / direct
   - `isolation`: branch / worktree
   - `verify_mode`: light / full
   - `verify_result`: pending / pass / fail
   - `archived`: true / false

3. **路径验证**:
   - `design_doc` 路径对应的文件是否存在
   - `plan` 路径对应的文件是否存在

4. **未知字段警告**:
   - 检测不在白名单中的字段

**使用方式**:

```bash
bash comet-yaml-validate.sh <change-name>
# 退出码 0 = 验证通过
# 退出码 1 = 验证失败
```

---

## 文件结构总览

```
docs/superpowers/
│
├── specs/                              # 技术设计文档
│   └── YYYY-MM-DD-<topic>-design.md
│       ├── 关联 Change 元数据
│       ├── 技术决策
│       ├── 项目结构
│       ├── 依赖清单
│       ├── API 规格
│       ├── 配置说明
│       └── 验证标准
│
└── plans/                              # 实施计划
    └── YYYY-MM-DD-<feature>.md
        ├── change: <name>              # 关联 change
        ├── design-doc: <path>          # 关联设计文档
        ├── 任务列表
        │   ├── Task N: 描述/验收/文件
        ├── 执行策略
        └── 验证计划
```

---

## 与其他组件的关系

```
OpenSpec (WHAT)                    Superpowers (HOW)
───────────────────────────────────────────────────────────

openspec/changes/<name>/
├── proposal.md ───────┐
├── design.md (高层) ────┤         docs/superpowers/
└── .openspec.yaml       │         ├── specs/
                         │         │   └── <date>-topic-design.md ◄──────┐
openspec/changes/<name>/ │         │        ▲                            │
└── .comet.yaml          │         │        │ 引用                        │
    ├── design_doc ──────┘         │        │                           │
    │  = specs/YYYY-MM-DD-...      │        │                           │
    │                              │        │                           │
    └── plan                       │   docs/superpowers/plans/          │
       = plans/YYYY-MM-DD-... ──────┘   └── <date>-feature.md ────────────┘
              ▲                              ▲
              │                              │
              │ 文件头元数据:                  │ 文件头元数据:
              │   change: <name>             │   change: <name>
              │   design-doc: <path>         │   design-doc: <path>
```

**依赖关系**:
1. **Design Doc** → 引用 OpenSpec 的 `proposal.md` 和 `design.md`
2. **Plan** → 引用 Design Doc（文件头 `design-doc` 字段）
3. **Plan** → 关联 OpenSpec Change（文件头 `change` 字段）
4. **.comet.yaml** → 记录 Design Doc 和 Plan 的路径

---

## 使用场景

### 场景 1: 创建技术设计

```bash
# 1. 创建设计文档
cat > docs/superpowers/specs/2026-06-28-my-feature-design.md << 'EOF'
---
关联 Change: my-feature
创建日期: 2026-06-28
---

# My Feature 技术设计

## 1. 技术决策
...
EOF

# 2. 更新 .comet.yaml
bash comet-state.sh set my-feature design_doc \
  docs/superpowers/specs/2026-06-28-my-feature-design.md
```

### 场景 2: 创建实施计划

```bash
# 1. 创建计划文档
cat > docs/superpowers/plans/2026-06-28-my-feature.md << 'EOF'
---
change: my-feature
design-doc: docs/superpowers/specs/2026-06-28-my-feature-design.md
---

# My Feature 实施计划

## 任务列表
...
EOF

# 2. 更新 .comet.yaml
bash comet-state.sh set my-feature plan \
  docs/superpowers/plans/2026-06-28-my-feature.md
```

### 场景 3: 阶段守卫检查

```bash
# 检查是否可以进入下一阶段
bash comet-guard.sh my-feature build

# 输出示例:
# === Guard: build → verify ===
#   [PASS] tasks.md all tasks checked
#   [PASS] proposal.md exists
#   [PASS] Maven compile passes
#
# ALL CHECKS PASSED — ready for next phase
```

---

## 最佳实践

1. **Design Doc 要完整** - 包含技术决策、API 规格、验证标准
2. **Plan 要可执行** - 任务描述清晰，验收标准明确
3. **关联元数据** - Design Doc 和 Plan 必须包含 change 和 design-doc 字段
4. **频繁更新** - Design Doc 是活文档，阶段 3 可修改
5. **归档标注** - 完成时标注 `archived-with` 状态
6. **执行策略选择**:
   - 简单任务 → `direct`
   - 多步骤 → `executing-plans`
   - 复杂/并行 → `subagent-driven-development`

---

## 与 OpenSpec 的协作

| OpenSpec | Superpowers | 协作点 |
|----------|-------------|--------|
| proposal.md | Design Doc | 需求输入 |
| design.md (高层) | Design Doc | 设计细化 |
| .comet.yaml | Design Doc + Plan | 状态关联 |
| tasks.md | Plan | 任务分解 |

---

**相关文档**: [OpenSpec 工作原理](../openspec/README.md)
