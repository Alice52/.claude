---
description: 根据用户需求为当前特性生成定制检查清单。
---


## 清单定位：“需求写作的单元测试”

**核心理念**：检查清单是**用来检验需求文本质量的单元测试**。它关注规格是否写得准确、完整、可验证，而不是实现代码是否运行正常。

**绝对不要做的事**：
- ❌ 禁止书写 “Verify the button clicks correctly”
- ❌ 禁止书写 “Test error handling works”
- ❌ 禁止书写 “Confirm the API returns 200”
- ❌ 禁止用来核对实现是否符合规格

**应该关注的要点**：
- ✅ “是否为所有卡片类型定义了视觉层级？”（Completeness）
- ✅ “‘突出展示’ 是否通过具体尺寸或位置量化？”（Clarity）
- ✅ “各交互元素的悬停状态要求是否一致？”（Consistency）
- ✅ “是否明确了键盘导航的可访问性要求？”（Coverage）
- ✅ “规格是否说明了徽标加载失败时的处理？”（Edge Case）

类比：如果把规格看成“用英语写的代码”，检查清单就是它的“单元测试套件”。我们要验证的是需求文本是否写得好，而不是实现是否可用。

## 用户输入

```text
$ARGUMENTS
```

若输入非空，必须先阅读并考虑，再继续执行。

## 执行步骤

1. **初始化**
   - 从仓库根目录运行 `.specify/scripts/bash/check-prerequisites.sh --json`，解析 JSON，获取 FEATURE_DIR 与 AVAILABLE_DOCS。
   - 所有文件路径必须是绝对路径。
   - 参数中若含单引号（例如 "I'm Groot"），需使用 `'I'\''m Groot'` 或改用双引号 `"I'm Groot"`。

2. **动态澄清用户意图**
   - 最多生成三条澄清问题（Q1–Q3），问题内容需根据用户输入及 spec/plan/tasks 中的信号即时推导。
   - 每个问题都必须影响检查清单内容；若 `$ARGUMENTS` 已清楚说明，则跳过对应问题。
   - 问题必须精准，避免泛泛而谈。

   **生成问题的参考流程**：
   1. 提取信号：领域关键词（auth、latency、UX、API 等）、风险词（critical、must、compliance）、角色线索（QA、review、security team）、交付物提示（a11y、rollback、contracts）。
   2. 将信号聚类为最多四个关注方向，并按相关度排序。
   3. 推断潜在受众与使用时机（作者自查/评审/QA/发布）。
   4. 识别仍缺失的决策维度：范围、深度、风险、排除项、可衡量标准等。
   5. 以如下模板构造问题：
      - 范围细化：例如“此清单是否需要覆盖与 X、Y 的集成，还是仅限本地模块？”
      - 风险优先级：例如“哪些风险领域必须成为强制检查项？”
      - 深度设定：例如“这是提交前的快速自检，还是必须全部通过的发布闸门？”
      - 受众定位：例如“清单主要由作者使用，还是给 PR 评审同事使用？”
      - 边界排除：例如“是否要明确排除性能调优项？”
      - 场景缺失：例如“尚未提及恢复流程——回滚/部分失败场景是否纳入？”

   **提问格式要求**：
   - 若使用选项，生成“选项 | 候选答案 | 价值说明”的紧凑表格，最多 A–E。
   - 若自由回答更合适，则直接采用简短描述。
   - 禁止让用户重复已有信息。
   - 如无法确认范围，应以“请确认 X 是否在范围内”形式询问。

   **无法交互时的默认值**：
   - 深度：Standard
   - 受众：若与代码相关默认为 Reviewer（PR），否则为 Author
   - 关注点：选择相关度最高的两个聚类

   若三个问题后仍有两类以上场景（备用/异常/恢复/非功能领域）不明确，可追加 Q4/Q5（总问题数 ≤5），每题需附一句理由（如 “仍存在恢复流程风险”）。用户若拒绝继续追问，应立即停止。

