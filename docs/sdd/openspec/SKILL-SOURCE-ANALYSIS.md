# OpenSpec Skill 源码分析

本文档分析 OpenSpec 相关的 Skill 源码，包括 `openspec-explore`、`openspec-new-change`、`openspec-propose` 等。

---

## 1. OpenSpec 架构总览

```
┌─────────────────────────────────────────────────────────────────────┐
│                        OpenSpec 系统架构                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ┌─────────────────────────────────────────────────────────────┐ │
│   │                      OpenSpec Skills                         │ │
│   │                                                              │ │
│   │  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐   │ │
│   │  │ openspec-    │───▶│ openspec-    │───▶│ openspec-    │   │ │
│   │  │ explore      │    │ new-change   │    │ propose      │   │ │
│   │  └──────────────┘    └──────────────┘    └──────────────┘   │ │
│   │                                                              │ │
│   │  ┌──────────────┐    ┌──────────────┐                      │ │
│   │  │ openspec-    │    │ openspec-    │                      │ │
│   │  │ verify-change│    │ list/status  │                      │ │
│   │  └──────────────┘    └──────────────┘                      │ │
│   │                                                              │ │
│   └─────────────────────────────────────────────────────────────┘ │
│                                                                     │
│   ┌─────────────────────────────────────────────────────────────┐ │
│   │                      核心数据结构                           │ │
│   │                                                              │ │
│   │  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐  │ │
│   │  │ .openspec    │    │ change/      │    │ specs/       │  │ │
│   │  │ .yaml        │    │              │    │              │  │ │
│   │  │ (元数据)     │    │ (内容文件)   │    │ (能力规格)   │  │ │
│   │  └──────────────┘    └──────────────┘    └──────────────┘  │ │
│   │                                                              │ │
│   └─────────────────────────────────────────────────────────────┘ │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. OpenSpec 核心概念

### 2.1 什么是 OpenSpec

OpenSpec 是一个**变更提案和需求管理系统**，关注 **WHAT**（做什么、为什么）：
- 问题背景（Why）
- 变更目标（What）
- 范围界定（Scope）
- 能力规格（Capability Spec）

### 2.2 核心文件结构

```
openspec/
├── config.yaml                    # 全局配置
├── changes/                       # 变更目录
│   ├── <name>/                    # 活跃变更
│   │   ├── .openspec.yaml        # 变更元数据
│   │   ├── proposal.md           # 提案（Why + What）
│   │   ├── design.md             # 高层设计
│   │   ├── tasks.md              # 任务清单
│   │   └── specs/                # Delta 能力规格
│   │       └── <capability>/
│   │           └── spec.md
│   └── archive/                   # 归档目录
│       └── YYYY-MM-DD-<name>/
└── specs/                         # 主能力规格
    └── <capability>/
        └── spec.md
```

### 2.3 与 Superpowers 的区别

| 维度 | OpenSpec | Superpowers |
|------|----------|-------------|
| **关注点** | WHAT（做什么） | HOW（怎么做） |
| **核心问题** | Why + What | 技术实现细节 |
| **主要产出** | proposal, design, spec | Design Doc, Plan, Code |
| **目标读者** | PM、架构师、业务方 | 开发者 |

---

## 3. openspec-explore Skill

### 3.1 功能定位
**探索问题空间** - 在创建正式变更前，自由探索和理解需求。

### 3.2 典型工作流程

```yaml
入口: 用户提出模糊需求或问题

流程:
  1. 理解当前项目上下文
     - 检查现有文件结构
     - 阅读相关文档
     - 了解技术栈

  2. 询问澄清问题
     - 一次一个问题
     - 使用选择题或开放性问题
     - 聚焦：目的、约束、成功标准

  3. 探索可能的方案
     - 提出 2-3 种不同方案
     - 分析每种方案的权衡
     - 给出推荐和理由

  4. 形成建议
     - 汇总发现
     - 推荐下一步行动
     - 可能建议创建正式 change

出口: 形成明确的需求理解或变更建议
```

### 3.3 关键约束

```yaml
禁止:
  - 直接开始实现
  - 编写代码
  - 创建文件（除非用于辅助理解）

必须:
  - 充分理解问题后再建议行动
  - 与用户确认理解是否正确
```

---

## 4. openspec-new-change Skill

### 4.1 功能定位
**创建正式变更结构** - 初始化 change 目录和核心文件。

### 4.2 创建的文件

```yaml
openspec/changes/<name>/
├── .openspec.yaml              # 变更元数据
├── proposal.md                # 提案文档
├── design.md                  # 高层设计
└── tasks.md                   # 任务清单
```

### 4.3 .openspec.yaml 结构

```yaml
name: spring-boot-hello-world
description: 实现一个 Spring Boot Hello World demo
author: user
created_at: 2026-06-28
capabilities:
  - spring-boot-app
