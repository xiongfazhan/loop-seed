# Claude Project Instructions

## Project Context

本项目是 Tax AI 执行中心，主要包含：

- 后端：`backend/`，Python Flask，数据库与业务路由较多。
- 前端：`frontend/`，Vue 3 + Vite + Element Plus + Vitest。
- 测试：根目录 `tests/unit`、`tests/integration`、`tests/e2e`，以及历史后端测试 `backend/tests`。
- CI/CD：`.github/workflows/cicd.yml` 主要负责 Docker build、push 和 Kubernetes 部署，不等同于完整测试流水线。

## Loop Engineering Roles

本项目支持 Claude Code 的 bounded loop 工作流：

- `builder`：只负责实现和修复代码。
- `checker`：只负责运行检查并报告失败，不允许修改文件。
- `/loop`：负责调度 `builder` 和 `checker` 循环。

## Loop stop rules

### 停止条件

循环在以下任一条件成立时必须停止：

1. `ALL GREEN`：checker 明确确认所有相关检查通过，并附上每项检查的通过证明。
2. 轮次用尽：达到 5 轮上限。
3. 同一失败连续两轮出现：builder 可能在猜测而不是修复。
4. 出现回归：某项之前通过的检查在后续轮次失败。
5. 无实质进展：连续 2 轮失败项数量没有减少。
6. 超出能力边界：失败依赖无法访问的外部服务、数据库、凭证、浏览器环境或生产资源。
7. 任务范围失控：修复需要大规模架构、数据库 schema、认证授权或生产部署变更。

### 红线

- 永远不在没有 checker 输出的情况下报告成功。
- 永远不弱化、删除、跳过测试或检查来达到 `ALL GREEN`。
- 永远不修改 checker 的工具白名单来让它能写文件。
- 永远不把格式化、重构或顺手修复混入无关任务。
- 永远不提交密钥、token、密码、真实客户数据或生产凭证。
- 永远不自动执行生产部署、集群变更、registry push 或 kubeconfig 操作。

### 升级协议

停止并升级给用户时，必须携带以下信息：

- 当前轮次：`Cycle N/5`
- 停止原因
- 仍失败的项列表
- 每项已尝试过的修复方法
- 判断：为什么继续循环不会解决问题
- 建议：下一步应拆分任务、补依赖、补测试，还是人工确认需求

## Repository Checks

根据改动范围选择检查：

- 后端主要测试：`python -m pytest tests/unit tests/integration -m "not slow"`
- 后端历史测试：相关模块涉及 `backend/tests` 时追加 `python -m pytest backend/tests -m "not slow"`
- 前端 Vue/Vite：在 `frontend/` 下运行 `npm run test:unit` 和 `npm run build`
- E2E/UI：在服务可用时运行 `npm run test:e2e` 或 `tests/e2e` 对应 Playwright 命令
- Docker/CI：只做本地构建或语法检查，不执行生产发布、push、kubectl apply
- 配置/文档：检查文件存在、frontmatter 完整、角色权限符合分工，并运行 `git diff --check`

如果检查因为缺少依赖、数据库、环境变量或外部服务而无法运行，必须如实报告阻塞，不得伪造通过。
