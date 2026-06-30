---
description: 按照 tasks.md 中定义的任务执行实施计划。
---


## 用户输入

```text
$ARGUMENTS
```

在继续前，若输入非空，必须先审视用户输入。

## 概览

1. 从仓库根目录运行 `.specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks`，解析 FEATURE_DIR 与 AVAILABLE_DOCS。路径必须为绝对路径。当参数含有单引号（如 "I'm Groot"）时，请使用 `'I'\''m Groot'` 或 `"I'm Groot"`。

2. **检查检查清单状态**（若存在 `FEATURE_DIR/checklists/`）：
   - 扫描 `checklists/` 中所有文件
   - 对每个清单统计：
     * 总条目：匹配 `- [ ]`、`- [X]` 或 `- [x]` 的行
     * 已完成条目：匹配 `- [X]` 或 `- [x]` 的行
     * 未完成条目：匹配 `- [ ]` 的行
   - 生成状态表：
     ```
     | Checklist | Total | Completed | Incomplete | Status |
     |-----------|-------|-----------|------------|--------|
     | ux.md     | 12    | 12        | 0          | ✓ PASS |
     | test.md   | 8     | 5         | 3          | ✗ FAIL |
     | security.md | 6   | 6         | 0          | ✓ PASS |
     ```
   - 计算总体状态：
     * **PASS**：所有清单的未完成条目数为 0
     * **FAIL**：任一清单存在未完成条目
   
   - **若存在未完成的检查清单**：
     * 输出表格并列出未完成数量
     * **停止执行** 并询问：“Some checklists are incomplete. Do you want to proceed with implementation anyway? (yes/no)”
     * 等待用户答复；如回答 “no”“wait”“stop” 等，终止流程
     * 若回答 “yes”“proceed”“continue” 等，则继续第 3 步
   
   - **若所有清单均已完成**：
     * 输出全部通过的表格
     * 自动进入第 3 步

3. 加载并分析实施上下文：
   - **必读**：`tasks.md`（完整任务与执行计划）
   - **必读**：`plan.md`（技术栈、架构、文件结构）
   - **若存在**：`data-model.md`（实体及关系）
   - **若存在**：`contracts/`（API 规格与测试要求）
   - **若存在**：`research.md`（技术决策与约束）
   - **若存在**：`quickstart.md`（集成场景）

4. **项目环境校验**：
   - **必须**：依据实际项目设置创建/校验忽略文件：
   
   **检测与创建逻辑**：
   - 运行：

     ```sh
     git rev-parse --git-dir 2>/dev/null
     ```

     判断是否为 git 仓库，若是则创建/校验 `.gitignore`
   - 存在 Dockerfile* 或 plan.md 中声明 Docker → 创建/校验 `.dockerignore`
   - 存在 `.eslintrc*` 或 `eslint.config.*` → 创建/校验 `.eslintignore`
  - 存在 `.prettierrc*` → 创建/校验 `.prettierignore`
   - 存在 `.npmrc` 或 `package.json` → 在需要发布时创建/校验 `.npmignore`
   - 存在 Terraform 文件（*.tf）→ 创建/校验 `.terraformignore`
   - 存在 Helm Chart → 创建/校验 `.helmignore`
   
   **若忽略文件已存在**：确认包含关键模式，仅补充缺失的核心条目。
   
   **若忽略文件缺失**：依据检测到的技术栈完整创建。
   
   **按技术栈常见模式**（参考 plan.md）：
   - **Node.js/JavaScript**：`node_modules/`、`dist/`、`build/`、`*.log`、`.env*`
   - **Python**：`__pycache__/`、`*.pyc`、`.venv/`、`venv/`、`dist/`、`*.egg-info/`
   - **Java**：`target/`、`*.class`、`*.jar`、`.gradle/`、`build/`
   - **C#/.NET**：`bin/`、`obj/`、`*.user`、`*.suo`、`packages/`
   - **Go**：`*.exe`、`*.test`、`vendor/`、`*.out`
   - **Ruby**：`.bundle/`、`log/`、`tmp/`、`*.gem`、`vendor/bundle/`
   - **PHP**：`vendor/`、`*.log`、`*.cache`、`*.env`
   - **Rust**：`target/`、`debug/`、`release/`、`*.rs.bk`、`*.rlib`、`*.prof*`、`.idea/`、`*.log`、`.env*`
   - **Kotlin**：`build/`、`out/`、`.gradle/`、`.idea/`、`*.class`、`*.jar`、`*.iml`、`*.log`、`.env*`
   - **C++**：`build/`、`bin/`、`obj/`、`out/`、`*.o`、`*.so`、`*.a`、`*.exe`、`*.dll`、`.idea/`、`*.log`、`.env*`
   - **C**：`build/`、`bin/`、`obj/`、`out/`、`*.o`、`*.a`、`*.so`、`*.exe`、`Makefile`、`config.log`、`.idea/`、`*.log`、`.env*`
   - **通用**：`.DS_Store`、`Thumbs.db`、`*.tmp`、`*.swp`、`.vscode/`、`.idea/`
   
   **工具专属模式**：
   - **Docker**：`node_modules/`、`.git/`、`Dockerfile*`、`.dockerignore`、`*.log*`、`.env*`、`coverage/`
   - **ESLint**：`node_modules/`、`dist/`、`build/`、`coverage/`、`*.min.js`
   - **Prettier**：`node_modules/`、`dist/`、`build/`、`coverage/`、`package-lock.json`、`yarn.lock`、`pnpm-lock.yaml`
   - **Terraform**：`.terraform/`、`*.tfstate*`、`*.tfvars`、`.terraform.lock.hcl`

5. 解析 `tasks.md` 结构，提取：
   - **任务阶段**：Setup、Tests、Core、Integration、Polish
   - **任务依赖**：串行与并行执行规则
   - **任务详情**：ID、描述、文件路径、并行标记 [P]
   - **执行流程**：顺序与依赖要求

6. 按任务计划执行实施：
   - **按阶段顺序**：完成一个阶段后再进入下一个
   - **遵循依赖关系**：串行任务按顺序执行，可并行的 [P] 任务可同时进行
   - **遵循 TDD 思路**：在相应实现任务前先完成测试任务
   - **基于文件协调**：作用于同一文件的任务需串行执行
   - **阶段校验**：每个阶段完成后确认状态再继续

7. 实施规则：
   - **Setup 先行**：初始化项目结构、依赖、配置
   - **先测后码**：涉及契约、实体、集成场景时先编写测试
   - **核心开发**：实现模型、服务、CLI 命令、端点等
   - **集成工作**：数据库连接、中间件、日志、外部服务
   - **完善与验证**：单元测试、性能优化、文档补充

8. 进度追踪与错误处理：
   - 每完成一个任务汇报进度
   - 串行任务失败时立即暂停
   - 并行任务 [P]：可继续执行成功任务并汇报失败项
   - 提供包含上下文的明确错误信息
   - 无法继续实施时给出下一步建议
   - **重要**：完成的任务务必在 tasks 文件中改为 `[X]`

9. 完成校验：
   - 确认所有必需任务已完成
   - 检查实现是否符合原始规格
   - 验证测试通过且覆盖率满足要求
   - 确认实施遵循技术计划
   - 汇报最终状态并总结完成工作

注意：此命令假设 `tasks.md` 已提供完整的任务拆解。若任务缺失或不完整，请先运行 `/tasks` 重新生成。
