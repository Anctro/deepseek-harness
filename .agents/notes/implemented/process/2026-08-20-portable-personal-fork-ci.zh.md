# Agent Note: 个人 fork 的可移植 CI

Status: implemented

[English](2026-08-20-portable-personal-fork-ci.md) | 中文

## 问题

官方仓库使用组织范围的大型 runner、私有自托管备用池、GitHub App 和发布凭据。个人 fork 会继承这些 workflow，却无法访问相应资源。Pull Request 任务会在不可用的 runner 标签上持续排队；没有 `DEEPSEEK_API_KEY_EXTERNAL` 时真实 API 任务会失败；没有 App 凭据时 Issue 生命周期自动化会失败；手动触发的 fork 发布任务也绝不能向官方 npm 或 PyPI 命名空间发布。

## 决策

现有 workflow 继续作为 CI 和文档部署的唯一事实来源。当 `github.repository` 不是 `deepseek-ai/deepseek-harness` 时，Pull Request 任务选择 GitHub 托管的标准 Linux 和 Windows runner；官方仓库继续使用大型 runner 与故障切换选择器。自托管备用演练与组织 runner benchmark 只在官方仓库执行。

覆盖率与快照/产物 lane 在 fork 中仍会执行，但只提供 advisory（参考性）证据，因为 GitHub 托管镜像暴露的可选工具和浏览器环境与官方 runner 池不同。fork 的稳定 verdict（裁决）仍由静态检查、受支持 Node 兼容性、Python SDK/runtime 打包、Windows Wine 和包演练阻断。昂贵且仅在 master 上运行的内核沙箱矩阵仍是官方仓库参考；fork 不会消耗 macOS 与 Arm runner 时间来复现这项基础设施证明。

真实 API E2E 默认只在官方仓库运行。fork 必须设置仓库变量 `DSH_REAL_API_E2E_ENABLED=true` 并配置 `DEEPSEEK_API_KEY_EXTERNAL`，任务才可运行。workflow 继续使用 `pull_request`，绝不使用 `pull_request_target`，因此来自其他 fork 的 Pull Request 无法取得仓库 secret。

每个 npm 和 PyPI 发布任务除要求手动发布输入外，还要求 `github.repository == 'deepseek-ai/deepseek-harness'`。fork 可以保留无密钥的打包和产物演练，但无法进入官方发布任务。Issue 生命周期自动化同样带有官方仓库保护，因为其 GitHub App 安装只属于该仓库。可移植的 Pages workflow 继续作为 fork 的部署路径。

## 考虑过的替代方案

**新增一套 fork 专用 workflow。** 不采用，因为它会复制仓库门禁清单，并随上游 CI 变化而漂移。根据仓库选择 runner 可以保留现有检查和稳定的 `all checks passed` 结论。

**禁用全部继承的 workflow。** 不采用，因为 fork 会失去原本无需私有凭据即可运行的覆盖率、构建、快照、打包、Python 和文档证据。

**允许配置同名 secret 的 fork 执行发布。** 不采用，因为包名和可信发布者身份属于官方命名空间。下游发行版需要自己的包名和单独的发布决策。

## 后果

个人 fork 无需访问 DeepSeek 基础设施，即可在 GitHub 托管 runner 上运行可移植的 Pull Request verdict，并通过 GitHub Pages 部署文档。参考性覆盖率和快照 lane 仍会暴露上游漂移，但官方 runner 假设不会成为 fork 合并要求。真实 Provider 测试仍为显式启用且由 secret 支撑。除非下游有意用自己的设计替换官方身份、环境和 workflow guard，否则发布、Issue 治理与内核沙箱参考仍不可用。

## 验证

workflow 检查会固定仓库保护、公共 runner 回退、E2E 显式启用变量和仅限官方仓库的发布条件。fork 通过打开 Pull Request、观察 `all checks passed` 并确认没有任务等待 `dsh-*` 或自托管 runner 来验证结果。master 上的文档变更通过现有 Pages 部署 workflow 验证 CD。
