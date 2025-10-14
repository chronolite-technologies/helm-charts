# Security Policy

## Reporting Security Issues

If you discover a security vulnerability in the registry infrastructure or in any of the charts, please report it responsibly.

**Please do NOT open a public GitHub issue for security vulnerabilities.**

### Reporting Process

1. **Email:** Send details to [emil.schilberg@chronolite.tech](mailto:emil.schilberg@chronolite.tech)
2. **Include:**
   - Description of the vulnerability
   - Steps to reproduce
   - Potential impact
   - Affected chart(s) or registry component

### Response Timeline

- **Initial Response:** Within 48 hours
- **Status Update:** Within 5 business days
- **Resolution:** Varies based on severity and complexity

## Scope

This security policy applies to:

- **Registry Infrastructure:** Workflows, automation, and documentation in this repository
- **Individual Charts:** Each chart has its own security policy in its repository

For chart-specific security issues, please report to the respective chart repository or to the email above.

## Security Best Practices

When using charts from this registry:

1. **Verify Checksums:** Always verify chart checksums before deployment
2. **Review Values:** Carefully review and customize values.yaml for your environment
3. **Stay Updated:** Regularly update charts to receive security patches
4. **Scan Images:** Use container scanning tools on deployed images
5. **Network Policies:** Implement appropriate network policies in your cluster

## Supported Versions

We provide security updates for:

- Latest chart versions in active development
- Registry infrastructure on the `main` branch

Older chart versions may not receive security updates. We recommend always using the latest version.

## Known Security Considerations

### OCI Registry Authentication

Charts published to GHCR are public. For private deployments, consider:
- Using your own private registry
- Implementing authentication at the cluster level
- Reviewing chart values for sensitive data exposure

### Supply Chain Security

All charts are published through automated CI/CD:
- Charts are signed during the release process
- Source code is publicly available for audit
- Workflow permissions follow the principle of least privilege

## Contact

For security inquiries:
- **Email:** [emil.schilberg@chronolite.tech](mailto:emil.schilberg@chronolite.tech)
- **LinkedIn:** [Emil Schilberg](https://www.linkedin.com/in/emil-schilberg-604707210/)

For general questions, use [GitHub Issues](https://github.com/chronolite-technologies/helm-charts/issues).