status: active
```

### 4.4 proposal.md 模板

```markdown
# <Change Name>

## 问题背景 (Why)
- 当前面临的问题
- 为什么需要这个变更
- 不解决会有什么影响

## 目标 (What)
- 期望达成的具体目标
- 可衡量的成功标准

## 范围

### 包含
- 具体要做什么

### 不包含
- 明确排除的内容

## 非目标
- 不会涉及的方向
```

### 4.5 design.md 模板

```markdown
# <Change Name> 设计

## 架构决策
- 技术选型
- 方案概述

## 项目结构
```
目录树...
```

## 数据流
- 关键流程描述

## API 设计
| 端点 | 方法 | 描述 |
```

### 4.6 tasks.md 格式

```markdown
# 任务清单

## Phase 1: Open
- [x] 创建 change 目录
- [x] 编写 proposal.md
- [x] 编写 design.md

## Phase 2: Design
- [ ] 技术设计
- [ ] API 规格定义

## Phase 3: Build
- [ ] 任务 1
- [ ] 任务 2
```

---

## 5. openspec-propose Skill

### 5.1 功能定位
**形成变更建议** - 当用户意图不明确时，帮助形成结构化提案。

### 5.2 与 explore 的区别

| openspec-explore | openspec-propose |
|------------------|------------------|
| 自由探索问题空间 | 结构化形成建议 |
| 输出是理解 | 输出是提案草案 |
| 可能不产生文件 | 产生 proposal.md 草案 |

### 5.3 工作流程

```yaml
入口: 用户有想法但不够清晰

流程:
  1. 询问关键问题
     - 问题背景
     - 期望结果
     - 约束条件

  2. 形成草案
     - 基于回答编写 proposal.md 草案
     - 包含初步的 design.md

  3. 与用户确认
     - 呈现草案
     - 征求修改意见
     - 迭代直到满意

  4. 创建正式 change
     - 调用 openspec-new-change
     - 或引导用户确认

出口: 形成可执行的变更提案
```

---

## 6. openspec-verify-change Skill

### 6.1 功能定位
**验证变更符合规格** - 深度检查实现与设计的符合性。

### 6.2 触发条件
- 由 Superpowers 系统在验证阶段调用
- 当规模评估为"大"时触发

### 6.3 检查项

```yaml
1. tasks.md 完整性
   - 所有任务已完成 [x]
   - 没有未完成的任务 [ ]

2. proposal.md 符合性
   - 目标已实现
   - 范围已覆盖

3. design.md 符合性
   - 架构决策已遵循
   - 技术选型一致

4. 能力规格符合性
   - delta spec 场景全部通过
   - 验收条件满足

5. Delta Spec 漂移检测
   - 若 spec 有变更，design doc 是否体现
   - 标注 Implementation Divergence（如有）
```

### 6.4 输出格式

```yaml
验证报告:
  检查项:
    - [✓] tasks.md 完整性
    - [✓] proposal 目标满足
    - [✓] design 架构遵循
    - [✗] spec 场景覆盖 (3/5 通过)
    - [✓] 无 spec 漂移

  发现:
    - 问题 1: 描述
    - 建议: 行动

  结论: PASS / FAIL / CONDITIONAL PASS
```

---

## 7. OpenSpec 工具命令

### 7.1 openspec list

```bash
# 列出所有活跃 changes
openspec list

# JSON 格式输出
openspec list --json

# 输出示例:
[
  {
    "name": "spring-boot-hello-world",
    "status": "active",
    "created_at": "2026-06-28",
    "capabilities": ["spring-boot-app"]
  }
]
```

### 7.2 openspec status

```bash
# 查看 change 状态
openspec status --change <name>

# JSON 格式
openspec status --change <name> --json

# 输出示例:
{
  "name": "spring-boot-hello-world",
  "phase": "build",
  "progress": "3/5 tasks completed",
  "files": {
    "proposal": true,
    "design": true,
    "tasks": true,
    "specs": 2
  }
}
```

### 7.3 openspec archive

```bash
# 归档 change（通常由工具自动调用）
openspec archive <name>

# 效果:
# 1. 移动 changes/<name>/ 到 changes/archive/YYYY-MM-DD-<name>/
# 2. 更新 status: archived
# 3. 同步 delta specs 到主 specs/
```

---

## 8. Delta Spec 机制

### 8.1 什么是 Delta Spec

Delta Spec 是在变更期间维护的**增量能力规格**，记录相对于主 spec 的变更。

### 8.2 结构

```markdown
# <Capability Name> Delta Spec

