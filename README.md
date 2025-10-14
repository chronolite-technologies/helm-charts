# Chronolite Helm Charts

Official Helm chart registry for Chronolite Technologies infrastructure components.

![Maintained](https://img.shields.io/badge/maintained-yes-brightgreen?style=for-the-badge)
![License](https://img.shields.io/badge/license-Apache%202.0-blue?style=for-the-badge)

---

## 📦 Installation

### Traditional Helm Repository

```bash
# Add repository
helm repo add chronolite https://chronolite-technologies.github.io/helm-charts

# Update repository index
helm repo update

# Search available charts
helm search repo chronolite
```

### OCI Registry (GitHub Container Registry)

```bash
# Install directly from OCI
helm install my-release oci://ghcr.io/chronolite-technologies/charts/<chart-name>
```

---

## 📊 Available Charts

| Chart | Description | Last Activity | Repository |
|-------|-------------|---------------|------------|
| **argocd** | GitOps continuous delivery for Kubernetes | ![Commit](https://img.shields.io/github/last-commit/chronolite-technologies/chart-argocd?style=flat-square&label=) | [chart-argocd](https://github.com/chronolite-technologies/chart-argocd) |
| **calico** | Network policy and security for Kubernetes | ![Commit](https://img.shields.io/github/last-commit/chronolite-technologies/chart-calico?style=flat-square&label=) | [chart-calico](https://github.com/chronolite-technologies/chart-calico) |
| **cert-manager** | Automated certificate management | ![Commit](https://img.shields.io/github/last-commit/chronolite-technologies/chart-cert-manager?style=flat-square&label=) | [chart-cert-manager](https://github.com/chronolite-technologies/chart-cert-manager) |
| **gitea** | Lightweight self-hosted Git service | ![Commit](https://img.shields.io/github/last-commit/chronolite-technologies/chart-gitea?style=flat-square&label=) | [chart-gitea](https://github.com/chronolite-technologies/chart-gitea) |
| **grafana** | Analytics and monitoring dashboards | ![Commit](https://img.shields.io/github/last-commit/chronolite-technologies/chart-grafana?style=flat-square&label=) | [chart-grafana](https://github.com/chronolite-technologies/chart-grafana) |
| **istio** | Service mesh for microservices | ![Commit](https://img.shields.io/github/last-commit/chronolite-technologies/chart-istio?style=flat-square&label=) | [chart-istio](https://github.com/chronolite-technologies/chart-istio) |
| **keycloak** | Identity and access management | ![Commit](https://img.shields.io/github/last-commit/chronolite-technologies/chart-keycloak?style=flat-square&label=) | [chart-keycloak](https://github.com/chronolite-technologies/chart-keycloak) |
| **knative** | Serverless workloads on Kubernetes | ![Commit](https://img.shields.io/github/last-commit/chronolite-technologies/chart-knative?style=flat-square&label=) | [chart-knative](https://github.com/chronolite-technologies/chart-knative) |
| **loki** | Log aggregation system | ![Commit](https://img.shields.io/github/last-commit/chronolite-technologies/chart-loki?style=flat-square&label=) | [chart-loki](https://github.com/chronolite-technologies/chart-loki) |
| **netbird** | Secure mesh VPN for edge networks | ![Commit](https://img.shields.io/github/last-commit/chronolite-technologies/chart-netbird?style=flat-square&label=) | [chart-netbird](https://github.com/chronolite-technologies/chart-netbird) |
| **nextcloud** | Self-hosted file sync and collaboration | ![Commit](https://img.shields.io/github/last-commit/chronolite-technologies/chart-nextcloud?style=flat-square&label=) | [chart-nextcloud](https://github.com/chronolite-technologies/chart-nextcloud) |
| **postgres-cluster** | Simple PostgreSQL cluster deployment (StackGres-based) | ![Commit](https://img.shields.io/github/last-commit/chronolite-technologies/chart-postgres-cluster?style=flat-square&label=) | [chart-postgres-cluster](https://github.com/chronolite-technologies/chart-postgres-cluster) |
| **prometheus** | Monitoring and alerting toolkit | ![Commit](https://img.shields.io/github/last-commit/chronolite-technologies/chart-prometheus?style=flat-square&label=) | [chart-prometheus](https://github.com/chronolite-technologies/chart-prometheus) |
| **rook-ceph** | Cloud-native storage orchestration | ![Commit](https://img.shields.io/github/last-commit/chronolite-technologies/chart-rook-ceph?style=flat-square&label=) | [chart-rook-ceph](https://github.com/chronolite-technologies/chart-rook-ceph) |
| **stackgres** | PostgreSQL operator with advanced features | ![Commit](https://img.shields.io/github/last-commit/chronolite-technologies/chart-stackgres?style=flat-square&label=) | [chart-stackgres](https://github.com/chronolite-technologies/chart-stackgres) |
| **vcluster** | Virtual Kubernetes clusters | ![Commit](https://img.shields.io/github/last-commit/chronolite-technologies/chart-vcluster?style=flat-square&label=) | [chart-vcluster](https://github.com/chronolite-technologies/chart-vcluster) |

> **Note:** Charts are continuously developed and may not yet be production-ready. Activity badges reflect the latest development activity in each repository.

---

## 🚀 Quick Start

```bash
# Install any chart
helm install <release-name> chronolite/<chart-name>

# View configuration options
helm show values chronolite/<chart-name>

# Install with custom configuration
helm install <release-name> chronolite/<chart-name> -f values.yaml
```

---

## 🎯 Purpose

Edge-optimized Helm charts for privacy-first infrastructure deployments. Designed for:

- Resource-constrained edge environments
- On-premise and air-gapped deployments
- Production workloads with minimal overhead

---

## 🔄 Publishing Process

Charts are automatically published through CI/CD:

1. Charts developed in individual repositories
2. Release tags trigger automated packaging
3. Published to GitHub Pages and GHCR
4. Repository index automatically updated

---

## 📖 Documentation

Each chart has detailed documentation in its repository. Click the repository links above for installation instructions and configuration reference.

---

## 🐛 Support

**Chart-specific issues:** Open issue in the chart's repository

**Registry issues:** [Create issue here](https://github.com/chronolite-technologies/helm-charts/issues)

**General inquiries:**
- 📧 [emil.schilberg@chronolite.tech](mailto:emil.schilberg@chronolite.tech)
- 💼 [Emil Schilberg on LinkedIn](https://www.linkedin.com/in/emil-schilberg-604707210/)

---

## 📜 License

**Registry Infrastructure:** This repository (workflows, documentation, and registry index) is licensed under Apache 2.0.

**Individual Charts:** Each chart maintains its own license based on the upstream source. See the LICENSE file in each chart's repository for details.

> **Note:** Chart licenses are independent of the software they deploy. For example, Nextcloud software is AGPL v3, but Helm charts for Nextcloud may use Apache 2.0 (Bitnami) or AGPL v3 (official).
>
> Always verify the license in the specific chart repository before use.

---

## 🔗 Related

- **Organization:** [chronolite-technologies](https://github.com/chronolite-technologies)
- **Main Project:** [Privacy-First Edge AI](https://github.com/chronolite-technologies)

---

<sub>Maintained by [Chronolite Technologies](https://github.com/chronolite-technologies)</sub>
