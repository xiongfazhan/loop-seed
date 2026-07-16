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

最多运行 `.claude/LOOP_RULES.md` 中定义的轮次上限（默认 5 轮，`CLAUDE.md` 可覆盖）。每轮开始时公开声明：

```text
Cycle N/上限
```

每一轮按顺序执行：

1. 派 `builder` 实现任务，或修复上一轮 checker 报告的失败。
2. 派 `checker` 运行与改动相关的全部检查。
3. 如果 checker 输出 `ALL GREEN`：停止循环，展示 diff 摘要和逐项检查结果。
4. 如果 checker 输出 `BLOCKED`：立即停止循环并升级，不进入下一轮重试，不计入 `FAILED` 轮次计数。
5. 如果 checker 输出 `FAILED`：把 checker 的完整失败报告原样转发给 builder，不要解读、过滤、改写、压缩。
6. 回到第 1 步。

## 必须遵守的停止规则

严格遵循 `.claude/LOOP_RULES.md`。尤其注意：

- 同一失败连续两轮出现就停止（按 LOOP_RULES.md 的归一化判定标准比较，不比较原始报错文本）。
- 修复导致之前通过的检查失败就停止。
- 连续两轮失败项数量没有减少就停止。
- 出现 `BLOCKED` 立即停止并升级。

## 停止后的输出

停止时必须输出：

```text
当前轮次：
停止原因：
仍失败/阻塞的项：
已尝试修复：
下一步建议：
```

只有 checker 明确输出 `ALL GREEN`，才能报告成功。
