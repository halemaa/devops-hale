\# GitHub Actions Guide



\## What it is

GitHub Actions is CI/CD built into GitHub. You write YAML files in

`.github/workflows/` and GitHub runs them on events — a push, a PR, a schedule,

a manual trigger. Free tier is generous for public repos.



\## The mental model

```mermaid

flowchart LR

&#x20;   Event\[Event: push, PR, schedule] --> Workflow\[Workflow YAML]

&#x20;   Workflow --> Job1\[Job: test]

&#x20;   Workflow --> Job2\[Job: build]

&#x20;   Workflow --> Job3\[Job: deploy]

&#x20;   Job1 --> Step1\[Step: checkout]

&#x20;   Job1 --> Step2\[Step: run tests]

&#x20;   Job2 --> Step3\[Step: build image]

&#x20;   Job3 --> Step4\[Step: deploy]

```



\*\*Workflow → Jobs → Steps.\*\* Jobs run in parallel by default on separate

runners; steps inside a job run sequentially on the same runner.



\## The vocabulary

\- \*\*Workflow\*\* — a YAML file in `.github/workflows/`.

\- \*\*Event\*\* — what triggers it (`push`, `pull\_request`, `schedule`, `workflow\_dispatch`).

\- \*\*Job\*\* — a set of steps that run on one runner.

\- \*\*Runner\*\* — the VM that executes the job (GitHub-hosted or self-hosted).

\- \*\*Step\*\* — a single command or an action.

\- \*\*Action\*\* — a reusable unit (`actions/checkout@v4`, `docker/build-push-action@v5`).

\- \*\*Secret\*\* — encrypted value injected as env var (`${{ secrets.AWS\_KEY }}`).

\- \*\*Matrix\*\* — run the same job across multiple configs (OS, language versions).



\## A minimal workflow

```yaml

name: CI



on:

&#x20; push:

&#x20;   branches: \[main]

&#x20; pull\_request:



jobs:

&#x20; test:

&#x20;   runs-on: ubuntu-latest

&#x20;   steps:

&#x20;     - uses: actions/checkout@v4

&#x20;     - uses: actions/setup-node@v4

&#x20;       with:

&#x20;         node-version: '20'

&#x20;     - run: npm ci

&#x20;     - run: npm test

```



Drop that in `.github/workflows/ci.yml`, push, and it runs.



\## A DevSecOps-flavored workflow

```yaml

name: Build, scan, deploy



on:

&#x20; push:

&#x20;   branches: \[main]



jobs:

&#x20; security:

&#x20;   runs-on: ubuntu-latest

&#x20;   steps:

&#x20;     - uses: actions/checkout@v4

&#x20;     - name: Secret scan

&#x20;       uses: gitleaks/gitleaks-action@v2

&#x20;     - name: IaC scan

&#x20;       uses: bridgecrewio/checkov-action@master

&#x20;     - name: SAST

&#x20;       uses: returntocorp/semgrep-action@v1



&#x20; build:

&#x20;   needs: security

&#x20;   runs-on: ubuntu-latest

&#x20;   steps:

&#x20;     - uses: actions/checkout@v4

&#x20;     - name: Build image

&#x20;       run: docker build -t myapp:${{ github.sha }} .

&#x20;     - name: Container scan

&#x20;       uses: aquasecurity/trivy-action@master

&#x20;       with:

&#x20;         image-ref: myapp:${{ github.sha }}

```



This one pattern — scan → build → scan image — is what "DevSecOps" looks like

in practice. Interviewers love seeing it.



\## Secrets — the safe way

Repo → Settings → Secrets and variables → Actions. Reference them as

`${{ secrets.NAME }}`. \*\*Never print them\*\* — GitHub masks them in logs, but

`echo $SECRET | base64` bypasses masking. Don't.



For AWS/GCP, prefer \*\*OIDC\*\* over long-lived keys — the workflow assumes an

IAM role at runtime, no static secrets stored.



\## Common gotchas

\- \*\*`pull\_request` from a fork has no secrets\*\* — deliberate, for security. Use `pull\_request\_target` carefully or split workflows.

\- \*\*`${{ }}` inside `run:` is expanded by GitHub before bash sees it\*\* — quote carefully to avoid injection.

\- \*\*YAML indentation\*\* — every third bug. Use an editor with YAML linting.

\- \*\*`actions/checkout@v4` fetches shallow (depth 1) by default\*\* — some tools need full history (`fetch-depth: 0`).

\- \*\*Cache misses silently\*\* — `actions/cache` will just build cold instead of failing. Verify with logs.

\- \*\*Minutes on private repos are metered\*\* — matrix builds can burn through the free tier fast.



\## Patterns worth memorizing

\- \*\*`needs:`\*\* — job dependencies. `build` waits for `test` to pass.

\- \*\*`if:`\*\* — conditional steps or jobs (`if: github.ref == 'refs/heads/main'`).

\- \*\*`matrix:`\*\* — test across versions:

```yaml

&#x20; strategy:

&#x20;   matrix:

&#x20;     node: \[18, 20, 22]

```

\- \*\*`environments`\*\* — protection rules + secrets scoped to `staging` / `prod`. Enables manual approval before deploy.

\- \*\*Reusable workflows\*\* — `uses: ./.github/workflows/deploy.yml` — DRY across repos.



\## What I want to try

\- Add a CI workflow to `devops-hale` that lints my Markdown on every PR.

\- Build a full pipeline for a Dockerized app: test → build → push to GHCR → deploy.

\- Add OIDC auth to AWS so the workflow deploys without any stored keys.

\- Add all four security scanners (Semgrep, Trivy, Checkov, Gitleaks) to break the build on critical findings.



\## Sources

\- GitHub Actions docs (docs.github.com/actions)

\- Awesome Actions list (github.com/sdras/awesome-actions)

\- "GitHub Actions in Action" — Krief, Aliyev, De Waele

