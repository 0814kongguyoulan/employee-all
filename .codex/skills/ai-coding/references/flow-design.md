# Employee All AI-CODING 数据流

该流程以任务目录作为可恢复状态，不依赖对话记忆：

```text
用户需求 / Grill 结论
        |
        v
requirements.md
        |
        v
impact-analysis.md -> design.md -> implementation-plan.md
                                      |
                                      v
                              coding-tasks.md
                                      |
                         Task 实施、验证、Review
                                      |
             changed-files.txt + code-review/TASK-xxx.md
                                      |
                                      v
              final-review.md + release-checklist-candidates.md
```

`progress.md` 是唯一阶段状态入口。恢复任务时，先读取它，再按需读取当前阶段依赖的产物，避免一次加载全部文档。

## 阶段依赖

| 阶段 | 必需输入 | 主要输出 |
| --- | --- | --- |
| 需求与边界确认 | 用户需求、Grill 结论 | requirements.md |
| 代码影响分析 | requirements.md、项目代码 | impact-analysis.md |
| 技术设计 | requirements.md、impact-analysis.md | design.md |
| 实施计划 | design.md、package.json、pom.xml | implementation-plan.md |
| Task 拆解 | 设计与计划 | coding-tasks.md、task-review.md |
| 实施 | 当前 Task、相关代码 | 代码、changed-files.txt |
| 验证 | 前后端构建配置、验收条件 | 验证记录 |
| Review | diff、Task Review、验证结果 | final-review.md |
| 发布检查 | 实际变更、Review | release-checklist-candidates.md |

## 边界

- 生命周期文档位于 `employee-all/docs/`，代码范围包括 `employee-web/` 和 `spring-boot-demo/`。
- 跨端变更必须在设计中明确 API 契约、数据流、错误处理和兼容方式。
- `changed-files.txt` 中的路径相对 `employee-all` 根目录记录，例如 `employee-web/src/App.vue` 或 `spring-boot-demo/src/main/java/...`。
- 外部任务系统、Git commit 和 push 都需要用户明确授权。
