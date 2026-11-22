# Loop — Everesto pipeline stages

Reusable CI/CD stages for every application created from Backstage. Apps call Loop pinned to a
commit SHA (`# v1.x.y` comment) — Dependabot proposes upgrades.

| Stage | Use | Can be switched off |
|---|---|---|
| guard (`.github/workflows/guard.yml`) | Gitleaks + Trivy dependencies — build/deploy `needs: guard` | **no** (mandatory) |
| `stages/install` | language setup, deps, tests + coverage (`stack: python \| java`) | no |
| `stages/semgrep` | code security | yes |
| `stages/sonarqube` | quality gate + coverage | yes |
| `stages/build-image` | arm64 image + Trivy image scan | scan always runs; `trivy-mode: report` = findings don't fail |
| `stages/push-image` | push to ghcr.io, keep last 5 | — |
| `stages/deploy-argocd` | bump tag in `<project>-gitops-cd` | CD only |

## v1.1 — the whole pipeline with switches (`pipeline.yml`)
One reusable workflow instead of listing every stage. Every run ends with a **scanned linux/arm64 image
in ghcr.io** (pushed from the default branch only; pull requests build + scan but publish nothing).

```yaml
jobs:
  loop:
    uses: Everesto-Inc/loop/.github/workflows/pipeline.yml@<sha> # v1.1.0
    with: { stack: python, tests: true, semgrep: false, sonarqube: true, trivy: enforce }
    secrets: { SONAR_TOKEN: '${{ secrets.SONAR_TOKEN }}' }   # only what Loop needs — never `inherit` (least privilege)
    permissions: { contents: read, packages: write }
```

| Switch | Values | Off means |
|---|---|---|
| `tests` | true / false | no install + tests + coverage. When on: **no tests written → skipped** (notice), **a failing test stops the pipeline**, pytest / pytest-cov installed if the repo doesn't list them |
| `semgrep` | true / false | no code-security scan |
| `sonarqube` | true / false | no quality gate (also skipped, with a notice, when there is no `SONAR_TOKEN`) |
| `trivy` | **enforce** / report | report = dependency + image scans still run and show findings, but never fail. Backstage flags it. |
| build image, Gitleaks, push | always on | — the pipeline's purpose is a scanned image |

Extra images (e.g. agents of the same repo): add your own job after `loop` (`needs: loop`).
The v1.0 stages below stay supported — existing apps keep working unchanged.

## Customising (3 levels)
1. **Inputs** — every stage has settings (`extra-args`, `test-command`, `dockerfile`, …).
2. **Own steps** — add `run:` steps between Loop stages in your `ci.yml`.
3. **Eject** — replace a stage by your own script. Backstage shows the stage as *ejected*.
   Guard stays mandatory; Backstage flags a pipeline without it.

Example: see `docs/index.md`.
