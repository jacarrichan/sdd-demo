# OpenSpec Skill 源码分析

本文档分析 OpenSpec 相关的所有 Skill 源码，包括 `comet-open`、`comet-archive` 以及相关的 shell 脚本。

---

## 1. OpenSpec 架构总览

```
┌─────────────────────────────────────────────────────────────────────┐
│                          OpenSpec 架构                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ┌──────────────┐        ┌──────────────┐        ┌──────────────┐ │
│   │   comet-open │───────▶│ comet-archive│◀───────│ comet-verify │ │
│   │   (Phase 1)  │        │  (Phase 5)   │        │  (Phase 4)   │ │
│   └──────────────┘        └──────────────┘        └──────────────┘ │
│          │                          ▲                           │   │
│          │                          │                           │   │
│          ▼                          │                           │   │
│   ┌──────────────┐                  │                           │   │
│   │   .comet.yaml│                  │                           │   │
│   │   状态管理   │──────────────────┘                           │   │
│   └──────────────┘                                              │   │
│          │                                                       │   │
│          ▼                                                       │   │
│   ┌─────────────────────────────────────────────────────────┐   │   │
│   │              Shell 脚本层 (OpenSpec Tools)               │   │   │
│   │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐       │   │   │
│   │  │comet-state  │  │comet-archive│  │comet-guard  │       │   │   │
│   │  │   .sh       │  │   .sh       │  │   .sh       │       │   │   │
│   │  └─────────────┘  └─────────────┘  └─────────────┘       │   │   │
│   └─────────────────────────────────────────────────────────┘   │   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. comet-open Skill 源码分析

### 2.1 文件位置
```
~/.claude/skills/comet-open/SKILL.md
```

### 2.2 核心功能
**Phase 1: 开启** - 通过 OpenSpec 探索想法、创建 change 结构

### 2.3 工作流程源码解析

```yaml
# SKILL.md 结构
---
name: comet-open
description: "Comet 阶段 1：开启。用 /comet-open 调用。通过 OpenSpec 探索想法、创建 change 结构"
---
```

### 2.4 执行步骤详解

#### Step 0: 入口状态验证 (Entry Check)

```bash
COMET_STATE="${COMET_STATE:-$(find . -path '*/comet/scripts/comet-state.sh' -type f -print -quit)}"
bash "$COMET_STATE" check <name> open
```

**源码逻辑**:
1. 动态查找 `comet-state.sh` 脚本位置
2. 调用 `check` 子命令验证入口条件
3. 验证项：
   - `.comet.yaml` 不存在（新 change）
   - `proposal.md` 非空
   - `design.md` 非空
   - `tasks.md` 非空

#### Step 1: 探索想法

```yaml
立即执行: 使用 Skill 工具加载 `openspec-explore` 技能
禁止跳过此步骤
```

**注意**: `openspec-explore` 是独立的 skill，未在当前仓库中找到源码。

#### Step 2: 创建 Change 结构

```yaml
立即执行: 使用 Skill 工具加载 `openspec-new-change` 技能
禁止跳过此步骤
```

**产出文件结构**:
```
openspec/changes/<name>/
├── .openspec.yaml      # Change 元数据
├── .comet.yaml         # Comet 状态追踪
├── proposal.md         # Why + What：问题、目标、范围
├── design.md           # How（高层）：架构决策、方案选型
└── tasks.md            # 任务清单（勾选框）
```

#### Step 3: 初始化 Comet 状态

```bash
bash "$COMET_STATE" init <name> full
```

**初始化逻辑** (来自 comet-state.sh):
```bash
case "$workflow" in
  full)
    phase="design"        # 完整流程从 design 开始
    build_mode="null"
    isolation="null"
    verify_mode="null"
    ;;
  hotfix|tweak)
    phase="build"         # 快速修复从 build 开始
    build_mode="direct"
    isolation="branch"
    verify_mode="light"
    ;;