3. **理解用户意图**
   - 综合 `$ARGUMENTS` 与澄清回答，确定检查清单主题（如 security、review、deploy、ux）。
   - 归纳用户强调的必选项。
   - 将关注点映射到清单分类结构。
   - 如需补充上下文，可参考 spec/plan/tasks，但禁止臆造。

4. **加载特性上下文**
   - 读取 FEATURE_DIR 中的 spec.md、plan.md（若存在）、tasks.md（若存在）。
   - 采用渐进式读取：先抓取与关注点最相关的段落，再按需追加。
   - 大段文本应提炼成简要要点，而不是直接粘贴原文。

5. **生成检查清单（需求的“单元测试”）**
   - 若 `FEATURE_DIR/checklists/` 不存在需先创建。
   - 清单文件名需短小且能表示主题（如 `ux.md`、`api.md`、`security.md`），格式 `[domain].md`。已存在同名文件时应追加。
   - 条目编号从 CHK001 递增。
   - 每次运行 `/speckit.checklist` 都会创建新的清单文件，不覆盖旧文件。

   **核心原则：检查需求，而非实现**
   - **Completeness**：需求是否覆盖全部必要情形？
   - **Clarity**：描述是否明确、无歧义？
   - **Consistency**：不同部分是否互相一致？
   - **Measurability**：是否可以客观衡量？
   - **Coverage**：场景、边界、错误流程是否完备？

   **推荐的分类结构**：
   - Requirement Completeness（需求完备性）
   - Requirement Clarity（需求清晰度）
   - Requirement Consistency（需求一致性）
   - Acceptance Criteria Quality（验收标准质量）
   - Scenario Coverage（场景覆盖）
   - Edge Case Coverage（边界情况）
   - Non-Functional Requirements（非功能需求）
   - Dependencies & Assumptions（依赖与假设）
   - Ambiguities & Conflicts（歧义与冲突）

   **编写条目的技巧**：
   - 使用问句形式，聚焦需求文档中的“写了什么 / 没写什么”。
   - 方括号注明质量维度，例如 `[Completeness]`、`[Gap]`、`[Spec §X.Y]`。
   - 每条都应帮助判断需求是否具备可实施性、可验证性。

   **示例**：
   - Completeness：`“是否为所有 API 失败模式定义了错误处理？ [Gap]”`
   - Clarity：`“‘快速加载’ 是否给出了明确的时间阈值？ [Clarity, Spec §NFR-2]”`
   - Consistency：`“各页面的导航要求是否一致？ [Consistency, Spec §FR-10]”`
   - Coverage：`“是否覆盖无数据、部分失败、并发交互等场景？ [Coverage, Edge Case]”`
   - Measurability：`“视觉层级要求是否可以客观量化？ [Measurability, Spec §FR-1]”`

   **场景分类提示**：
   - 检查是否涵盖主要、替代、异常、恢复、非功能等场景。
   - 如果缺失，应新增条目质疑该场景是否被忽略或需补充。
   - 涉及状态变更时，添加韧性/回滚类问题（例如迁移失败如何恢复）。

   **可追溯性要求**：
   - 至少 80% 的条目应包含追溯信息（规格章节、[Gap]、[Assumption] 等）。
   - 若缺少需求/验收标准 ID 体系，应加入 `“是否已建立需求与验收标准的编号体系？ [Traceability]”`。

   **问题归并策略**：
   - 候选条目超过 40 条时，先按风险与影响排序。
   - 合并内容高度相似的条目，避免重复。
   - 低风险边界场景若超过 5 条，可合并成一条总括问题。

   **禁止事项**：
   - ❌ 使用 “Verify/Test/Confirm/Check” 等描述实现行为的句式。
   - ❌ 引用实际代码执行、UI 交互动作。
   - ❌ 使用 “works properly”“functions as expected” 等模糊评价。
   - ❌ 描述实现细节或测试用例。

   **允许的模板**：
   - ✅ “是否为 [scenario] 明确了 [requirement type]？[Completeness]”
   - ✅ “模糊术语 [term] 是否被量化？[Clarity]”
   - ✅ “需求在 [section A] 与 [section B] 是否保持一致？[Consistency]”
   - ✅ “该需求是否可被客观验证？[Measurability]”
   - ✅ “是否覆盖了 [edge case]？[Coverage]”
   - ✅ “规格是否定义了 [missing aspect]？[Gap]”

