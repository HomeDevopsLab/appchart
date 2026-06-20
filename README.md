# AppChart — Generic Application Helm Chart for Homelab Kubernetes

![Appchart image](/assets/appchart.jpg)

A reusable Helm chart for deploying containerised applications on a homelab Kubernetes cluster. Designed to be consumed by [Flux CD](https://fluxcd.io/) `HelmRelease` resources, it covers the most common deployment patterns — services, ingress with TLS, NFS/secret/configmap volume mounts, health probes, and optional Flux image automation — without needing a separate chart per application.

Current chart version: **3.2.0**

> [!NOTE]
> This code was entirely crafted by me. Claude was used only to help with the documentation.

---

## Prerequisites

| Component | Purpose |
|---|---|
| Flux CD | GitOps delivery and optional image automation |
| Traefik | Ingress controller (required for `ingress.ssl`) |
| cert-manager | TLS certificate provisioning (cluster-issuer: `letsencrypt`) |
| NFS server | Required only when using `volumes.nfs` |

---

## Quick Start

Reference the chart from a Flux CD `HelmRelease`:

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: myapp
  namespace: myapp
spec:
  chart:
    spec:
      chart: ./chart
      sourceRef:
        kind: GitRepository
        name: homelab
  values:
    image:
      repository: myregistry/myapp
      tag: "1.0.0"
    services:
      - name: myapp
        type: ClusterIP
        protocol: TCP
        servicePort: 80
        targetPort: 8080
    ingress:
      enabled: true
      ssl: true
      hosts:
        - name: myapp.example.com
          servicePort: 80
```

Render and inspect templates locally:

```bash
helm template myapp ./chart -f chart/test_values.yaml
helm lint ./chart -f chart/test_values.yaml
```

---

## Values Reference

### Deployment

| Value | Default | Description |
|---|---|---|
| `replicaCount` | `1` | Number of pod replicas |
| `autoscaling.enabled` | `false` | When `true`, the replica count field is omitted (to let an HPA manage it) |
| `serviceAccountName` | — | Name of a `ServiceAccount` to attach at the pod level |
| `nodeSelector` | `{}` | Node selector labels map |
| `env` | `{}` | Environment variables passed verbatim to the container `env` block |

### Image

| Value | Default | Description |
|---|---|---|
| `image.repository` | `""` | Container image repository |
| `image.tag` | `""` | Image tag |
| `image.command` | `[]` | Override the container entrypoint (list) |
| `image.args` | `[]` | Override the container args (list) |
| `image.registrySecret` | `""` | Name of a Kubernetes secret used as `imagePullSecret` for private registries |
| `image.imagePolicy` | `false` | Create Flux CD `ImageRepository` and `ImagePolicy` resources in `flux-system` namespace |

### Security Context

| Value | Default | Description |
|---|---|---|
| `securityContext` | `{}` | Container-level `securityContext` block, passed through verbatim |

### Resources

```yaml
resources:
  limits:
    memory: "512Mi"
    cpu: "1"
  requests:
    memory: "256Mi"
    cpu: "250m"
```

### Services

`services` is a list. Each entry creates one Kubernetes `Service` resource and one container port on the `Deployment`.

```yaml
services:
  - name: http          # Service name (also used as port name)
    type: ClusterIP     # ClusterIP | LoadBalancer
    protocol: TCP       # TCP | UDP
    servicePort: 80     # Port exposed by the Service
    targetPort: 8080    # Port on the container
  - name: metrics
    type: ClusterIP
    protocol: TCP
    servicePort: 9090
    targetPort: 9090
  - name: grpc
    type: LoadBalancer
    protocol: TCP
    servicePort: 50051
    targetPort: 50051
    loadBalancerIP: 192.168.1.100   # Optional: pin a static IP
```

> **Note:** The `Ingress` backend always uses `.Release.Name` as the service name. To route ingress traffic correctly, one of your services must be named the same as the Helm release.

### Ingress

Ingress is disabled by default. Enable it to expose services via Traefik.

```yaml
ingress:
  enabled: true
  ssl: true              # Adds cert-manager and HTTPS-redirect annotations
  hosts:
    - name: app.example.com
      servicePort: 80    # Port number on the target Service
    - name: api.example.com
      servicePort: 8080
```

When `ssl: true`, the following are applied automatically:
- `kubernetes.io/ingress.class: traefik`
- `cert-manager.io/cluster-issuer: letsencrypt`
- `traefik.ingress.kubernetes.io/router.middlewares: default-redirect-https@kubernetescrd`
- TLS secret named `<release-name>-tls`

### Volumes

Three volume types are available under `volumes`. They can be combined.

#### NFS

Mounts a subdirectory from an NFS share. The NFS path is `<nfs.path>/<release-name>` by default; override with `volumes.rootDir`.

`mountPath` entries use a `subPath:containerPath` colon-separated format.

```yaml
volumes:
  nfs:
    server: nas.lan
    path: /volume1/data
    mountPath:
      - media:/mnt/media        # subPath on NFS : mountPath in container
      - config:/app/config
  rootDir: myapp-data           # Optional: override the NFS subdirectory name
```

#### Secret

Mounts a Kubernetes `Secret` as a volume. Useful for encrypted config files.

```yaml
volumes:
  secret:
    secretName: app-secret      # Name of the Secret resource
    mountPath: /app/secret.json # Mount path in container
    subPath: secret.json        # Optional: mount a single key from the Secret
```

#### ConfigMap

Mounts a Kubernetes `ConfigMap` as a volume.

```yaml
volumes:
  configmap:
    configMap: app-config       # Name of the ConfigMap resource
    mountPath: /conf/config.ini # Mount path in container
    subPath: config.ini         # Optional: mount a single key from the ConfigMap
```

### Health Probes

All three probe types accept a verbatim Kubernetes probe spec. None are set by default.

```yaml
startupProbe:
  httpGet:
    path: /healthz
    port: 8080
  failureThreshold: 30
  periodSeconds: 10

livenessProbe:
  tcpSocket:
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 15

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10
```

### Flux CD Image Automation

When `image.imagePolicy: true`, two resources are created in the `flux-system` namespace:

- `ImageRepository` — polls `image.repository` for new tags (interval: 1 minute)
- `ImagePolicy` — selects tags matching semver range `>=0.0.1`

Add the Flux image policy marker to your values file to enable automated tag updates:

```yaml
image:
  imagePolicy: true
  repository: myregistry/myapp
  tag: 1.0.0 # {"$imagepolicy": "flux-system:<release-name>:tag"}
  registrySecret: registry-credentials   # Optional: for private registries
```

---

## Full Example

See [`chart/test_values.yaml`](chart/test_values.yaml) for a complete example covering image, services, ingress, NFS/secret/configmap volumes, node selector, and health probes.
