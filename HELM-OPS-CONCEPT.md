# Chronolite Helm Charts - Operations Concept

**Version:** 2.0  
**Last Updated:** October 14, 2025  
**Maintainer:** Chronolite Technologies

---

## Overview

Distributed Helm chart development model with centralized registry, automated upstream synchronization, security scanning, and image replication for air-gapped deployments.

This document describes the operational model implemented by the [helm-workflows](https://github.com/chronolite-technologies/helm-workflows) reusable workflows repository.

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

All workflows are provided as reusable workflows in [helm-workflows](https://github.com/chronolite-technologies/helm-workflows) repository.

### 1. Upstream Synchronization (`sync-upstream.yml`)

**Purpose:** Keep charts aligned with official upstream sources

**Implementation:**
- Single job with automatic change detection
- No separate secrets required (uses `GITHUB_TOKEN`)
- Combines submodule update and sync script execution
- Automatic PR creation with change detection

**Process:**

```text
┌─────────────┐
│   Trigger   │ (Schedule OR manual)
└──────┬──────┘
       │
       ▼
┌─────────────────────┐
│ Update Submodule    │ • Fetch latest upstream tag
│ & Run sync.sh       │ • Execute sync script
│                     │ • Extract/templatize manifests
└──────┬──────────────┘
       │
       ▼
┌─────────────────────┐
│  Create PR          │ Auto-creates PR if changes detected
└─────────────────────┘
```

**Configuration:**

```yaml
jobs:
  sync:
    uses: chronolite-technologies/helm-workflows/.github/workflows/sync-upstream.yml@main
    with:
      upstream-path: upstream/source    # Path to git submodule
      chart-path: charts/chart-name     # Path to chart directory
      sync-script: scripts/sync.sh      # Optional, default shown
```

### 2. Security Scanning (`scan-images.yml`)

**Purpose:** Continuous vulnerability monitoring of all container images

**Implementation:**
- Single job (no matrix overhead)
- Inline image parsing from `.images.yaml`
- Unified severity threshold configuration
- Automatic SARIF upload to GitHub Security tab

**Process:**

```text
┌─────────────┐
│   Trigger   │ (Schedule OR manual)
└──────┬──────┘
       │
       ▼
┌─────────────────────┐
│  Parse & Scan       │ • Read .images.yaml
│                     │ • Loop through images/versions
│                     │ • Trivy scan each image
│                     │ • Upload SARIF reports
│                     │ • Check severity threshold
└─────────────────────┘
```

**Configuration:**

```yaml
jobs:
  scan:
    uses: chronolite-technologies/helm-workflows/.github/workflows/scan-images.yml@main
    with:
      images-config: .images.yaml      # Optional, default shown
      severity: CRITICAL               # CRITICAL, HIGH, or MEDIUM
```

**Severity Handling:**
- **CRITICAL**: Workflow fails, blocks release
- **HIGH**: Workflow fails if configured
- **MEDIUM**: Workflow fails if configured
- **LOW**: Informational only

### 3. Image Replication (`replicate-images.yml`)

**Purpose:** Mirror public images to private registry for:
- Air-gapped deployments
- Rate limit avoidance
- Supply chain security

**Implementation:**
- Single job with inline processing
- No matrix strategy overhead
- Uses `GITHUB_TOKEN` for authentication
- Automatic prefix handling from config

**Process:**

```text
┌─────────────┐
│   Trigger   │ (Schedule OR manual)
└──────┬──────┘
       │
       ▼
┌─────────────────────┐
│  Parse & Replicate  │ • Read .images.yaml
│                     │ • Loop through images/versions
│                     │ • Pull from source
│                     │ • Tag for target registry
│                     │ • Push to target
└─────────────────────┘
```

**Configuration:**

```yaml
jobs:
  replicate:
    uses: chronolite-technologies/helm-workflows/.github/workflows/replicate-images.yml@main
    with:
      images-config: .images.yaml            # Optional, default shown
      target-org: your-org                   # Required
      target-registry: ghcr.io               # Optional, default shown
```

### 4. Chart Publishing (`publish-chart.yml`)

**Purpose:** Release charts to registries on version tags

**Implementation:**
- Single job combining package and publish
- Uses `GITHUB_TOKEN` for registry authentication
- Requires `PAT_TOKEN` only for main registry trigger
- Automatic GitHub release creation

**Process:**

```text
┌─────────────┐
│  Git Tag    │ v1.2.3 pushed
└──────┬──────┘
       │
       ▼
┌─────────────────────┐
│  Package & Publish  │ • helm package
│                     │ • Extract version
│                     │ • Push to OCI registry
└──────┬──────────────┘
       │
       ▼
┌─────────────────────┐
│  Trigger Main Repo  │ repository_dispatch event
└──────┬──────────────┘
       │
       ▼
┌─────────────────────┐
│  Create Release     │ GitHub release with chart archive
└─────────────────────┘
```

**Configuration:**

```yaml
jobs:
  publish:
    uses: chronolite-technologies/helm-workflows/.github/workflows/publish-chart.yml@main
    with:
      chart-path: charts/chart-name         # Required
      chart-name: chart-name                # Required
      registry-org: your-org                # Required
      main-registry-repo: your-org/helm-charts  # Required
      oci-registry: ghcr.io                 # Optional, default shown
    secrets:
      pat-token: ${{ secrets.PAT_TOKEN }}   # Required for main registry trigger
```

## Image Configuration (`.images.yaml`)

Simplified image tracking configuration:

```yaml
registry:
  prefix: "mirrors"  # Optional: prefix for target images

images:
  keycloak:
    source: quay.io/keycloak/keycloak
    versions:
      - "23.0"
      - "24.0"
  
  postgres:
    source: docker.io/library/postgres
    versions:
      - "16"
      - "15"
```

**Notes:**

- The `source` field specifies the upstream image repository
- The `versions` array lists all versions to scan/replicate
- The optional `prefix` adds a namespace to target images (e.g., `ghcr.io/org/mirrors/keycloak`)
- Scanning policy is controlled at workflow level via `severity` input
- Configuration is simple and focused on essential tracking

## Security Model

### Multi-layer Security

1. **Source Verification**
   - Git submodules pin to specific commits/tags
   - Submodule updates reviewed via PR

2. **Image Scanning**
   - Scheduled Trivy scans on all images
   - Results uploaded to GitHub Security tab
   - Configurable severity thresholds (CRITICAL, HIGH, MEDIUM)

3. **Image Replication**
   - Pull from trusted upstream registries
   - Push to private registry for air-gap support
   - Version pinning in `.images.yaml`

4. **Supply Chain**
   - Provenance tracking via Git history
   - All changes reviewed via PR
   - Automated workflows use `GITHUB_TOKEN` only

### Vulnerability Response

```text
CVE Detected
    │
    ▼
[Trivy Scan]
    │
    ▼
[Severity Check]
    │
    ├─ Meets Threshold (CRITICAL/HIGH/MEDIUM)
    │   ├─ Block Workflow
    │   └─ Upload to Security Tab
    │
    └─ Below Threshold
        └─ Continue (logged)
```

### Required Secrets

**Minimal secret configuration:**

- `PAT_TOKEN` - Personal Access Token (only for `publish-chart.yml`)
  - Scopes: `repo`, `write:packages`
  - Used to trigger main registry updates via `repository_dispatch`

**No longer required:**
- ❌ ~~`SOURCE_USERNAME`/`SOURCE_PASSWORD`~~ - Removed (public registries)
- ❌ ~~`REGISTRY_TOKEN`~~ - Use `GITHUB_TOKEN` instead
- ❌ ~~Custom tokens~~ - Workflows use automatic token

## Automation Strategy

### Minimal Manual Intervention

**Automated:**

- Upstream sync checks (scheduled or manual)
- Security scanning (scheduled or manual)
- Image replication (scheduled or manual)
- Chart publishing (on tag push)
- Registry index updates (via repository dispatch)

**Manual Review Required:**

- Upstream sync PRs (review changes)
- Severity threshold configuration
- Version updates in `.images.yaml`
- Chart configuration changes

## Integration Points

### Main Registry Triggers

Chart repos → Main registry communication via `repository_dispatch`:

```yaml
# In chart repo publish workflow (automatic via reusable workflow)
- name: Trigger Main Registry Update
  uses: peter-evans/repository-dispatch@v3
  with:
    token: ${{ secrets.pat-token }}
    repository: ${{ inputs.main-registry-repo }}
    event-type: chart-published
    client-payload: |
      {
        "chart": "${{ inputs.chart-name }}",
        "version": "${{ steps.publish.outputs.version }}",
        "repository": "${{ github.repository }}",
        "oci_url": "oci://${{ inputs.oci-registry }}/${{ inputs.registry-org }}/charts/${{ inputs.chart-name }}"
      }
```

```yaml
# In main registry repository
on:
  repository_dispatch:
    types: [chart-published]

jobs:
  update:
    steps:
      - name: Pull chart from OCI registry
        run: |
          helm pull ${{ github.event.client_payload.oci_url }} \
            --version ${{ github.event.client_payload.version }}
      
      - name: Update repository index
        run: helm repo index charts/ --url https://your-org.github.io/helm-charts
      
      - name: Commit and push
        run: |
          git add charts/ index.yaml
          git commit -m "Add ${{ github.event.client_payload.chart }} ${{ github.event.client_payload.version }}"
          git push
```

### Workflow Reference

All chart repositories reference reusable workflows from `helm-workflows`:

```yaml
# Example: .github/workflows/publish.yml in chart repository
name: Publish
on:
  push:
    tags: ['v*']

jobs:
  publish:
    uses: chronolite-technologies/helm-workflows/.github/workflows/publish-chart.yml@main
    with:
      chart-path: charts/keycloak
      chart-name: keycloak
      registry-org: chronolite-technologies
      main-registry-repo: chronolite-technologies/helm-charts
    secrets:
      pat-token: ${{ secrets.PAT_TOKEN }}
```

## Best Practices

### Chart Development

1. **Keep values.yaml simple** - Abstract complexity into templates
2. **Use helpers** - DRY principle for labels, names
3. **Default to secure** - Minimal permissions, read-only filesystems
4. **Resource limits** - Always define requests/limits

### Workflow Configuration

1. **Pin workflow versions** - Use `@v1.0.0` tags instead of `@main` for stability
2. **Configure schedules appropriately** - Balance freshness vs. resource usage
3. **Set severity thresholds** - Match your security requirements
4. **Use descriptive chart names** - Follow conventions for discoverability

### Maintenance

1. **Review sync PRs promptly** - Delays accumulate changes
2. **Test in staging** - Never auto-merge to production
3. **Update `.images.yaml` regularly** - Track new versions
4. **Monitor Security tab** - Address findings proactively

### Security

1. **Scan regularly** - Schedule daily or per-PR scans
2. **Set appropriate thresholds** - Start with CRITICAL, add HIGH as needed
3. **Review SARIF reports** - Use GitHub Security tab for tracking
4. **Rotate PAT tokens** - Follow security best practices for PAT_TOKEN

## Workflow Optimization

The reusable workflows in `helm-workflows` are optimized for:

- **Simplicity**: Single-job design, no matrix overhead
- **Performance**: Inline processing, minimal steps
- **Maintainability**: Clear configuration, sensible defaults
- **Security**: Minimal secrets, automatic token usage

---

**Document Version:** 2.0  
**Last Review:** October 14, 2025  
**Next Review:** January 2026
