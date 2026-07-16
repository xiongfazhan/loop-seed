# Claude Project Instructions

## Project Context

本项目是 Tax AI 执行中心。技术栈、目录结构与检查命令见 `.claude/CHECKS.md`。

## Loop Engineering

本项目使用 Claude Code 的 bounded loop 工作流调度代码修改，四个角色构成 计划 → 实现 → 验收 → 反馈 的循环：

- `planner`：只负责方案设计与修订，不写代码。
- `builder`：只负责按方案实现和修复代码。
- `checker`：只负责运行检查并报告失败，不允许修改文件。
- `analyst`：只负责分析失败，给出修实现 / 改方案 / 停止升级的结论，不允许修改文件。
- `/pdca`：负责调度以上角色，机械执行停止规则。（不叫 `/loop` 是为了避开 Claude Code 的同名内置命令）

角色分工、停止条件、红线、升级协议等通用规则见 `.claude/LOOP_RULES.md`——该文件与技术栈无关，可原样复用到其他项目。
本项目专属的技术栈、检查命令见 `.claude/CHECKS.md`——复用这套 loop 框架到新项目时，只需要重写这一个文件和上面的 Project Context。

轮次上限：默认 5 轮（`.claude/LOOP_RULES.md`），本项目未覆盖默认值。
