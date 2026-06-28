# Superpowers Skill 源码分析

本文档分析 Superpowers 相关的 Skill 源码，专注于技术设计、计划制定和执行实现。

---

## 1. Superpowers 架构总览

```
┌─────────────────────────────────────────────────────────────────────┐
│                      Superpowers 系统架构                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ┌─────────────────────────────────────────────────────────────┐ │
│   │                      设计层 (Design)                         │ │
│   │                                                              │ │
│   │  ┌─────────────────────────────────────────────────────────┐│ │
│   │  │                brainstorming                             ││ │
│   │  │  - 探索项目上下文                                        ││ │
│   │  │  - 询问澄清问题                                          ││ │
│   │  │  - 提出 2-3 种方案                                       ││ │
│   │  │  - 产出 Design Doc                                       ││ │
│   │  └─────────────────────────────────────────────────────────┘│ │
│   │                                                              │ │
│   └─────────────────────────────────────────────────────────────┘ │
│                              │                                      │
│                              ▼                                      │
│   ┌─────────────────────────────────────────────────────────────┐ │
│   │                      计划层 (Plan)                           │ │
│   │                                                              │ │
│   │  ┌─────────────────────────────────────────────────────────┐│ │
│   │  │                writing-plans                           ││ │
│   │  │  - 将设计拆分为任务                                      ││ │
│   │  │  - 定义文件和接口                                        ││ │
│   │  │  - 指定验证步骤                                          ││ │
│   │  └─────────────────────────────────────────────────────────┘│ │
│   │                                                              │ │
│   └─────────────────────────────────────────────────────────────┘ │
│                              │                                      │
│                              ▼                                      │
│   ┌─────────────────────────────────────────────────────────────┐ │
│   │                    执行层 (Execute)                          │ │
│   │                                                              │ │
│   │  ┌────────────────┐    ┌────────────────┐                  │ │
│   │  │ subagent-driven│    │ executing-plans│                  │ │
│   │  │ -development   │    │                │                  │ │
│   │  │                │    │                │                  │ │
│   │  │ 子代理 per 任务 │    │ 批量执行       │                  │ │
│   │  │ 任务审查        │    │ 检查点审查     │                  │ │
│   │  │ 最终审查        │    │                │                  │ │
│   │  └────────────────┘    └────────────────┘                  │ │
│   │                                                              │ │
│   └─────────────────────────────────────────────────────────────┘ │
│                              │                                      │
│                              ▼                                      │
│   ┌─────────────────────────────────────────────────────────────┐ │
│   │                    收尾层 (Finish)                         │ │
│   │                                                              │ │
│   │  ┌─────────────────────────────────────────────────────────┐│ │
│   │  │         finishing-a-development-branch                   ││ │
│   │  │  - 验证测试                                              ││ │
│   │  │  - 呈现选项（合并/PR/保留/丢弃）                         ││ │
│   │  │  - 执行选择                                              ││ │
│   │  └─────────────────────────────────────────────────────────┘│ │
│   │                                                              │ │
│   └─────────────────────────────────────────────────────────────┘ │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Superpowers 核心概念

### 2.1 什么是 Superpowers

Superpowers 是一个**技术实现系统**，关注 **HOW**（怎么做、技术细节）：
- 技术选型与架构决策
- 详细设计方案（RFC）
- 实施计划与任务分解
- 代码执行与构建

### 2.2 核心文件结构

```
docs/superpowers/
├── specs/                          # 技术设计文档
│   └── YYYY-MM-DD-<topic>-design.md
└── plans/                          # 实施计划
    └── YYYY-MM-DD-<feature>.md
```

### 2.3 与 OpenSpec 的区别

| 维度 | Superpowers | OpenSpec |
|------|-------------|----------|
| **关注点** | HOW（怎么做） | WHAT（做什么） |
| **核心问题** | 技术实现细节 | 为什么做、做什么 |
| **主要产出** | Design Doc, Plan, Code | proposal, design (高层) |
| **目标读者** | 开发者 | PM、架构师 |

---

## 3. brainstorming Skill

### 3.1 功能定位
**将想法转化为完整设计** - 通过协作对话产出技术设计文档。

### 3.2 核心原则

```yaml
HARD-GATE（硬约束）:
  在呈现设计并获得用户批准之前：
    - 禁止调用任何实现技能
    - 禁止编写任何代码
    - 禁止搭建任何项目
    - 禁止任何实现操作

  适用场景：
    - 每个项目，无论感知复杂度
    - todo list、单功能工具、配置变更
