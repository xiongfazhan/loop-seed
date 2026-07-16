# Claude Project Instructions

## Project Context

本项目是 Tax AI 执行中心。技术栈、目录结构与检查命令见 `.claude/CHECKS.md`。

## Loop Engineering

本项目使用 Claude Code 的 bounded loop 工作流调度代码修改：

- `builder`：只负责实现和修复代码。
- `checker`：只负责运行检查并报告失败，不允许修改文件。
- `/loop`：负责调度 `builder` 和 `checker` 循环。

角色分工、停止条件、红线、升级协议等通用规则见 `.claude/LOOP_RULES.md`——该文件与技术栈无关，可原样复用到其他项目。
本项目专属的技术栈、检查命令见 `.claude/CHECKS.md`——复用这套 loop 框架到新项目时，只需要重写这一个文件和上面的 Project Context。

轮次上限：默认 5 轮（`.claude/LOOP_RULES.md`），本项目未覆盖默认值。
