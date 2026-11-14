# Loop — Everesto pipeline stages

Reusable CI/CD stages for every application created from Backstage. Apps call Loop pinned to a
commit SHA (`# v1.x.y` comment) — Dependabot proposes upgrades.

| Stage | Use | Can be switched off |
|---|---|---|
| guard (`.github/workflows/guard.yml`) | Gitleaks + Trivy dependencies — build/deploy `needs: guard` | **no** (mandatory) |
| `stages/install` | language setup, deps, tests + coverage (`stack: python \| java`) | no |
| `stages/semgrep` | code security | yes |
| `stages/sonarqube` | quality gate + coverage | yes |
| `stages/build-image` | arm64 image + Trivy image scan | no (scan is part of the build) |
| `stages/push-image` | push to ghcr.io, keep last 5 | — |
| `stages/deploy-argocd` | bump tag in `<project>-gitops-cd` | CD only |

## Customising (3 levels)
1. **Inputs** — every stage has settings (`extra-args`, `test-command`, `dockerfile`, …).
2. **Own steps** — add `run:` steps between Loop stages in your `ci.yml`.
3. **Eject** — replace a stage by your own script. Backstage shows the stage as *ejected*.
   Guard stays mandatory; Backstage flags a pipeline without it.

Example: see `docs/index.md`.
