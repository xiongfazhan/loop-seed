# loop-seed

通用的 **bounded loop 工程框架**，用于 Claude Code：把一次代码修改拆成四个互不信任的角色，防止 AI "自己改代码、自己说通过"。

> 📖 **完整图文教程（含零基础入门、工作流程图）**：[`docs/pdca-guide.html`](docs/pdca-guide.html)
> （GitHub 不渲染 HTML，请下载后用浏览器打开，或在 Claude Code 里查看）

## 这是什么

让 AI 改代码最大的坑是它可能"自说自话"——改完直接说"测试通过"，但可能根本没跑，甚至偷偷删了测试来凑绿。这套机制把一次修改拆成四个角色，权限互相制约：

| 角色 | 阶段 | 职责 | 能改文件吗 |
|---|---|---|---|
| `planner` | Plan · 方案 | 先出方案：改哪、改成什么样、怎么算完工 | 否（只读） |
| `builder` | Do · 实现 | 只按方案写代码，不擅自扩大范围 | ✅ 是 |
| `checker` | Check · 验收 | 只跑检查、只报结果，报错原文照抄 | **否（工具白名单里就没有写文件权限）** |
| `analyst` | Act · 反馈 | 判断失败该返工、改方案还是叫停 | 否（只读） |

关键在于 **checker 物理上无法改代码**，所以"测试通过"不可能是编出来的。**"有界"** 指最多来回 5 轮就必须停，触发"越改越糟""连续两轮同一个错"等信号会强制刹车并把情况交还给你。

## 快速开始

在 Claude Code 会话里：

```
/pdca 修复登录接口在密码含中文时返回 500 的问题
```

调度器会自动跑 planner → builder → checker → analyst 循环，直到 `ALL GREEN` 或触发停止规则。

> ⚠️ 命令是 `/pdca` 不是 `/loop`——`/loop` 是 Claude Code 的内置定时循环命令，与本框架无关，本框架特意改名避开撞名。

## 装到你自己的项目

框架层原样复制，**项目层必须改写**：

| 文件 | 层 | 搬家时 |
|---|---|---|
| `.claude/agents/`、`.claude/commands/pdca.md`、`.claude/LOOP_RULES.md` | 框架 | 原样复制，零修改 |
| `.claude/CHECKS.md` | 项目 | **必须重写**为你项目的技术栈和检查命令 |
| `CLAUDE.md`（Project Context） | 项目 | **必须重写**为你项目的背景 |

> ⚠️ **CHECKS.md 不改会直接失效**：现在它写的是 pytest / npm 命令，原样搬到别的技术栈，checker 找不到命令会直接报 `BLOCKED`，循环跑起来但验收是空的。checker 和 builder 都从这个文件"发现"该跑什么检查。

详细步骤见 [`docs/pdca-guide.html`](docs/pdca-guide.html) 的 §07。

## 目录结构

```
loop-seed/
├── CLAUDE.md                  # 项目背景 + 指向下面的规则文件（项目层）
├── README.md                  # 本文件
├── docs/
│   └── pdca-guide.html        # 完整图文教程
└── .claude/
    ├── LOOP_RULES.md          # 通用规则：角色分工、停止条件、红线、升级协议（框架层）
    ├── CHECKS.md              # 本项目的技术栈与检查命令（项目层，搬家必改）
    ├── agents/
    │   ├── planner.md         # Plan  · 方案设计
    │   ├── builder.md         # Do    · 实现代码
    │   ├── checker.md         # Check · 运行检查
    │   └── analyst.md         # Act   · 分析反馈
    └── commands/
        └── pdca.md            # /pdca 命令：调度上述四角色
```

## 停止规则

循环由调度器机械执行以下停止条件，`analyst` 只能建议提前停、不能让循环多跑一轮：

1. `ALL GREEN`（唯一成功出口）
2. 达到轮次上限（默认 5 轮）
3. 同一失败连续两轮出现
4. 出现回归（之前通过的检查后来失败）
5. 连续两轮失败项数量没有减少
6. `BLOCKED`（缺依赖/服务/凭证/环境）
7. 范围失控（需架构、schema、认证或生产部署级变更）
8. `REPLAN` 超过 1 次

完整定义见 [`.claude/LOOP_RULES.md`](.claude/LOOP_RULES.md)。
