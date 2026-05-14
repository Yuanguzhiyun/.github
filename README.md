# Yuanguzhiyun org infra (`.github` repo)

Org-wide GitHub infrastructure SSOT。本次选择 **public** 是为覆盖所有 repo 类型 + 简化访问控制（free org plan 不可用 internal；private alternative `.github-private` 适合需收敛可见性的场景，但需配套权限管理）。本 repo 不含 secrets / 业务代码 / CI workflow，无 fork PR 暴露面。

## `workflow-templates/`

新 workflow 起手优先从这里复制（GitHub UI **Actions → New workflow** 自动列出，分组 "By Yuanguzhiyun"）。所有模板 `runs-on:` 默认 `[self-hosted, racknerd]`。

| 模板 | 用途 | 关键点 |
|---|---|---|
| `secret-scan-base.yml` | 全 git history secret 扫描 | gitleaks MIT binary install（避开 gitleaks-action 商业 license） |
| `go-base.yml` | Go vet + race test | `go-version-file: go.mod`；race OOM 时改 `-parallel 1`（racknerd MemoryMax=1.5G） |
| `python-base.yml` | uv + pytest | setup-uv + `uv sync --all-extras` + `uv run pytest` |

⚠️ 各模板 yml 头部有 **trust boundary 依赖** 注释段，复用前必须确认 consumer repo 满足（private + org fork PR approval + workflow read default + 最小特权 permissions）；不满足请改 ubuntu-hosted 或 ephemeral runner，不要复用本模板。

## Yuanguzhiyun org-scoped CI runtime policy

> **新 workflow `runs-on:` 默认 `[self-hosted, racknerd]`**（runner = `racknerd-org`，`23.95.215.158`，2C/1.9G/3G swap，labels=`self-hosted, Linux, X64, racknerd`）。
>
> 例外场景需在 PR description / spec 显式声明：
> - 需 macOS / Windows / GPU → 对应 GitHub-hosted
> - 需 ephemeral / per-job container 隔离（trust boundary 收紧）
> - 临时 debug 跑 GitHub-hosted 对照 → 合入前必改回 self-hosted
>
> Yuanguzhiyun org 内 LLM session / human PR review 共同 gate 此约定。dev-workflow baseline 提供"新 workflow 优先 owner-defined runner"通用纪律，本 README 提供 Yuanguzhiyun-specific 实例（per dev-workflow agent-rules 分层规则：baseline 通用、owner/project 层挂钩具体）。

## 上下文

- self-hosted runner: `racknerd-org` (`23.95.215.158`，2C/1.9G/3G swap)
- 上游：`ubuntu-maintance` task `011-rackNerdCiRunner`
- runner 资源限位：`MemoryMax=1.5G` / `CPUQuota=180%` / `TasksMax=4096`（systemd drop-in）
- 反向兜底：GitHub 没有 org/repo 级别"default runner"开关，runs-on 由 workflow 文件 declarative 决定；本 repo 模板 + dev-workflow baseline 通用纪律 + Yuanguzhiyun policy（本段）三层叠加推动新 workflow 默认 self-hosted
- reusable workflows（中期演进）：业务 CI 高度同质化时可改 `uses: Yuanguzhiyun/.github/.github/workflows/<base>.yml@main` 让 repo inherit `runs-on`；今天不做，等 5 repo 完成 self-hosted 首轮迁移后评估
