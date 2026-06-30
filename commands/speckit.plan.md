---
description: 使用计划模板执行实施规划流程并生成设计产物。
handoffs:
  - label: 创建任务
    agent: speckit.tasks
    prompt: 将计划分解为任务
    send: true
  - label: 创建检查清单
    agent: speckit.checklist
    prompt: 为需求创建质量检查清单
    send: true
---


## 用户输入

```text
$ARGUMENTS
```

在继续前，若输入非空，必须先审视用户输入。

## 概览

1. **初始化**：从仓库根目录依次运行插件预处理与 `.specify/scripts/bash/setup-plan.sh --json`，解析 JSON 获取 FEATURE_SPEC、IMPL_PLAN、SPECS_DIR、BRANCH。若参数包含单引号（如 "I'm Groot"），请使用 `'I'\''m Groot'` 或 `"I'm Groot"`。
   - **插件预处理**：执行 `.specify/scripts/bash/plugin-loader.sh --hook pre-plan`。插件可能返回 `10` 或 `20` 作为警告/建议，请记录信息但不要中断流程。
   - **主流程初始化**：随后执行 `.specify/scripts/bash/setup-plan.sh --json` 并保留其 JSON 输出。

2. **加载上下文**：读取 FEATURE_SPEC 与 `/.specify/memory/constitution.md`，并加载已复制的 IMPL_PLAN 模板。

3. **执行计划流程**：按照 IMPL_PLAN 模板结构完成以下内容：
   - 填写 Technical Context（未知项标注为 “NEEDS CLARIFICATION”）
   - 根据宪章填写 Constitution Check
   - 评估各项门禁（如存在未解释的违规定义则报错）
   - Phase 0：生成 `research.md`，解决所有 NEEDS CLARIFICATION
   - Phase 1：生成 `data-model.md`、`contracts/`、`quickstart.md`
   - Phase 1：运行代理脚本更新 agent 上下文
   - 设计完成后重新评估 Constitution Check

4. **后置插件与汇报**：在生成全部产物后运行插件后处理 `.specify/scripts/bash/plugin-loader.sh --hook post-plan`，将输出写回 plan.md 等文件。随后汇总分支名、IMPL_PLAN 路径及生成的产物；若插件产生附加提示，也需同步至汇报结果。

## 各阶段说明

### Phase 0：梳理与调研

1. **从 Technical Context 提取未知项**：
   - 每个 “NEEDS CLARIFICATION” → 生成调研任务
   - 每个依赖项 → 生成最佳实践任务
   - 每个集成点 → 生成模式调研任务

2. **生成并派发调研代理**：
   ```
   For each unknown in Technical Context:
     Task: "Research {unknown} for {feature context}"
   For each technology choice:
     Task: "Find best practices for {tech} in {domain}"
   ```
   （上述任务语句需按原文字符串派发。）

3. **在 `research.md` 中整合结论**，建议格式：
   - Decision（决策）：[最终选择]
   - Rationale（理由）：[选择原因]
   - Alternatives considered（备选方案）：[评估过的其他方案]

**产出**：`research.md`，其中所有 NEEDS CLARIFICATION 均已解决。

### Phase 1：设计与契约

**前置条件**：`research.md` 已完成。

1. **从规格提取实体** → 写入 `data-model.md`：
   - 实体名称、字段、关系
   - 来自需求的校验规则
   - 若适用，记录状态转换

2. **基于功能需求生成 API 契约**：
   - 每个用户动作对应一个端点
   - 遵循标准 REST/GraphQL 模式
   - 输出 OpenAPI/GraphQL Schema 至 `/contracts/`

3. **更新代理上下文**：
   - 运行 `.specify/scripts/bash/update-agent-context.sh claude`
   - 脚本会识别当前使用的 AI 代理
   - 更新匹配的代理上下文文件
   - 仅追加本次计划新增的技术信息
   - 保留标记之间的人工补充内容

**产出**：`data-model.md`、`/contracts/*`、`quickstart.md`、代理上下文文件。

## 关键规则

- 路径必须为绝对路径
- 任一门禁失败或存在未解决澄清时需立即报错
