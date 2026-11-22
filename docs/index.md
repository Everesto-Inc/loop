# Loop

Pipeline stages shared by all Everesto applications.

## v1.1 (recommended for new apps): one workflow with switches

```yaml
jobs:
  loop:
    uses: Everesto-Inc/loop/.github/workflows/pipeline.yml@<sha> # v1.1.0
    with: { stack: python, tests: true, semgrep: false, sonarqube: true, trivy: enforce }
    secrets: { SONAR_TOKEN: '${{ secrets.SONAR_TOKEN }}' }   # only what Loop needs — never `inherit` (least privilege)
    permissions: { contents: read, packages: write }
```

`tests` · `semgrep` · `sonarqube`: true / false · `trivy`: enforce / report (scans always run).
Always on: Gitleaks, Trivy scans, image build, push from the default branch. See the README for all inputs.

## v1.0: stage by stage

An app's `ci.yml` looks like:

```yaml
jobs:
  guard:
    uses: Everesto-Inc/loop/.github/workflows/guard.yml@<sha> # v1.0.0
  ci:
    needs: guard
    runs-on: ubuntu-24.04-arm
    steps:
      - uses: actions/checkout@<sha>
      - name: Tests + coverage
        uses: Everesto-Inc/loop/stages/install@<sha>
        with: { stack: python }
      - name: Quality gate (SonarQube)
        uses: Everesto-Inc/loop/stages/sonarqube@<sha>
        with: { token: '${{ secrets.SONAR_TOKEN }}' }
      - run: ./scripts/my-own-step.sh          # level 2: own step
      - name: Build + scan image (Trivy)
        id: image
        uses: Everesto-Inc/loop/stages/build-image@<sha>
        with: { image: ghcr.io/everesto-inc/<repo> }
```

```mermaid
flowchart LR
  guard[Guard: Gitleaks + Trivy deps] --> install[Install + tests]
  install --> semgrep[Semgrep] --> sonar[SonarQube] --> build[Build + Trivy image]
  build --> push[Push ghcr.io] --> deploy[Deploy: tag bump → ArgoCD]
```
