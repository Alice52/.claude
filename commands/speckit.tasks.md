---
description: 基于可用设计产物为该功能生成可执行、带依赖顺序的 tasks.md。
handoffs:
  - label: 分析一致性
    agent: speckit.analyze
    prompt: 运行项目一致性分析
    send: true
  - label: 实施项目
    agent: speckit.implement
    prompt: 实施项目
    send: true
---


## 用户输入

```text
$ARGUMENTS
```

在继续前，若输入非空，必须先审视用户输入。

## 概览

1. **初始化**：从仓库根目录依次运行插件预处理与 `.specify/scripts/bash/check-prerequisites.sh --json`，解析 FEATURE_DIR 与 AVAILABLE_DOCS。所有路径须为绝对路径。若参数含有单引号（如 "I'm Groot"），请使用 `'I'\''m Groot'` 或 `"I'm Groot"`。
   - **插件预处理**：执行 `.specify/scripts/bash/plugin-loader.sh --hook pre-tasks` 并捕获其标准输出（JSON 结构）备用。如插件返回 `10`/`20`，视为警告或建议，记录信息即可。
   - **主流程初始化**：随后执行 `.specify/scripts/bash/check-prerequisites.sh --json` 并保存其 JSON 输出。

2. **加载设计文档**：从 FEATURE_DIR 读取：
   - **必需**：`plan.md`（技术栈、依赖、结构）、`spec.md`（按优先级排列的用户故事）
   - **可选**：`data-model.md`（实体）、`contracts/`（API 端点）、`research.md`（决策）、`quickstart.md`（测试场景）
   - 注意：并非所有项目都具备全部文件，需基于现有资料生成任务。

3. **执行任务生成流程**：
   - 解析 `plan.md`，提取技术栈、依赖、项目结构
   - 解析 `spec.md`，提取用户故事及优先级（P1、P2、P3…）
   - 若存在 `data-model.md`：提取实体并映射到用户故事
   - 若存在 `contracts/`：将端点映射到用户故事
   - 若存在 `research.md`：抽取决策用于 Setup 任务
   - 将插件返回的 `additional_tasks` 与 `task_augmentations` 并入对应阶段/故事，确保任务格式仍符合规范
   - 按用户故事组织任务（见后文规则）
   - 生成用户故事完成顺序的依赖图
   - 为每个用户故事提供可并行执行的示例
   - 校验任务完整性（每个用户故事都具备独立可测试的完整任务集合）

4. **生成 tasks.md**：使用 `.specify/templates/tasks-template.md` 作为结构，填充以下内容：
   - 取自 `plan.md` 的正确功能名称
   - Phase 1：Setup 任务（项目初始化）
   - Phase 2：Foundational 任务（所有故事的前置阻塞项）
   - Phase 3 及之后：按 `spec.md` 优先级为每个用户故事单独成相位
   - 每个阶段包含：故事目标、独立测试标准、（如有需求）测试任务、实现任务
   - 最终阶段：Polish & cross-cutting concerns
   - 所有任务必须遵循严格的清单格式（见规则）
   - 为每项任务提供明确路径
   - 添加 Dependencies 部分，展示用户故事的完成顺序（同步包含插件追加的依赖信息，如有）
   - 为每个故事列出并行执行示例（包含插件追加的任务）
   - 提供实施策略部分（先 MVP，再增量交付）
   - 写入文件后执行 `.specify/scripts/bash/plugin-loader.sh --hook post-tasks`，允许插件进行收尾注入或校验；若返回 `10`/`20`，记录提示即可

5. **汇报**：输出生成的 tasks.md 路径及摘要：
   - 任务总数
   - 每个用户故事的任务数
   - 已识别的并行机会
   - 每个故事的独立测试标准
   - 建议的 MVP 范围（通常为第一个用户故事）
   - 格式校验：确认所有任务均符合检查清单格式（复选框、ID、标签、路径）
   - 汇总 `.specify/scripts/bash/plugin-loader.sh --hook post-tasks` 输出的警告或建议，确保被纳入最终反馈

任务生成上下文：$ARGUMENTS

生成的 tasks.md 必须可立即执行——每个任务都要具体到 LLM 无需额外上下文即可完成。

## 任务生成规则

**关键**：任务必须按用户故事组织，以便独立实施与测试。

**测试为可选**：仅当规格明确要求或用户声明采用 TDD 时生成测试任务。

### 检查清单格式（必需）

所有任务必须严格遵循以下格式：

```text
- [ ] [TaskID] [P?] [Story?] Description with file path
```

**格式组成**：

1. **复选框**：始终以 `- [ ]` 开头
2. **任务 ID**：按执行顺序递增（T001、T002、T003…）
3. **[P] 标记**：仅在任务可并行（作用于不同文件、无未完成依赖）时添加
4. **[Story] 标签**：仅在用户故事阶段必填
   - 形如 [US1]、[US2]、[US3]（与 `spec.md` 中的故事对应）
   - Setup 阶段：无故事标签
   - Foundational 阶段：无故事标签  
   - 用户故事阶段：必须带故事标签
   - Polish 阶段：无故事标签
5. **描述**：明确动作并包含准确文件路径

**示例**：

- ✅ 正确：`- [ ] T001 按实施计划创建项目结构`
- ✅ 正确：`- [ ] T005 [P] 在 src/middleware/auth.py 中实现认证中间件`
- ✅ 正确：`- [ ] T012 [P] [US1] 在 src/models/user.py 中创建 User 模型`
- ✅ 正确：`- [ ] T014 [US1] 在 src/services/user_service.py 中实现 UserService`
- ❌ 错误：`- [ ] Create User model`（缺少 ID 与故事标签）
- ❌ 错误：`T001 [US1] Create model`（缺少复选框）
- ❌ 错误：`- [ ] [US1] Create User model`（缺少任务 ID）
- ❌ 错误：`- [ ] T001 [US1] Create model`（缺少文件路径）

### 任务组织

1. **来源：用户故事（spec.md）——主干结构**：
   - 每个用户故事（P1、P2、P3…）占独立阶段
   - 将相关组件映射至对应故事：
     - 该故事所需的模型
     - 该故事所需的服务
     - 该故事所需的端点/UI
     - 若要求测试：故事专属测试任务
   - 标明故事之间的依赖（多数故事应相互独立）
   
2. **来源：契约（contracts/）**：
   - 将每个契约/端点映射到对应的故事
   - 若要求测试：在该故事阶段先生成契约测试任务 `[P]`，再安排实现
   
3. **来源：数据模型（data-model.md）**：
   - 将每个实体映射到需要它的故事
   - 若实体服务于多个故事：放到最早的故事或 Setup 阶段
   - 实体关系 → 在对应故事阶段生成服务层任务
   
4. **来源：初始化/基础设施**：
   - 共享基础设施 → Setup 阶段（Phase 1）
   - 基础阻塞任务 → Foundational 阶段（Phase 2）
   - 特定故事的初始化 → 放在该故事阶段内

### 阶段结构

- **Phase 1**：Setup（项目初始化）
- **Phase 2**：Foundational（共有阻塞项，必须完成后才进入用户故事）
- **Phase 3+**：按优先级排列的用户故事（P1、P2、P3…）
  - 每个故事内的执行顺序建议：Tests（如需）→ Models → Services → Endpoints → Integration
  - 每个阶段都应形成可独立测试的增量
- **Final Phase**：Polish & Cross-Cutting Concerns