esac
```

**生成的 .comet.yaml**:
```yaml
workflow: full
phase: design
design_doc: null
plan: null
build_mode: null
isolation: null
verify_mode: null
verify_result: pending
verified_at: null
archived: false
```

#### Step 4: 内容完整性检查

验证三个文档内容完整：
- **proposal.md**: 问题背景、目标、范围、非目标
- **design.md**: 高层架构决策、方案选型、数据流
- **tasks.md**: 任务列表，每个任务有明确描述

### 2.5 退出条件

```yaml
- proposal.md、design.md、tasks.md 均已创建且内容完整
- 阶段守卫: bash $COMET_GUARD <change-name> open
```

**guard 检查项** (来自 comet-guard.sh):
```bash
guard_open() {
  check "proposal.md exists and non-empty"
  check "design.md exists and non-empty"
  check "tasks.md exists and non-empty"
  check "tasks.md has at least one task"
}
```

### 2.6 自动流转

```yaml
退出后自动执行: comet-design skill
```

---

## 3. comet-archive Skill 源码分析

### 3.1 文件位置
```
~/.claude/skills/comet-archive/SKILL.md
```

### 3.2 核心功能
**Phase 5: 归档** - 同步 delta spec 到主 spec，归档 change

### 3.3 前置条件

```yaml
- 验证已通过（阶段 4 完成）
- 分支已处理
- openspec/changes/<name>/.comet.yaml 中 verify_result: pass
```

### 3.4 执行步骤详解

#### Step 0: 入口状态验证

```bash
bash "$COMET_STATE" check <name> archive
```

**检查项**:
- `phase: archive`
- `verify_result: pass`
- `archived: false`

#### Step 1: 执行归档

```bash
COMET_ARCHIVE="${COMET_ARCHIVE:-$(find . -path '*/comet/scripts/comet-archive.sh' -type f -print -quit)}"
bash "$COMET_ARCHIVE" "<change-name>"
```

**comet-archive.sh 脚本执行流程**:

##### 3.4.1 读取 .comet.yaml

```bash
yaml_field() {
  local field="$1"
  if [ -f "$STATE_SH" ]; then
    bash "$STATE_SH" get "$CHANGE" "$field" 2>/dev/null
  else
    grep "^${field}:" "$YAML" | sed "s/^${field}: *//" | tr -d '"' | tr -d "'"
  fi
}

DESIGN_DOC=$(yaml_field "design_doc")
PLAN_PATH=$(yaml_field "plan")
```

##### 3.4.2 入口状态验证

```bash
PHASE_VAL=$(yaml_field "phase")
VERIFY_VAL=$(yaml_field "verify_result")
ARCHIVED_VAL=$(yaml_field "archived")

if [ "$PHASE_VAL" != "archive" ]; then
  exit 1
fi
if [ "$VERIFY_VAL" != "pass" ]; then
  exit 1
fi
if [ "$ARCHIVED_VAL" = "true" ]; then
  exit 1
