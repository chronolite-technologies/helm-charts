# Chronolite Helm Charts - Operations Concept

**Version:** 1.0  
**Last Updated:** October 2025  
**Maintainer:** Chronolite Technologies

---

## Overview

Distributed Helm chart development model with centralized registry, automated upstream synchronization, security scanning, and image replication for air-gapped deployments.

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    Main Registry Repository                      │
│              (helm-charts - GitHub Pages)                        │
│  • Aggregates all charts                                         │
│  • Serves via GitHub Pages + GHCR                                │
│  • Single source of truth for users                              │
└───────────────────────────┬─────────────────────────────────────┘
                            │ (triggers via repository_dispatch)
                            │
        ┌───────────────────┴───────────────────┐
        │                                       │
┌───────▼──────────┐                   ┌───────▼──────────┐
│  chart-keycloak  │                   │  chart-grafana   │
│  • Git submodule │                   │  • Git submodule │
│  • Sync script   │                   │  • Sync script   │
│  • CI/CD         │       ...         │  • CI/CD         │
│  • Security scan │                   │  • Security scan │
└───────┬──────────┘                   └───────┬──────────┘
        │                                      │
        │ (reads from)                         │
        │                                      │
┌───────▼──────────┐                   ┌───────▼──────────┐
│ upstream/source  │                   │ upstream/source  │
│ (git submodule)  │                   │ (git submodule)  │
└──────────────────┘                   └──────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│              Image Replication & Security Layer                  │
│  • Mirror public images to GHCR                                  │
│  • Daily Trivy vulnerability scans                               │
│  • Version tracking via .images.yaml                             │
│  • Dependabot for automated updates                              │
└─────────────────────────────────────────────────────────────────┘
```

## Repository Structure

### Main Registry (`helm-charts`)

Central aggregation point for all charts:

```
helm-charts/
├── charts/                    # All packaged charts
│   ├── keycloak-1.0.0.tgz
│   ├── grafana-2.1.0.tgz
│   └── ...
├── index.yaml                 # Helm repository index
├── .github/workflows/
│   ├── update-index.yml      # Regenerates index on triggers
│   └── publish-pages.yml     # Deploys to GitHub Pages
└── README.md                  # Main documentation
```

### Individual Chart Repositories (`chart-*`)

Each chart in dedicated repository:

```
chart-keycloak/
├── .github/workflows/
│   ├── sync-upstream.yml     # Weekly: sync from submodule
│   ├── publish.yml           # On tag: publish to registries
│   ├── scan-images.yml       # Daily: security scanning
│   └── replicate-images.yml  # Daily: mirror images
├── charts/keycloak/
│   ├── Chart.yaml
│   ├── values.yaml           # Simplified user config
│   └── templates/            # Synced from upstream
├── upstream/
│   └── source-repo/          # Git submodule to official charts
├── scripts/
│   └── sync.sh               # Extracts + templatizes manifests
├── .images.yaml              # Image replication config
└── README.md
```

## Core Workflows

### 1. Upstream Synchronization

**Purpose:** Keep charts aligned with official upstream sources

**Process:**
```
┌─────────────┐
│   Trigger   │ (Weekly cron OR manual OR submodule update)
└──────┬──────┘
       │
       ▼
┌─────────────────────┐
│ Update Submodule    │ git submodule update --remote
└──────┬──────────────┘
       │
       ▼
┌─────────────────────┐
│  Run sync.sh        │ • Extract CRDs, manifests
│                     │ • Templatize with Helm values
│                     │ • Update Chart.yaml version
└──────┬──────────────┘
       │
       ▼
┌─────────────────────┐
│  Create PR          │ Auto-PR for review
└─────────────────────┘
```

**Sync Script Logic:**
1. Fetch latest upstream tag/commit
2. Extract relevant manifests (CRDs, deployments, etc.)
3. Convert to Helm templates with `{{ .Values }}` placeholders
4. Add conditional `{{- if .Values.component.enabled }}`
5. Update Chart.yaml with upstream version

### 2. Security Scanning

**Purpose:** Continuous vulnerability monitoring of all container images

**Process:**
```
┌─────────────┐
│   Trigger   │ (Daily 2 AM OR PR with .images.yaml changes)
└──────┬──────┘
       │
       ▼
┌─────────────────────┐
│  Parse .images.yaml │ Extract all images + versions
└──────┬──────────────┘
       │
       ▼
┌─────────────────────┐
│  Trivy Scan         │ For each image:
│                     │ • Pull image
│                     │ • Scan vulnerabilities
│                     │ • Generate SARIF report
└──────┬──────────────┘
       │
       ▼
┌─────────────────────┐
│  Upload Results     │ • GitHub Security tab
│                     │ • Fail on CRITICAL/HIGH
│                     │ • Create issue if threshold exceeded
└─────────────────────┘
```

**Severity Handling:**
- **CRITICAL**: Block release, create issue
- **HIGH**: Block release, create issue
- **MEDIUM**: Warn only
- **LOW**: Informational

### 3. Image Replication

**Purpose:** Mirror public images to private registry for:
- Air-gapped deployments
- Rate limit avoidance
- Supply chain security
- Version control

**Process:**
```
┌─────────────┐
│   Trigger   │ (Daily 3 AM OR .images.yaml changes)
└──────┬──────┘
       │
       ▼
┌─────────────────────┐
│  Parse Config       │ Read .images.yaml
└──────┬──────────────┘
       │
       ▼
