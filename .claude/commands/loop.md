---
description: Run planner, builder, checker and analyst in a bounded loop until checks pass or stop rules trigger.
argument-hint: <task>
allowed-tools: Read, Glob, Grep, Bash, Task
---

以循环方式执行此任务：

`$ARGUMENTS`

## 第 0 步：对齐目标

先写一行任务简报，必须包含：

- 目标
- 预计涉及区域
- 完成标准
- 要运行的检查

## 第 1 步：方案

派 `planner` 基于任务简报输出方案。方案要原样传给后续每个角色。

## 循环流程

最多运行 `.claude/LOOP_RULES.md` 中定义的轮次上限（默认 5 轮，`CLAUDE.md` 可覆盖）。每轮开始时公开声明：

```text
Cycle N/上限
```

每一轮按顺序执行：

1. 派 `builder` 按方案实现，或按 analyst 的修复指引 + checker 原始报告修复。
2. 派 `checker` 运行与改动相关的全部检查。
3. 如果 checker 输出 `ALL GREEN`：停止循环，展示 diff 摘要和逐项检查结果。
4. 如果 checker 输出 `BLOCKED`：立即停止循环并升级，不进入下一轮重试，不计入 `FAILED` 轮次计数。
5. 如果 checker 输出 `FAILED`：把方案、builder 汇报、checker 完整报告三份材料交给 `analyst`。checker 报告必须原样传递，不要解读、过滤、改写、压缩。
6. 按 analyst 的结论分派：
   - `FIX`：把 analyst 修复指引 + checker 原始报告交给 builder，回到第 1 步。
   - `REPLAN`：把 analyst 分析 + checker 原始报告交给 planner 出修订方案，然后回到第 1 步。整个任务最多重新规划 1 次，analyst 第二次给出 `REPLAN` 时视为任务范围失控，停止并升级。
   - `STOP`：按升级协议停止。

## 必须遵守的停止规则

严格遵循 `.claude/LOOP_RULES.md`。停止规则由本命令机械执行，analyst 的结论不能覆盖它们：

- 同一失败连续两轮出现就停止（按 LOOP_RULES.md 的归一化判定标准比较，不比较原始报错文本）。
- 修复导致之前通过的检查失败就停止。
- 连续两轮失败项数量没有减少就停止。
- 出现 `BLOCKED` 立即停止并升级。
- REPLAN 超过 1 次即停止并升级。
- analyst 可以建议提前停止（`STOP`），但无论 analyst 说什么，以上条件命中就必须停。

## 停止后的输出

停止时必须输出：

```text
当前轮次：
停止原因：
方案版本：<v1 / v2，及是否发生过 REPLAN>
仍失败/阻塞的项：
已尝试修复：
下一步建议：
```

只有 checker 明确输出 `ALL GREEN`，才能报告成功。