fi
```

##### 3.4.3 同步 Delta Spec → Main Spec

```bash
sync_delta_specs() {
  local delta_root="$CHANGE_DIR/specs"
  if [ ! -d "$delta_root" ]; then
    return 0
  fi

  for delta_spec_dir in "$delta_root"/*/; do
    capability=$(basename "$delta_spec_dir")
    delta_spec="$delta_spec_dir/spec.md"
    main_spec="openspec/specs/$capability/spec.md"

    if [ ! -f "$main_spec" ]; then
      mkdir -p "openspec/specs/$capability"
    fi
    cp "$delta_spec" "$main_spec"
  done
}
```

**核心逻辑**:
1. 遍历 `changes/<name>/specs/<capability>/spec.md`
2. 复制到 `openspec/specs/<capability>/spec.md`
3. 如果不存在则创建目录

##### 3.4.4 标注设计文档

```bash
annotate_frontmatter() {
  local file="$1"
  local extra_fields="$2"

  if head -1 "$file" | grep -q '^---'; then
    # 已有 frontmatter，添加 archived-with
    awk -v archive="$ARCHIVE_NAME" '
      /^archived-with:/ { next }
      NR==1 && /^---/ { print; next }
      /^---/ && NR>1 {
        print "archived-with: " archive
        print; next
      }
      { print }
    '
  else
    # 没有 frontmatter，创建新的
    {
      echo "---"
      echo "archived-with: $ARCHIVE_NAME"
      echo "status: final"
      echo "---"
      cat "$file"
    }
  fi
}
```

##### 3.4.5 移动 Change 到归档目录

```bash
TODAY=$(date +%Y-%m-%d)
ARCHIVE_NAME="${TODAY}-${CHANGE}"
ARCHIVE_DIR="openspec/changes/archive/${ARCHIVE_NAME}"

mkdir -p "openspec/changes/archive"
mv "$CHANGE_DIR" "$ARCHIVE_DIR"
```

##### 3.4.6 更新 archived: true

```bash
ARCHIVE_YAML="$ARCHIVE_DIR/.comet.yaml"
sed -i 's/^archived:.*/archived: true/' "$ARCHIVE_YAML"
```

### 3.5 生命周期闭环

```
brainstorming → delta spec → 实施 → 验证 → 主 spec 覆盖 → design doc 标注 → 归档
```

---

## 4. Shell 脚本层深度分析

### 4.1 comet-state.sh - 状态管理器

#### 4.1.1 整体架构

```bash
#!/bin/bash
# Comet State — unified interface for .comet.yaml state management

# 子命令分发
case "$SUBCOMMAND" in
  init)  cmd_init "$@" ;;
  get)   cmd_get "$@" ;;
  set)   cmd_set "$@" ;;
  check) cmd_check "$@" ;;
  scale) cmd_scale "$@" ;;
esac
```

#### 4.1.2 cmd_init - 初始化

```bash
cmd_init() {
  local change_name="$1"
  local workflow="$2"

  # 验证输入
  validate_change_name "$change_name"
  validate_enum "$workflow" "full" "hotfix" "tweak"

  # 检查是否已存在
  if [ -f "$yaml_file" ]; then
    exit 1
  fi

  # 根据 workflow 设置默认值
  case "$workflow" in
    full)
      phase="design"
      build_mode="null"
      isolation="null"
      verify_mode="null"
      ;;
    hotfix|tweak)
      phase="build"
      build_mode="direct"
      isolation="branch"
      verify_mode="light"
      ;;
  esac

  # 写入文件
  cat > "$yaml_file" <<EOF
workflow: $workflow
phase: $phase
build_mode: $build_mode
isolation: $isolation
verify_mode: $verify_mode
design_doc: null
plan: null
verify_result: pending
verified_at: null
archived: false
EOF
}
```

#### 4.1.3 cmd_get / cmd_set - 读写字段

```bash
# 读取字段
yaml_field() {
  local field="$1"
  local yaml_file="$2"
  grep "^${field}:" "$yaml_file" | sed "s/^${field}: *//" | tr -d '"' | tr -d "'"
}

