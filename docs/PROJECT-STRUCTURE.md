# 项目文档结构说明

本文档详细说明 `docs/` 和 `openspec/` 目录中各文件的作用及其在 Comet 开发流程中的出现阶段。

---

## 目录结构总览

```
├── docs/                          # Superpowers - HOW（技术实现）
│   └── superpowers/
│       ├── specs/                 # 技术设计文档（Phase 2）
│       └── plans/                 # 实施计划（Phase 3）
│
└── openspec/                      # OpenSpec - WHAT（需求管理）
    └── changes/
        └── archive/               # 归档的变更（Phase 5）
            └── <change-name>/
                ├── .openspec.yaml # Change 元数据
                ├── .comet.yaml    # Comet 状态追踪
                ├── proposal.md    # 提案文档
                ├── design.md      # 高层设计
                └── tasks.md       # 任务清单
```

---

## OpenSpec 文件（WHAT - 需求/目标）

| 文件 | 阶段 | 作用 | 关键内容 |
|------|------|------|----------|
| `.openspec.yaml` | Phase 1: Open | Change 元数据文件 | `name`, `description`, `author`, `created_at`, `capabilities` |
| `proposal.md` | Phase 1: Open | **提案文档** - 回答 Why + What | 问题背景、目标、范围、非目标 |
| `design.md` | Phase 1: Open | **高层设计** - 回答 How（粗粒度） | 架构决策、方案选型、数据流、API 概览 |
| `tasks.md` | Phase 1-5 | **任务清单** - 追踪进度 | 各阶段的 checkbox 任务列表 |
| `.comet.yaml` | Phase 1-5 | **状态追踪** - Comet 工作流状态 | `phase`, `design_doc`, `plan`, `verify_result`, `archived` |

---

## Superpowers 文件（HOW - 技术实现）

| 文件 | 阶段 | 作用 | 关键内容 |
|------|------|------|----------|
| `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md` | Phase 2: Design | **技术设计文档（RFC）** | 技术决策、详细架构、API 规格、依赖清单、验证标准 |
| `docs/superpowers/plans/YYYY-MM-DD-<feature>.md` | Phase 3: Build | **实施计划** | 任务分解、执行策略、验证计划、文件头关联 change |

---

## 各阶段产物对应表

```
┌────────────────────────────────────────────────────────────────────────────────┐
│                           Comet 开发流程                                        │
├──────────┬──────────────────────────┬──────────────────────────────────────────┤
│   阶段   │     OpenSpec 产物        │        Superpowers 产物                  │
├──────────┼──────────────────────────┼──────────────────────────────────────────┤
│ Phase 1  │ • .openspec.yaml         │                                          │
│  Open    │ • proposal.md (Why+What) │                                          │
│          │ • design.md (高层 How)   │                                          │
│          │ • tasks.md (初始化)      │                                          │
│          │ • .comet.yaml (phase:    │                                          │
│          │   open)                  │                                          │
├──────────┼──────────────────────────┼──────────────────────────────────────────┤
│ Phase 2  │ • .comet.yaml (phase:    │ • docs/superpowers/specs/                │
│  Design  │   design)                │   YYYY-MM-DD-topic-design.md             │
│          │ • tasks.md (设计任务 ✓)  │   (技术 RFC: 技术决策、API 规格、        │
│          │                          │    依赖、验证标准)                       │
├──────────┼──────────────────────────┼──────────────────────────────────────────┤
│ Phase 3  │ • .comet.yaml (phase:    │ • docs/superpowers/plans/                │
│  Build   │   build)                 │   YYYY-MM-DD-feature.md                  │
│          │ • tasks.md (构建任务 ✓)  │   (实施计划: 任务分解、执行策略)         │
│          │                          │                                          │
│          │   → 代码实现 src/        │   → 代码实现                             │
├──────────┼──────────────────────────┼──────────────────────────────────────────┤
│ Phase 4  │ • .comet.yaml (phase:    │                                          │
│  Verify  │   verify,                │                                          │
│          │   verify_result: pass)   │                                          │
│          │ • tasks.md (验证任务 ✓)  │                                          │
├──────────┼──────────────────────────┼──────────────────────────────────────────┤
│ Phase 5  │ • 移至 archive/          │ • specs/ 文档标注                        │
│  Archive │   目录                   │   `archived-with: change-name`           │
│          │ • .comet.yaml (phase:    │ • plans/ 文档标注                        │
│          │   archive,               │   `archived-with: change-name`           │
│          │   archived: true)        │                                          │
└──────────┴──────────────────────────┴──────────────────────────────────────────┘
```

---

## 文件关联关系

```
OpenSpec (WHAT)                          Superpowers (HOW)
─────────────────────────────────────────────────────────────────────

openspec/changes/<name>/                 docs/superpowers/specs/
├── proposal.md ─────────────────────┐   └── <date>-topic-design.md
├── design.md (高层) ────────────────┤        ▲
└── .openspec.yaml                    │        │ 引用
                                      │        │
openspec/changes/<name>/              │   docs/superpowers/plans/
└── .comet.yaml                       │   └── <date>-feature.md
    ├── design_doc ───────────────────┘        ▲
    │  = specs/YYYY-MM-DD-topic-design.md      │
    │                                          │
    └── plan ──────────────────────────────────┘
       = plans/YYYY-MM-DD-feature.md
       (文件头: change: <name>)
```

---

## 实际示例（本项目）

| 文件路径 | 所属 | 阶段 | 当前状态 |
|----------|------|------|----------|
| `openspec/changes/archive/2026-06-28-spring-boot-hello-world/.../.openspec.yaml` | OpenSpec | Phase 5 (已归档) | archived |
| `.../proposal.md` | OpenSpec | Phase 5 (已归档) | 需求文档 |
| `.../design.md` | OpenSpec | Phase 5 (已归档) | 高层设计 |
| `.../tasks.md` | OpenSpec | Phase 5 (已归档) | 任务完成 |
| `.../.comet.yaml` | Comet | Phase 5 (已归档) | phase: archive |
| `docs/superpowers/specs/2026-06-28-spring-boot-hello-design.md` | Superpowers | Phase 5 (已归档) | archived-with: spring-boot-hello-world |
| `docs/superpowers/plans/2026-06-28-spring-boot-hello.md` | Superpowers | Phase 5 (已归档) | archived-with: spring-boot-hello-world |

---

## OpenSpec vs Superpowers 对比

| 维度 | OpenSpec | Superpowers |
|------|----------|-------------|
| **关注点** | WHAT（做什么、为什么） | HOW（怎么做、技术细节） |
| **文档粒度** | 业务/用户视角 | 技术实现视角 |
| **目标读者** | PM、架构师、业务方 | 开发者、架构师 |
| **变更管理** | proposal → archive | Design Doc → Plan → Archive |
| **生命周期** | 随 change 归档 | 设计文档长期保存，标注状态 |

---

## 使用建议

1. **Phase 1 (Open)**: 先在 OpenSpec 中写清楚问题和目标，再启动技术设计
2. **Phase 2 (Design)**: 技术设计文档要引用对应的 OpenSpec change
3. **Phase 3 (Build)**: 实施计划文件头必须包含 `change:` 和 `design-doc:` 元数据
4. **Phase 5 (Archive)**: 归档时同步更新 design doc 和 plan 的状态标注

---

**生成日期**: 2026-06-28  
**关联 Change**: spring-boot-hello-world  
**文档版本**: v1.0.0
