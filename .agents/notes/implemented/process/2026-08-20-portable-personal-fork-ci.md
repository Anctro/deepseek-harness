# Agent Note: Portable CI for personal forks

Status: implemented

English | [中文](2026-08-20-portable-personal-fork-ci.zh.md)

## Problem

The official repository uses organization-scoped larger runners, private self-hosted standby pools, a GitHub App, and publication credentials. A personal fork inherits the workflows but cannot access those resources. Pull-request jobs would remain queued on unavailable runner labels, real-API runs would fail without `DEEPSEEK_API_KEY_EXTERNAL`, issue lifecycle automation would fail without the app credentials, and a manually dispatched release must never publish the official npm or PyPI namespaces from a fork.

## Decision

The existing workflows remain the source of truth for CI and documentation deployment. Pull-request jobs select standard GitHub-hosted Linux and Windows runners when `github.repository` is not `deepseek-ai/deepseek-harness`; the official repository retains its larger-runner and failover selectors. Self-hosted standby drills and organization runner benchmarks execute only in the official repository.

Real-API E2E runs in the official repository by default. A fork must set the repository variable `DSH_REAL_API_E2E_ENABLED=true` and configure `DEEPSEEK_API_KEY_EXTERNAL` before the job can run. The workflow continues to use `pull_request`, never `pull_request_target`, so forked pull requests cannot receive repository secrets.

Every npm and PyPI publish job requires `github.repository == 'deepseek-ai/deepseek-harness'` in addition to its manual publish input. Forks may retain keyless package and artifact rehearsals, but cannot enter the official publication jobs. Issue lifecycle automation has the same official-repository guard because its GitHub App installation is repository-specific. The portable Pages workflow remains the fork's deployment path.

## Alternatives considered

**Add a second fork-only workflow.** Rejected because it would duplicate the repository's gate inventory and drift as upstream CI changes. Repository-aware runner selection preserves the existing checks and stable `all checks passed` verdict.

**Disable all inherited workflows.** Rejected because a fork would lose coverage, build, snapshot, package, Python, and documentation evidence that already runs without private credentials.

**Allow publication when a fork supplies similarly named secrets.** Rejected because the packages and trusted-publisher identities belong to the official namespaces. A downstream distribution needs its own package names and a separate release decision.

## Consequences

Personal forks can run the upstream pull-request checks on GitHub-hosted runners and deploy documentation through GitHub Pages without access to DeepSeek infrastructure. These runners have less capacity than the official pools, so fork checks may take longer. Real-provider tests remain opt-in and secret-backed. Publication and issue lifecycle jobs remain unavailable until a downstream intentionally replaces the official identities and workflow guards with its own distribution design.

## Verification

Workflow checks pin the repository guards, public-runner fallback, E2E opt-in variable, and official-only publication conditions. A fork validates the result by opening a pull request, observing `all checks passed`, and confirming that no job waits for a `dsh-*` or self-hosted runner. A master documentation change validates CD through the existing Pages deployment workflow.