# 设置字段
cmd_set() {
  if grep -q "^${field}:" "$yaml_file"; then
    # 字段存在，替换
    sed -i "s/^${field}:.*/${field}: ${value}/" "$yaml_file"
  else
    # 字段不存在，追加
    echo "${field}: ${value}" >> "$yaml_file"
  fi
}
```

#### 4.1.4 cmd_check - 阶段入口验证

```bash
cmd_check() {
  local phase="$1"
  local change_name="$2"

  case "$phase" in
    open)
      check_file_not_exists ".comet.yaml" "$yaml_file"
      check_nonempty "proposal.md" "$proposal_file"
      check_nonempty "design.md" "$design_file"
      check_nonempty "tasks.md" "$tasks_file"
      ;;
    design)
      check_yaml_is "phase" "design"
      check_yaml_is "workflow" "full"
      check_yaml_empty "design_doc"
      ;;
    build)
      check_yaml_is "phase" "build"
      check_design_doc_exists
      ;;
    verify)
      check_yaml_is "phase" "verify"
      check_verify_result_pending
      ;;
    archive)
      check_yaml_is "phase" "archive"
      check_yaml_is "verify_result" "pass"
      check_archived_not_true
      ;;
  esac
}
```

#### 4.1.5 cmd_scale - 规模评估

```bash
cmd_scale() {
  # 读取指标
  # 1. 任务数
  task_count=$(grep -c '^\- \[' "$tasks_file")

  # 2. Delta spec 数
  delta_spec_count=$(find "$change_dir/specs" -name "spec.md" | wc -l)

  # 3. 变更文件数
  changed_files=$(git diff --stat HEAD | tail -1 | grep -o '[0-9]\+ file')

  # 决策规则
  if [ "$task_count" -gt 3 ] || [ "$delta_spec_count" -gt 1 ] || [ "$changed_files" -gt 5 ]; then
    result="full"
  else
    result="light"
  fi

  # 更新 verify_mode
  sed -i "s/^verify_mode:.*/verify_mode: $result/" "$yaml_file"
}
```

---

### 4.2 comet-guard.sh - 阶段守卫

#### 4.2.1 核心机制

```bash
# 阶段检查映射
case "$PHASE" in
  open)     preflight "design";  guard_open ;;
  design)   preflight "build";   guard_design ;;
  build)    preflight "verify";  guard_build ;;
  verify)   preflight "archive"; guard_verify ;;
  archive)  preflight "archive"; guard_archive ;;
esac
```

#### 4.2.2 preflight - 预检

```bash
preflight() {
  local expected_phase="$1"

  # 检查目录存在
  if [ ! -d "$CHANGE_DIR" ]; then
    exit 1
  fi

  # 检查 .comet.yaml 存在
  if [ ! -f "$CHANGE_DIR/.comet.yaml" ]; then
    exit 1
  fi

  # 验证 phase 字段
  actual_phase=$(yaml_field_value "phase")
  if [ "$actual_phase" != "$expected_phase" ]; then
    exit 1
  fi

  # Schema 验证
  bash "$validate_script" "$CHANGE"
}
```

#### 4.2.3 各阶段守卫检查

```bash
guard_open() {
  check "proposal.md exists and non-empty"
  check "design.md exists and non-empty"
  check "tasks.md exists and non-empty"
  check "tasks.md has at least one task"
}

guard_design() {
  check "proposal.md exists"
  check "tasks.md exists"
  check "Design Doc exists"  # 如果有 design_doc 字段
}

guard_build() {
  check "tasks.md all tasks checked"
  check "Maven compile passes"  # 或对应语言的构建命令
}

guard_verify() {
  check "verify_result is pass"
  check "tasks.md all tasks checked"
}

guard_archive() {
  check "archived is true"
  check "tasks.md all tasks checked"
}
```

#### 4.2.4 自动状态更新 (--apply)

```bash
apply_state_update() {
  local p="$1"

  case "$p" in
    open)
      bash "$state_sh" set "$CHANGE" phase design
      ;;
    design)
      bash "$state_sh" set "$CHANGE" phase build
      ;;
    build)
      bash "$state_sh" set "$CHANGE" phase verify
      bash "$state_sh" set "$CHANGE" verify_result pending
      ;;
    verify)
      bash "$state_sh" set "$CHANGE" phase archive
      bash "$state_sh" set "$CHANGE" verify_result pass
      bash "$state_sh" set "$CHANGE" verified_at "$(date +%Y-%m-%d)"
      ;;
  esac
}
```

---

### 4.3 comet-yaml-validate.sh - 配置验证器

#### 4.3.1 验证流程

```bash
validate_change_name "$1"

CHANGE="$1"
YAML="openspec/changes/$CHANGE/.comet.yaml"