```

### 3.3 工作流程

```
┌─────────────────────────────────────────────────────────────────┐
│                    Brainstorming 流程                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. Explore project context                                     │
│     ├─ 检查文件                                                 │
│     ├─ 阅读文档                                                 │
│     └─ 了解技术栈                                               │
│                                                                 │
│  2. Ask clarifying questions                                    │
│     ├─ 一次一个问题                                             │
│     ├─ 选择题优先                                               │
│     └─ 聚焦：目的、约束、成功标准                                 │
│                                                                 │
│  3. Propose 2-3 approaches                                      │
│     ├─ 不同方案                                                 │
│     ├─ 权衡分析                                                 │
│     └─ 推荐理由                                                 │
│                                                                 │
│  4. Present design sections                                     │
│     ├─ 按复杂度缩放                                             │
│     ├─ 每部分后确认                                             │
│     └─ 涵盖：架构、组件、数据流、错误处理、测试                   │
│                                                                 │
│  5. Write design doc                                            │
│     ├─ 保存到 docs/superpowers/specs/                           │
│     ├─ 提交到 git                                               │
│     └─ 自我审查                                                 │
│                                                                 │
│  6. User reviews written spec                                 │
│     ├─ 请求用户审查                                             │
│     ├─ 如有变更，修改                                           │
│     └─ 获得最终批准                                             │
│                                                                 │
│  7. Transition to implementation                                │
│     └─ 调用 writing-plans                                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 3.4 设计文档结构

```markdown
---
关联 Change: <name>
创建日期: YYYY-MM-DD
---

# <Topic> 设计文档

## 1. 技术决策

### 1.1 <决策点>
- **选择**: <选择>
- **理由**: <理由>

## 2. 项目结构

```
目录树...
```

## 3. 依赖清单

| 依赖 | 版本 | 用途 |

## 4. API 规格

### 4.1 <端点>
- **描述**: <描述>
- **请求**: <请求格式>
- **响应**: <响应格式>

## 5. 配置

## 6. 验证标准
```

### 3.5 自我审查清单

```yaml
Spec Self-Review:
  1. Placeholder scan:
     - Any "TBD", "TODO", incomplete sections?
     - Fix them.

  2. Internal consistency:
     - Do sections contradict each other?
     - Does architecture match features?

  3. Scope check:
     - Is this focused enough?
     - Does it need decomposition?

  4. Ambiguity check:
     - Could any requirement be interpreted two ways?
     - Make it explicit.
```

### 3.6 视觉伴侣 (Visual Companion)

```yaml
提供时机: just-in-time，非预先

何时提供:
  - 问题适合视觉展示时
  - mockups, wireframes, diagrams

何时不提供:
  - 概念性问题（用终端）
  - 权衡列表（用终端）

使用流程:
  1. "This next part might be easier if I show you..."
  2. 等待用户响应
  3. 如接受，启动 server 并打开浏览器
```

---

## 4. writing-plans Skill

### 4.1 功能定位
**将设计转化为可执行计划** - 创建详细的实施计划文档。

### 4.2 核心假设

```yaml
假设:
  - 工程师对代码库零了解
  - 工程师品味 questionable
  - 需要完整的上下文

必须提供:
  - 每个任务涉及的文件
  - 代码（如需）
  - 测试方法
  - 文档参考
  - 验证步骤
```

### 4.3 计划文档结构

```markdown
# [Feature] Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL:
> Use superpowers:subagent-driven-development (recommended)
> or superpowers:executing-plans

**Goal:** [One sentence]
**Architecture:** [2-3 sentences]
**Tech Stack:** [Key technologies]

## Global Constraints
[Project-wide requirements copied verbatim from spec]

---

### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

**Interfaces:**
- Consumes: [what this task uses]
- Produces: [what this task provides]

- [ ] **Step 1: Write the failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

- [ ] **Step 3: Write minimal implementation**

```python
def function(input):
    return expected
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
```

### 4.4 任务粒度

```yaml
每个步骤: 2-5 分钟
示例:
  - "Write the failing test"
  - "Run it to make sure it fails"
  - "Implement minimal code to pass"
  - "Run tests to verify"
  - "Commit"