┌─────────────────────┐
│  For Each Image     │ Matrix strategy
└──────┬──────────────┘
       │
       ▼
┌─────────────────────┐
│  Pull → Tag → Push  │ source → target registry
└──────┬──────────────┘
       │
       ▼
┌─────────────────────┐
│  Scan Replica       │ Trivy scan mirrored image
└──────┬──────────────┘
       │
       ▼
┌─────────────────────┐
│  Update Metadata    │ Track version, scan date
└─────────────────────┘
```

### 4. Chart Publishing

**Purpose:** Release charts to registries on version tags

**Process:**
```
┌─────────────┐
│  Git Tag    │ v1.2.3 pushed
└──────┬──────┘
       │
       ▼
┌─────────────────────┐
│  Package Chart      │ helm package charts/name
└──────┬──────────────┘
       │
       ▼
┌─────────────────────┐
│  Push to GHCR       │ helm push oci://ghcr.io/...
└──────┬──────────────┘
       │
       ▼
┌─────────────────────┐
│  Trigger Main Repo  │ repository_dispatch event
└──────┬──────────────┘
       │
       ▼
┌─────────────────────┐
│  Main: Pull Chart   │ Download from GHCR
│  Main: Update Index │ helm repo index
│  Main: Deploy Pages │ Publish to GitHub Pages
└─────────────────────┘
```

## Image Configuration (`.images.yaml`)

Centralized image tracking per chart:

```yaml
registry:
  source: quay.io
  target: ghcr.io/chronolite-technologies
  prefix: "mirrors"

images:
  component-name:
    source: quay.io/project/image
    target: ghcr.io/chronolite-technologies/mirrors/image
    versions:
      - "1.2.3"
      - "1.2.2"
    scanPolicy: "daily"
    autoUpdate: true

scanning:
  enabled: true
  schedule: "0 2 * * *"
  severity:
    fail:
      - CRITICAL
      - HIGH
    warn:
      - MEDIUM
  ignoredCVEs:
    - CVE-2024-12345  # Reason: false positive

dependencyBot:
  enabled: true
  schedule: "weekly"
  autoMerge:
    enabled: false
```

## Dependency Management

### Dependabot Configuration

Tracks multiple dependency types:

```yaml
updates:
  - package-ecosystem: "github-actions"   # Workflow actions
  - package-ecosystem: "docker"           # Container images
  - package-ecosystem: "gitsubmodule"     # Upstream sources
```

**Auto-update Strategy:**
- GitHub Actions: Weekly, auto-merge patch versions
- Docker images: Weekly, manual review required
- Submodules: Weekly, triggers sync workflow

## Security Model

### Multi-layer Security

1. **Source Verification**
   - Git submodules pin to specific commits/tags
   - Submodule updates reviewed via PR

2. **Image Scanning**
   - Daily Trivy scans on all images
   - Results uploaded to GitHub Security tab
   - CI blocks on CRITICAL/HIGH vulnerabilities

3. **Image Replication**
   - Pull from trusted upstream registries
   - Scan before pushing to private registry
   - Version pinning in `.images.yaml`

4. **Supply Chain**
   - SBOM generation for all images
   - Provenance tracking via Git history
   - Signed commits (optional)

### Vulnerability Response

```
CVE Detected
    │
    ▼
[Severity Check]
    │
    ├─ CRITICAL/HIGH
    │   ├─ Block PR/Release
    │   ├─ Create Issue
    │   └─ Notify Team
    │
    └─ MEDIUM/LOW
        ├─ Log Warning
        └─ Continue
```

## Automation Strategy

### Minimal Manual Intervention

**Automated:**
- Upstream sync checks (weekly)
- Security scanning (daily)
- Image replication (daily)
- Dependency updates (weekly)
- Chart publishing (on tag)
- Registry index updates (on publish)

**Manual Review Required:**
- Upstream sync PRs (breaking changes)
- Critical vulnerability fixes
- Major version updates
- Chart configuration changes

## Integration Points

### Main Registry Triggers

Chart repos → Main registry communication:

```yaml
# In chart repo publish workflow
- name: Trigger main registry
  uses: peter-evans/repository-dispatch@v3
  with:
    token: ${{ secrets.PAT_TOKEN }}
    repository: chronolite-technologies/helm-charts
    event-type: chart-published
    client-payload: |
      {
        "chart": "keycloak",
        "version": "v1.2.3",
        "repository": "chart-keycloak"
      }
```

```yaml
# In main registry
on:
  repository_dispatch:
    types: [chart-published]

jobs:
  update:
    steps:
      - name: Pull chart
        run: |
          helm pull oci://ghcr.io/${{ github.event.client_payload.repository }}
      
      - name: Update index
        run: helm repo index charts/
```

## Best Practices

### Chart Development

1. **Keep values.yaml simple** - Abstract complexity into templates
2. **Use helpers** - DRY principle for labels, names
3. **Default to secure** - Minimal permissions, read-only filesystems
4. **Resource limits** - Always define requests/limits

### Maintenance

1. **Review sync PRs promptly** - Delays accumulate changes
2. **Test in staging** - Never auto-merge to production
3. **Document overrides** - When deviating from upstream
4. **Version pinning** - Lock critical dependencies

### Security

1. **Scan early, scan often** - Don't wait for release
2. **Mirror critical images** - Reduce external dependencies
3. **Track CVEs** - Maintain ignored CVE list with justifications
4. **Rotate credentials** - Registry tokens, PATs

---

**Document Version:** 1.0  
**Last Review:** October 2025  
**Next Review:** January 2026