6. **结构参考**
   - 默认遵循 `.specify/templates/checklist-template.md`。
   - 如模板缺失，可自建结构：H1 标题 + 元数据（Purpose/Created/Feature）+ 各分类的 `##` 标题，每条使用 `- [ ] CHK### ...`。

7. **汇报**
   - 输出新建清单的完整路径与条目数量，并提醒每次运行都会生成新文件。
   - 概述本次聚焦的领域、深度、目标使用者、用户指定的必选项。

**注意**：保持清单目录整洁，命名需可读，完成后记得清理过期文件。

## 示例清单类型

**UX 需求质量（`ux.md`）**  
示例条目：
- “页面上的视觉层级要求是否可量化？ [Clarity, Spec §FR-1]”
- “UI 元素的数量与位置是否明写？ [Completeness, Spec §FR-1]”
- “悬停/聚焦/激活状态的描述是否一致？ [Consistency]”
- “是否覆盖键盘导航的可访问性需求？ [Coverage, Gap]”
- “图片加载失败时是否写出回退方案？ [Edge Case, Gap]”
- “‘Prominent display’ 是否可客观衡量？ [Measurability, Spec §FR-4]”

**API 需求质量（`api.md`）**  
示例条目：
- “是否定义所有失败场景下的错误响应格式？ [Completeness]”
- “限流要求是否写明具体阈值？ [Clarity]”
- “认证需求是否在所有端点保持一致？ [Consistency]”
- “是否明确外部依赖的重试/超时策略？ [Coverage, Gap]”
- “版本策略是否记录在案？ [Gap]”

**性能需求质量（`performance.md`）**  
示例条目：
- “性能指标是否给出具体数值？ [Clarity]”
- “关键用户路径的性能目标是否全量覆盖？ [Coverage]”
- “不同负载场景的性能要求是否写出？ [Completeness]”
- “性能标准是否可被客观验证？ [Measurability]”
- “高负载退化策略是否有文档说明？ [Edge Case, Gap]”

**安全需求质量（`security.md`）**  
示例条目：
- “受保护资源的认证需求是否齐全？ [Coverage]”
- “敏感数据的保护措施是否写明？ [Completeness]”
- “威胁模型是否记录并与需求对齐？ [Traceability]”
- “需求是否符合合规要求？ [Consistency]”
- “安全故障/泄露的响应流程是否有定义？ [Gap, Exception Flow]”

## 反例与正例

**错误示例（测试实现）**
```markdown
- [ ] CHK001 - Verify landing page displays 3 episode cards [Spec §FR-001]
- [ ] CHK002 - Test hover states work correctly on desktop [Spec §FR-003]
- [ ] CHK003 - Confirm logo click navigates to home page [Spec §FR-010]
- [ ] CHK004 - Check that related episodes section shows 3-5 items [Spec §FR-005]
```

**正确示例（检验需求质量）**
```markdown
- [ ] CHK001 - Are the number and layout of featured episodes explicitly specified? [Completeness, Spec §FR-001]
- [ ] CHK002 - Are hover state requirements consistently defined for all interactive elements? [Consistency, Spec §FR-003]
- [ ] CHK003 - Are navigation requirements clear for all clickable brand elements? [Clarity, Spec §FR-010]
- [ ] CHK004 - Is the selection criteria for related episodes documented? [Gap, Spec §FR-005]
- [ ] CHK005 - Are loading state requirements defined for asynchronous episode data? [Gap]
- [ ] CHK006 - Can "visual hierarchy" requirements be objectively measured? [Measurability, Spec §FR-001]
```

**对比总结**
- 错误示例关注“系统是否做到”，正确示例关注“需求是否写清楚”。
- 错误示例描述行为，正确示例检视文本质量。
- 错误示例问 “Does it do X?”，正确示例问 “Is X clearly specified?”。