```

### 4.5 禁止的占位符

```yaml
Plan Failures:
  ❌ "TBD", "TODO", "implement later"
  ❌ "Add appropriate error handling"
  ❌ "Write tests for the above" (without actual code)
  ❌ "Similar to Task N" (repeat the code)
  ❌ Steps without code blocks
  ❌ References to undefined types/functions
```

### 4.6 自我审查

```yaml
After Writing Plan:
  1. Spec coverage:
     - Can you point to a task for each spec requirement?
     - List any gaps.

  2. Placeholder scan:
     - Search for red flags.
     - Fix them.

  3. Type consistency:
     - Do signatures match across tasks?
     - `clearLayers()` vs `clearFullLayers()` is a bug.
```

### 4.7 执行方式选择

```
Plan complete and saved to `docs/superpowers/plans/<filename>.md`.

Two execution options:

1. Subagent-Driven (recommended)
   - Fresh subagent per task
   - Review between tasks
   - Fast iteration
   - REQUIRED SUB-SKILL: superpowers:subagent-driven-development

2. Inline Execution
   - Execute tasks in this session
   - Batch execution with checkpoints
   - REQUIRED SUB-SKILL: superpowers:executing-plans

Which approach?
```

---

## 5. subagent-driven-development Skill

### 5.1 功能定位
**通过子代理执行任务** - 每个任务使用独立子代理，带审查机制。

### 5.2 核心机制

```
核心公式:
  新鲜子代理 per 任务
  + 任务审查（规格符合性 + 代码质量）
  + 广泛最终审查
  = 高质量、快速迭代
```

### 5.3 执行流程

```
┌─────────────────────────────────────────────────────────────────┐
│              Subagent-Driven Development 流程                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. Read plan, create todos                                     │
│     └─ 创建所有任务的 todo 列表                                  │
│                                                                 │
│  2. Per-Task Loop                                               │
│     ┌────────────────────────────────────────────────────────┐  │
│     │                                                        │  │
│     │  a. Dispatch implementer subagent                     │  │
│     │     └─ 使用 implementer-prompt.md                     │  │
│     │                                                        │  │
│     │  b. Implementer asks questions?                     │  │
│     │     ├─ Yes: Answer, re-dispatch                      │  │
│     │     └─ No: Continue                                  │  │
│     │                                                        │  │
│     │  c. Implement, test, commit, self-review              │  │
│     │                                                        │  │
│     │  d. Write diff, dispatch task reviewer              │  │
│     │     └─ 使用 task-reviewer-prompt.md                   │  │
│     │                                                        │  │
│     │  e. Reviewer reports approved?                      │  │
│     │     ├─ No: Dispatch fix subagent                   │  │
│     │     │      └─ Re-review                            │  │
│     │     └─ Yes: Mark task complete                     │  │
│     │                                                        │  │
│     └────────────────────────────────────────────────────────┘  │
│                                                                 │
│  3. All Tasks Complete                                          │
│     └─ Dispatch final code reviewer                            │
│        └─ 使用 code-reviewer.md                                │
│                                                                 │
│  4. Use finishing-a-development-branch                         │
│     └─ 完成开发工作                                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 5.4 模型选择策略

| 任务类型 | 模型 | 说明 |
|----------|------|------|
| 机械实现 | 快速、便宜 | 独立函数、清晰规格、1-2 文件 |
| 集成和判断 | 标准 | 多文件协调、模式匹配、调试 |
| 架构和设计 | 最强可用 | 需要广泛代码库理解 |
| 审查 | 根据 diff 大小选择 | 小 diff 不需要最强模型 |

**关键规则**:
- 始终显式指定模型
- 遗漏模型会继承会话默认（通常最贵）
- Token 价格 < Turn 数量（便宜模型多次迭代可能更贵）