ERRORS=0
WARNINGS=0
```

#### 4.3.2 必需字段检查

```bash
REQUIRED_FIELDS="workflow phase design_doc plan build_mode isolation verify_mode verify_result verified_at archived"

for field in $REQUIRED_FIELDS; do
  if ! grep -q "^${field}:" "$YAML"; then
    fail "missing required field '$field'"
  fi
done
```

#### 4.3.3 枚举值验证

```bash
validate_enum() {
  local field="$1" value="$2"
  shift 2
  local valid_values="$*"

  # null 或空值始终接受
  if [ -z "$value" ] || [ "$value" = "null" ]; then
    return 0
  fi

  for v in $valid_values; do
    if [ "$value" = "$v" ]; then
      return 0
    fi
  done
  fail "$field='$value' is not valid"
}

# 验证各字段
validate_enum "workflow"      "$workflow"      "full hotfix tweak"
validate_enum "phase"         "$phase"          "design build verify archive"
validate_enum "build_mode"    "$build_mode"     "subagent-driven-development executing-plans direct"
validate_enum "isolation"     "$isolation"      "branch worktree"
validate_enum "verify_mode"   "$verify_mode"    "light full"
validate_enum "verify_result" "$verify_result"  "pending pass fail"
validate_enum "archived"      "$archived"       "true false"
```

#### 4.3.4 路径验证

```bash
if [ -n "$design_doc" ] && [ "$design_doc" != "null" ]; then
  if [ ! -f "$design_doc" ]; then
    fail "design_doc='$design_doc' does not exist"
  fi
fi

if [ -n "$plan" ] && [ "$plan" != "null" ]; then
  if [ ! -f "$plan" ]; then
    fail "plan='$plan' does not exist"
  fi
fi
```

---

## 5. OpenSpec 数据流分析

### 5.1 状态流转图

```
┌────────────────────────────────────────────────────────────────────────┐
│                         OpenSpec 状态流转                              │
├────────────────────────────────────────────────────────────────────────┤
│                                                                        │
│   ┌─────────┐                                                          │
│   │  Init   │                                                          │
│   │ (null)  │                                                          │
│   └────┬────┘                                                          │
│        │                                                               │
│        ▼                                                               │
│   ┌─────────┐     comet-state init      ┌─────────┐                   │
│   │  Open   │ ────────────────────────▶│  Phase  │                   │
│   │ (文件)  │                          │  design │                   │
│   └────┬────┘                          │(hotfix: │                   │
│        │                               │ build)  │                   │
│        │                               └────┬────┘                   │
│        │                                     │                         │
│        │        ┌────────────────────────────┘                         │
│        │        │                                                      │
│        │        ▼                                                      │
│        │   ┌─────────┐    comet-guard design --apply                  │
│        │   │ Design  │ ───────────────────────────────────▶           │
│        │   │(yaml)   │                                               │
│        │   └────┬────┘                                               │
│        │        │                                                     │
│        │        ▼                                                     │
│        │   ┌─────────┐    comet-guard build --apply                   │
│        │   │  Build  │ ───────────────────────────────────▶           │
│        │   │(yaml)   │                                               │
│        │   └────┬────┘                                               │
│        │        │                                                     │
│        │        ▼                                                     │
│        │   ┌─────────┐    comet-guard verify --apply                  │
│        │   │ Verify  │ ───────────────────────────────────▶           │
│        │   │(yaml)   │    verify_result: pass                         │
│        │   └────┬────┘    verified_at: YYYY-MM-DD                    │
│        │        │                                                     │
│        │        ▼                                                     │
│        │   ┌─────────┐    comet-archive.sh                            │
│        │   │ Archive │ ───────────────────────────────────▶           │
│        │   │(yaml)   │    archived: true                                │
│        │   └────┬────┘    移动到 archive/                             │
│        │        │                                                     │
│        │        ▼                                                     │
│        │   ┌─────────┐                                                │
│        └──▶│  Done   │                                                │
│            │(归档)   │                                                │
│            └─────────┘                                                │
│                                                                        │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 6. 关键设计模式

