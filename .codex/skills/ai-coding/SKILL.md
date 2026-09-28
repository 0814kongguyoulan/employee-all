---
name: ai-coding
description: 管理 employee-all 员工系统前后端需求的文件驱动开发生命周期，包括恢复进度、影响分析、技术设计、任务拆解、Vue 与 Spring Boot 编码、构建验证和 Review。用于明确要求启动或继续 AI-CODING 流程，或已有 docs/ai-coding 进度文件的任务；不用于普通问答和无需生命周期管理的小改动。
---

# Employee All AI Coding

通过文件保存需求开发状态，使任务可以跨对话恢复。Skill 从 `employee-all` 根目录运行，覆盖 `employee-web/` 前端和 `spring-boot-demo/` 后端。流程不依赖 Jira、专属 subagent 或外部 MCP。

## 入口与恢复

1. 确定 `task-id`。优先使用用户给出的任务号；否则使用简短的小写连字符标识。
2. 工作目录为 `docs/ai-coding/{task-id}/`。
3. 查找 `docs/grill/{task-id}/task-effort-estimate.md`。存在时读取其中已确认范围、范围外、假设和待确认问题；不存在时，仅在需求存在会影响设计的歧义时调用 `$task-grill-me`。
4. 如果已有 `progress.md`，读取它和其中列出的输入文件，从第一个未完成阶段恢复。
5. 如果没有 `progress.md`，根据 [references/progress-template.md](references/progress-template.md) 创建。

不要移动或覆盖 Grill 文档；在 `progress.md` 中记录引用路径即可。

## 生命周期

按依赖顺序执行，并在完成后只把对应 checkbox 从 `[ ]` 改成 `[x]`：

1. 需求与边界确认
2. 代码影响分析
3. 技术设计
4. 实施计划
5. Coding Tasks 拆解与 Review
6. 按 Task 实施
7. 验证
8. 整体 Review
9. 发布检查
10. Git 提交（可选）
11. 知识归档

设计存在重大取舍、范围发生变化、需要外部系统写入或准备 Git 提交时，必须停下向用户确认。普通的代码读取、实现和本地验证不需要重复确认。

## 阶段产物

所有产物写入 `docs/ai-coding/{task-id}/`：

- `requirements.md`：目标、范围、范围外、验收条件、来源和未决问题。
- `impact-analysis.md`：前端组件与状态、后端分层与 API、数据库、缓存、配置和构建影响，附代码路径证据。
- `design.md`：数据流、组件职责、接口契约、状态与错误处理、响应式行为。
- `implementation-plan.md`：按文件和依赖顺序说明实现与验证步骤。
- `api-cases.md`：仅在接口新增或变化时生成，使用 [references/api-cases-template.md](references/api-cases-template.md)。
- `coding-tasks.md`：可独立实施、Review、验证的任务。
- `task-review.md`：任务覆盖性、边界、依赖和风险 Review。
- `changed-files.txt`：实际修改的源码/配置文件，相对 `employee-all` 根目录，一行一个并去重，例如 `employee-web/src/App.vue` 或 `spring-boot-demo/src/main/java/...`。
- `code-review/TASK-xxx.md`：每个任务的 Review 结论。
- `release-checklist-candidates.md`：使用 [references/release-checklist-candidates-template.md](references/release-checklist-candidates-template.md)。
- `final-review.md`：整体回归风险、验证结果和残余风险。

更完整的数据流说明见 [references/flow-design.md](references/flow-design.md)。

## Coding Task 规则

每个 Task 必须包含：状态、目标、输入依据、候选文件、依赖、期望改动、不允许做的事、验收条件、最小验证、风险和实际变更文件。

状态使用：`pending`、`coding`、`reviewing`、`needs-fix`、`needs-human`、`passed`、`skipped`。

执行循环：

1. 选择依赖已满足的第一个 `pending` Task，标记为 `coding`。
2. 只实现当前 Task 范围，并实时维护 `changed-files.txt`。
3. 完成后标记为 `reviewing`，生成对应 `code-review/TASK-xxx.md`。
4. Review 通过后标记 `passed`；发现问题则标记 `needs-fix` 并修复；需要用户决策则标记 `needs-human` 并暂停。
5. 当前 Task 未 `passed` 或确认 `skipped` 前，不进入下一个 Task。

## 项目实施约束

- 前端先读取 `employee-web/package.json`、`employee-web/vite.config.js`、现有组件和样式，沿用项目模式，不引入无必要依赖。
- 后端先读取 `spring-boot-demo/pom.xml`、相关 Controller、Service、Repository、Entity、配置和测试，沿用现有 Spring Boot 分层与 JPA 模式。
- API 契约发生变化时，同步更新调用方，并处理 loading、empty、error、校验和异常响应。
- 涉及 MySQL 表结构、初始化数据或 Redis 缓存时，明确迁移/初始化方式、缓存键、TTL 和失效策略。
- 保持桌面和移动端布局可用，使用项目已有图标库。
- 不擅自创建分支、提交或推送。

## 验证

按实际影响范围验证：

- 前端从 `employee-web/package.json` 读取实际脚本，并在 `employee-web/` 运行适用的 build、lint、test 命令；前端改动至少运行生产构建。
- 后端从 `spring-boot-demo/pom.xml` 和 Maven Wrapper 读取实际配置，并在 `spring-boot-demo/` 运行适用的测试或构建命令。
- 跨端功能验证接口契约、前端操作和数据持久化；涉及 Redis 时同时验证缓存命中及写操作后的失效行为。
- 对用户可见变化，在运行中的本地页面验证关键交互；涉及响应式布局时检查桌面和移动端。
- 记录实际执行结果、未执行项和残余风险。失败时先修复属于本次变更的问题。

## 发布与 Git

发布候选根据实际变更覆盖：环境变量、API 契约、前后端依赖、构建与启动命令、MySQL DDL/数据、Redis 缓存、静态资源、兼容性和人工验证。不生成与本项目无关的 Mongo、Job 或 Apollo 项。

Git 提交是可选阶段：只有用户明确要求提交时才执行。提交前核对 `changed-files.txt`、实际 diff 和 Review 结果；推送远程需要用户请求或已有明确授权。

## 完成条件

- 所有必需阶段已标记完成。
- 所有已执行 Task 均为 `passed`，对应 Review 存在且通过。
- 构建和适用验证通过，或明确记录阻塞与残余风险。
- `changed-files.txt` 与实际修改一致。
- 最终回复包含产物目录、功能结果、验证结果和未解决事项。