### 5.5 实现者状态处理

```yaml
DONE:
  操作: 生成审查包，分派任务审查器

DONE_WITH_CONCERNS:
  含义: 完成但有疑虑
  操作: 阅读疑虑，处理或记录

NEEDS_CONTEXT:
  含义: 需要更多信息
  操作: 提供上下文，重新分派

BLOCKED:
  含义: 无法完成任务
  处理:
    1. 上下文问题 → 提供更多上下文
    2. 需要更多推理 → 使用更强模型
    3. 任务太大 → 拆分任务
    4. 计划错误 → 上报人类
```

### 5.6 审查者处理

```yaml
审查者输入:
  - 简报文件 (task-N-brief.md)
  - 报告文件 (task-N-report.md)
  - 审查包 (git diff)
  - 全局约束

⚠️ Cannot verify from diff:
  - 不阻塞当前审查
  - 必须在标记任务完成前解决
  - 如确认是缺口，视为审查失败

发现 Critical/Important:
  - 分派修复子代理
  - 重新审查
  - 重复直到通过

发现 Minor:
  - 记录到进度台账
  - 最终审查时处理
```

### 5.7 文件交接机制

```yaml
Task Brief:
  命令: scripts/task-brief PLAN_FILE N
  输出: 唯一命名文件（如 task-3-brief.md）
  作用: 任务需求的单一来源

Report File:
  命名: task-N-report.md
  内容: 实现者写入完整报告
  返回: 状态、提交、测试摘要、疑虑

Reviewer Inputs:
  1. 简报文件（相同）
  2. 报告文件
  3. 审查包（diff）

Fix Report:
  追加到: 相同报告文件
  返回: 简短摘要
```

### 5.8 持久化进度

```yaml
Progress Ledger:
  位置: .superpowers/sdd/progress.md

用途:
  - 会话压缩后恢复
  - 避免重新分派已完成任务

格式:
  Task N: complete (commits <base7>..<head7>, review clean)

检查:
  启动时读取
  完成的任务标记为 DONE
  从第一个未完成任务恢复
```

---

## 6. executing-plans Skill

### 6.1 功能定位
**在当前会话执行计划** - 适用于任务紧密耦合或无法使用子代理的场景。

### 6.2 与 subagent-driven-development 的区别

| 特性 | subagent-driven-development | executing-plans |
|------|---------------------------|-----------------|
| 会话 | 同一会话 | 并行会话 |
| 子代理 | 每个任务新鲜子代理 | 批量执行 |
| 审查 | 每个任务后审查 | 检查点审查 |
| 速度 | 更快迭代 | 需要会话切换 |
| 适用 | 任务独立 | 任务紧密耦合 |

### 6.3 执行流程

```yaml
Step 1: Load and Review Plan
  - 读取计划文件
  - 批判性审查
  - 如有疑虑，提出

Step 2: Execute Tasks
  For each task:
    - Mark as in_progress
    - Follow each step exactly
    - Run verifications
    - Mark as completed

Step 3: Complete Development
  - 使用 finishing-a-development-branch
  - 验证测试
  - 呈现选项
  - 执行选择

Stop Conditions:
  - 遇到 blocker
  - 计划有关键缺口
  - 不理解指令
  - 验证反复失败
```

---

## 7. finishing-a-development-branch Skill

### 7.1 功能定位
**完成开发工作** - 验证测试、呈现选项、执行分支处理。

### 7.2 核心流程

```
Step 1: Verify Tests
  ├─ 运行项目测试套件
  ├─ 如失败，停止
  └─ 如通过，继续

Step 2: Detect Environment
  ├─ 确定 workspace 状态
  ├─ GIT_DIR vs GIT_COMMON
  └─ 确定菜单类型

Step 3: Determine Base Branch
  ├─ git merge-base HEAD main
  └─ 或询问用户

Step 4: Present Options
  ├─ 普通仓库/命名分支: 4 个选项
  └─ 分离 HEAD: 3 个选项

Step 5: Execute Choice
  ├─ Option 1: 本地合并
  ├─ Option 2: 推送并创建 PR
  ├─ Option 3: 保持现状
  └─ Option 4: 丢弃（需确认）

Step 6: Cleanup Workspace
  ├─ 仅选项 1 和 4 执行
  ├─ 检查 worktree 来源
  └─ 安全清理
```

