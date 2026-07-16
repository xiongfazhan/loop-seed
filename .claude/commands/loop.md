---
description: Run builder and checker in a bounded loop until checks pass or stop rules trigger.
argument-hint: <task>
allowed-tools: Read, Glob, Grep, Bash, Task
model: sonnet
---

以循环方式执行此任务：

`$ARGUMENTS`

## 第 0 步：对齐目标

先写一行任务简报，必须包含：

- 目标
- 预计涉及区域
- 完成标准
- 要运行的检查

这行简报要原样传给 builder 和 checker。

## 循环流程

最多运行 5 轮。每轮开始时公开声明：

```text
Cycle N/5
```

每一轮按顺序执行：

1. 派 `builder` 实现任务，或修复上一轮 checker 报告的失败。
2. 派 `checker` 运行与改动相关的全部检查。
3. 如果 checker 输出 `ALL GREEN`：停止循环，展示 diff 摘要和逐项检查结果。
4. 如果 checker 输出 `FAILED`：把 checker 的完整失败报告原样转发给 builder，不要解读、过滤、改写、压缩。
5. 回到第 1 步。

## 必须遵守的停止规则

严格遵循根目录 `CLAUDE.md` 的 `Loop stop rules`。

尤其注意：

- 同一失败连续出现两轮就停止。
- 修复导致之前通过的检查失败就停止。
- 连续两轮失败项数量没有减少就停止。
- 遇到外部服务、数据库、凭证、浏览器环境等不可访问问题，停止并升级。

## 停止后的输出

停止时必须输出：

```text
当前轮次：
停止原因：
仍失败的项：
已尝试修复：
下一步建议：
```

只有 checker 明确输出 `ALL GREEN`，才能报告成功。
