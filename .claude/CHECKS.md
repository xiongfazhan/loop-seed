# 项目检查清单（项目专属，非通用部分）

本文件是本项目的检查命令来源，`builder` 与 `checker` 都从这里发现要跑什么，而不是在各自的 prompt 里硬编码。
复用 `.claude/LOOP_RULES.md` 那套通用框架到新项目时，只需要重写这一个文件（以及 `CLAUDE.md` 的 Project Context）。

## 技术栈与目录

- 后端：`backend/`，Python Flask，数据库与业务路由较多。
- 前端：`frontend/`，Vue 3 + Vite + Element Plus + Vitest。
- 测试：根目录 `tests/unit`、`tests/integration`、`tests/e2e`，以及历史后端测试 `backend/tests`。
- CI/CD：`.github/workflows/cicd.yml` 主要负责 Docker build、push 和 Kubernetes 部署，不等同于完整测试流水线。

## 检查命令发现顺序

checker 找命令时按以下顺序读取，不要假设命令存在：

1. `CLAUDE.md` 和 `AGENTS.md`（如果存在）。
2. `README.md`。
3. 根目录 `pytest.ini`、`tests/pytest.ini`、`tests/Makefile`。
4. `frontend/package.json`、`frontend/vitest.config.js`、`backend/requirements.txt`。
5. `.github/workflows/cicd.yml` 中的 Docker build/deploy 命令（注意：当前 CI 主要构建并部署镜像，不等同于完整本地测试）。

## 默认检查顺序（按改动范围选择最小但足够的检查集）

1. **后端业务代码、路由、数据库工具、迁移脚本**：优先运行 `python -m pytest tests/unit tests/integration -m "not slow"`。如果改动触及 `backend/tests/` 覆盖的旧测试或后端历史模块，追加运行 `python -m pytest backend/tests -m "not slow"`。
2. **前端 Vue/Vite 代码**：在 `frontend/` 下运行 `npm run test:unit`，再运行 `npm run build`。
3. **跨前后端用户流、登录、录音、任务流或可视化行为**：在服务可用时运行 `npm run test:e2e` 或 `tests/e2e` 对应 Playwright 命令。
4. **Dockerfile、nginx、部署脚本、CI 配置**：检查对应 Docker build 或 YAML 语法；不要触碰生产发布、集群、registry 或 kubeconfig。
5. **Claude 配置或文档**：检查文件存在、frontmatter 完整、角色工具权限符合分工，必要时运行 `git diff --check`。

如果命令缺依赖、缺服务、缺环境变量、缺数据库或浏览器环境不可用，按 `.claude/LOOP_RULES.md` 的 BLOCKED 格式报告，不要自行安装依赖、启动生产资源或修改配置，除非任务明确要求。