## NEW Requirements
新增的需求

## MODIFIED Requirements
修改的需求（仅 hotfix）

## REMOVED Requirements
移除的需求（如有）

## Acceptance Criteria
- [ ] 场景 1
- [ ] 场景 2
```

### 8.3 生命周期

```
创建 (openspec-new-change)
    ↓
更新 (设计/构建阶段)
    ↓
验证 (openspec-verify-change)
    ↓
归档 (openspec archive)
    ↓
    ├─ 复制到 openspec/specs/<capability>/spec.md
    └─ 标注主 spec 版本
```

### 8.4 与主 Spec 的关系

```
openspec/specs/auth/spec.md          # 主 spec（长期维护）
    ↑
    │ 归档时复制
    │
openspec/changes/<name>/specs/auth/spec.md   # delta spec（临时）
```

---

## 9. OpenSpec 状态流转

### 9.1 Change 生命周期

```
┌─────────────────────────────────────────────────────────────────┐
│                      Change 生命周期                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────┐                                                   │
│  │  Draft  │  草稿状态                                         │
│  └────┬────┘                                                   │
│       │  openspec-new-change                                    │
│       ▼                                                         │
│  ┌─────────┐                                                   │
│  │ Active  │  活跃状态（可被引用）                              │
│  └────┬────┘                                                   │
│       │  实施中                                                 │
│       ▼                                                         │
│  ┌─────────┐                                                   │
│  │Completed│  实施完成，待验证                                  │
│  └────┬────┘                                                   │
│       │  验证通过                                               │
│       ▼                                                         │
│  ┌─────────┐                                                   │
│  │Archived │  已归档（delta spec 已合并到主 spec）             │
│  └─────────┘                                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 9.2 状态字段

```yaml
# .openspec.yaml
status: draft | active | completed | archived

# 或更详细
status:
  state: active
  phase: design  # 可选，与外部流程关联
  progress: 3/5
```

---

## 10. 与其他系统的协作

### 10.1 与 Superpowers 的接口

```yaml
OpenSpec 产出 → Superpowers 输入:

proposal.md:
  - 目标描述 → Design Doc 的需求来源
  - 范围界定 → Plan 的任务边界
  
design.md:
  - 高层架构 → Design Doc 的设计基础
  - API 概览 → Design Doc 的初始输入

tasks.md:
  - 任务列表 → Plan 的任务来源
  - checkbox → 进度追踪

delta specs:
  - 能力规格 → Design Doc 的验收标准来源
```

### 10.2 与 Comet 的集成

```yaml
Comet 调用 OpenSpec Skills:
  Phase 1 (Open):
    - 调用 openspec-explore（可选）
    - 调用 openspec-new-change
    - 或调用 openspec-propose

  Phase 4 (Verify):
    - 调用 openspec-verify-change（完整验证时）

  Phase 5 (Archive):
    - 调用 openspec archive（内部命令）
```

---

## 11. 扩展机制

### 11.1 自定义字段

```yaml
# .openspec.yaml 可扩展字段
name: my-change
custom:
  priority: high
  estimated_effort: 3d
  stakeholders:
    - team-a
    - team-b
```

### 11.2 自定义验证规则

```yaml
# config.yaml
validation:
  required_fields:
    - name
    - description
    - priority
  custom_rules:
    - name: priority_check
      condition: priority in [low, medium, high]
```

### 11.3 钩子机制

```yaml
# config.yaml
hooks:
  on_create: scripts/on-create.sh
  on_archive: scripts/on-archive.sh
```

---

## 12. 总结

### 12.1 OpenSpec 核心职责

1. **需求捕获** - 通过 explore/propose 理解问题
2. **变更结构化** - 通过 new-change 创建标准结构
3. **规格管理** - 通过 delta spec 跟踪能力变更
4. **符合性验证** - 通过 verify-change 确保实现符合设计
5. **生命周期管理** - 从创建到归档的完整流程

### 12.2 关键 Skill

| Skill | 职责 | 触发时机 |
|-------|------|----------|
| openspec-explore | 探索问题空间 | 需求不明确时 |
| openspec-new-change | 创建变更结构 | 需求明确时 |
| openspec-propose | 形成变更建议 | 需要结构化提案时 |
| openspec-verify-change | 验证符合性 | 验证阶段 |

### 12.3 设计理念

- **What 先于 How** - 先明确做什么，再考虑怎么做
- **活文档** - Delta spec 可在实施中更新
- **显式状态** - 变更状态明确可追踪
- **规格驱动** - 能力规格是验收的核心依据

---

**生成日期**: 2026-06-28  
**文档版本**: v1.0.0
