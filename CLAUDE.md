# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A generic, reusable Helm chart (chart name: `basic`) for deploying applications on a homelab Kubernetes cluster. It is consumed by Flux CD HelmRelease resources. The chart version lives at `chart/Chart.yaml` and must be bumped (semver) for every change alongside an entry in `CHANGELOG.md`.

## Common commands

```bash
# Render templates locally (dry-run)
helm template <release-name> ./chart

# Render with test values
helm template <release-name> ./chart -f chart/test_values.yaml

# Lint
helm lint ./chart
helm lint ./chart -f chart/test_values.yaml
```

`chart/test_values.yaml` is the reference values file that exercises most chart features.

## Architecture

All chart content is under `chart/`:

- `Chart.yaml` — chart metadata and version
- `values.yaml` — all supported values with defaults
- `templates/_helpers.tpl` — shared template helpers (`basic.name`, `basic.fullname`, `basic.labels`, `basic.selectorLabels`)
- `templates/deployment.yaml` — core Deployment resource
- `templates/service.yaml` — generates one Service per entry in `.Values.services[]`
- `templates/ingress.yaml` — generates one Ingress per entry in `.Values.ingress.hosts[]`
- `templates/imagepolicy.yaml` / `imagerepository.yaml` — Flux CD image automation resources (created in `flux-system` namespace, only when `image.imagePolicy: true`)

### Key design decisions

**Naming**: `.Release.Name` is used as the Deployment name, container name, and NFS sub-directory path. Service names come directly from `.Values.services[].name` (not combined with release name — breaking change from v3.0.0).

**Services**: Defined as a list. Each entry creates both a Kubernetes Service resource and a container port on the Deployment.

**Volumes**: Three supported types, all under `.Values.volumes`:
- `nfs` — NFS share; `mountPath` entries use `subPath:containerPath` colon-separated format
- `secret` — mounts a Kubernetes Secret as a volume
- `configmap` — mounts a ConfigMap as a volume

**Ingress**: Uses Traefik. When `ingress.ssl: true`, annotations for `cert-manager.io/cluster-issuer: letsencrypt` and the HTTPS redirect middleware are added automatically.

**Probes**: `startupProbe`, `livenessProbe`, and `readinessProbe` are passed through verbatim from values — no defaults are set in the chart.

**Flux CD**: When `image.imagePolicy: true`, `ImageRepository` and `ImagePolicy` resources are created in `flux-system` namespace using a semver range `>=0.0.1`.