### 6.1 状态机模式

```yaml
# .comet.yaml 是一个有限状态机
phase: design → build → verify → archive

# 状态转换由 guard 控制
# 每个状态有明确的进入/退出条件
```

### 6.2 命令模式

```bash
# comet-state.sh 使用命令模式
bash comet-state.sh <command> <args>

commands:
  - init: 创建状态
  - get: 读取字段
  - set: 更新字段
  - check: 验证入口
  - scale: 评估规模
```

### 6.3 策略模式

```yaml
# workflow 字段决定策略
workflow: full     # 完整流程 (5阶段)
workflow: hotfix   # 热修复 (跳过 brainstorming)
workflow: tweak    # 小调整 (跳过 brainstorming + plan)
```

### 6.4 守卫模式

```bash
# comet-guard.sh 实现守卫模式
# 阶段转换必须通过守卫检查
bash comet-guard.sh <change> <phase> [--apply]

# --apply 自动应用状态更新
```

---

## 7. OpenSpec vs Superpowers 接口

### 7.1 OpenSpec 产出

| 文件 | 内容 | Superpowers 使用 |
|------|------|------------------|
| proposal.md | Why + What | Design Doc 的需求来源 |
| design.md (高层) | 架构决策 | Design Doc 的设计基础 |
| .comet.yaml | 状态追踪 | 关联 Design Doc 和 Plan |

### 7.2 协作流程

```
OpenSpec                    Superpowers
────────────────────────────────────────────────
proposal.md ──────────────▶ Design Doc
                             (需求输入)

design.md ───────────────▶ Design Doc
                             (设计细化)

.comet.yaml ─────────────▶ Plan 文件头
  design_doc: <path>         (change: <name>)
  plan: <path>               (design-doc: <path>)
```

---

## 8. 扩展点分析

### 8.1 新增 Workflow 类型

在 `comet-state.sh` 的 `cmd_init` 中添加：

```bash
case "$workflow" in
  full|hotfix|tweak)
    # 现有逻辑
    ;;
  new-workflow)
    phase="custom"
    build_mode="..."
    ;;
esac
```

### 8.2 新增 Phase

在 `comet-guard.sh` 中添加：

```bash
case "$PHASE" in
  # 现有阶段
  new-phase)
    preflight "next-phase"
    guard_new_phase
    ;;
esac
```

### 8.3 新增验证规则

在 `comet-yaml-validate.sh` 中添加：

```bash
# 新增字段
NEW_FIELDS="new_field"
REQUIRED_FIELDS="... $NEW_FIELDS"

# 新增枚举
validate_enum "new_field" "$new_field" "value1 value2"
```

---

## 9. 总结

### 9.1 OpenSpec 核心职责

1. **Change 生命周期管理** - 从创建到归档的完整流程
2. **状态追踪** - .comet.yaml 作为单一事实来源
3. **准入准出控制** - guard 脚本确保质量门槛
4. **规格同步** - delta spec 归档时合并到 main spec

### 9.2 关键源码文件

| 文件 | 职责 | 核心函数 |
|------|------|----------|
| comet-state.sh | 状态管理 | init, get, set, check, scale |
| comet-guard.sh | 阶段守卫 | guard_open, guard_design, guard_build, guard_verify, guard_archive |
| comet-archive.sh | 归档流程 | sync_delta_specs, annotate_frontmatter |
| comet-yaml-validate.sh | 配置验证 | validate_enum, validate_path |

### 9.3 设计理念

- **显式优于隐式** - 所有状态变更必须通过脚本
- **守卫模式** - 阶段转换必须通过验证
- **可扩展** - 通过 workflow 和枚举值支持扩展
- **双轨归档** - OpenSpec 归档到 archive/，Superpowers 标注 archived-with

---

**生成日期**: 2026-06-28  
**分析版本**: v1.0.0