### 7.3 选项矩阵

| 选项 | 合并 | 推送 | 保留 Worktree | 清理分支 |
|------|------|------|---------------|----------|
| 1. 本地合并 | ✓ | - | - | ✓ |
| 2. 创建 PR | - | ✓ | ✓ | - |
| 3. 保持现状 | - | - | ✓ | - |
| 4. 丢弃 | - | - | - | ✓ (强制) |

### 7.4 Worktree 清理

```yaml
清理流程:
  1. 先执行合并
  2. 确认成功
  3. cd 到主仓库根目录
  4. git worktree remove <path>
  5. git worktree prune

来源检查:
  - .worktrees/ 或 worktrees/: 安全清理
  - 其他: 环境拥有，不清理

注意事项:
  - 从 worktree 内部运行 remove 会失败
  - 必须先 cd 到主仓库
```

---

## 8. 协作关系

### 8.1 Superpowers 内部协作

```
brainstorming ──▶ writing-plans ──▶ subagent-driven-development
                       │                      │
                       ▼                      ▼
              Design Doc + Plan      finishing-a-development-branch
                                              │
                                              ▼
                                         Done

或:

brainstorming ──▶ writing-plans ──▶ executing-plans ──▶ finishing-a-development-branch
```

### 8.2 与 OpenSpec 的协作

```yaml
OpenSpec 提供输入:
  - proposal.md → Design Doc 的需求来源
  - design.md → Design Doc 的设计基础
  - tasks.md → Plan 的任务来源

Superpowers 提供输出:
  - Design Doc → 由 OpenSpec 归档
  - Plan → 由 OpenSpec 归档
  - Code → 实现成果
```

### 8.3 元数据关联

```yaml
# Superpowers Plan 文件头
---
change: <openspec-change-name>
design-doc: docs/superpowers/specs/YYYY-MM-DD-topic-design.md
---
```

---

## 9. 设计原则

### 9.1 Brainstorming 不可跳过

```yaml
除非使用 hotfix/tweak 预设:
  - 每个项目必须经过 brainstorming
  - 即使是 "简单" 项目
  - 设计可以短，但必须有

原因:
  - "简单" 项目是未检查假设造成最多浪费的地方
  - 短设计（几句话）可以接受，但必须有
```

### 9.2 显式优于隐式

```yaml
必须显式选择:
  - 执行方式（subagent-driven / executing-plans）
  - 分支处理方式
  - 模型选择

禁止:
  - 隐式默认
  - 未声明的假设
```

### 9.3 质量门槛

```yaml
每个任务:
  - 自我审查
  - 规格符合性审查
  - 代码质量审查
  - 最终整体审查
```

### 9.4 规模自适应

```yaml
任务复杂度信号:
  - 1-2 文件 + 完整 spec → 便宜模型
  - 多文件 + 集成考虑 → 标准模型
  - 需要设计判断 → 最强模型
```

---

## 10. 总结

### 10.1 Superpowers 核心职责

1. **技术设计** - brainstorming 产出 Design Doc
2. **计划制定** - writing-plans 创建可执行计划
3. **代码执行** - subagent-driven 或 executing-plans 实施
4. **质量验证** - 多阶段审查确保质量
5. **分支收尾** - finishing-a-branch 完成工作

### 10.2 关键 Skill

| Skill | 职责 | 下一 Skill |
|-------|------|------------|
| brainstorming | 设计 | writing-plans |
| writing-plans | 计划 | subagent-driven / executing-plans |
| subagent-driven-development | 执行（子代理） | finishing-a-branch |
| executing-plans | 执行（当前会话） | finishing-a-branch |
| finishing-a-development-branch | 收尾 | Done |

### 10.3 核心理念

- **设计先行** - 先设计，后实现
- **审查驱动** - 每个任务必须经过审查
- **质量优先** - 不牺牲质量追求速度
- **规模适配** - 根据任务复杂度选择合适方法

---

**生成日期**: 2026-06-28  
**文档版本**: v1.0.0
