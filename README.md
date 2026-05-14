# Yuanguzhiyun org infra (`.github` repo)

Org-wide GitHub infrastructure SSOT。本 repo **public** 是 GitHub 设计要求（org workflow templates / org README 必须 public 才能被其他 repo 看到）；不含 secrets / 业务代码 / CI workflow，无 fork PR 暴露面。

## `workflow-templates/`

新 workflow 起手默认从这里复制（GitHub UI **Actions → New workflow** 自动列出）。所有模板 `runs-on:` 默认 `[self-hosted, racknerd]`。

| 模板 | 用途 | 关键点 |
|---|---|---|
| `secret-scan-base.yml` | 全 git history secret 扫描 | gitleaks MIT binary install（避开 gitleaks-action 商业 license） |
| `go-base.yml` | Go vet + race test | `go-version-file: go.mod`；race OOM 时改 `-parallel 1`（racknerd MemoryMax=1.5G） |
| `python-base.yml` | uv + pytest | setup-uv + `uv sync --all-extras` + `uv run pytest` |

## 上下文

- self-hosted runner: `racknerd-org` (`23.95.215.158`，2C/1.9G/3G swap)
- 上游：`ubuntu-maintance` task `011-rackNerdCiRunner`
- runner 资源限位：`MemoryMax=1.5G` / `CPUQuota=180%` / `TasksMax=4096`（systemd drop-in）
- GitHub 没有"org default runner"开关，runs-on 由 workflow 文件 declarative；本 repo 模板 + agent-rules 软约定共同推动新 workflow 默认 self-hosted
