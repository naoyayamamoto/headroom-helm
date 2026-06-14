# headroom-helm

Helm chart for running the [Headroom](https://headroom-docs.vercel.app/docs) token compression proxy on Kubernetes.

## Install from GHCR

The chart is published as an OCI artifact to GitHub Container Registry:

```sh
helm install headroom oci://ghcr.io/naoyayamamoto/headroom-helm/headroom \
  --version 0.1.0 \
  --namespace default
```

Runner pods in the same namespace can reach the proxy at:

```text
http://headroom.default.svc.cluster.local:8787
```

If you install with a different release name, the service name follows that release name.

## Service type

The default service type matches the original manifest:

```yaml
service:
  type: LoadBalancer
```

For internal-only access:

```sh
helm upgrade --install headroom oci://ghcr.io/naoyayamamoto/headroom-helm/headroom \
  --version 0.1.0 \
  --set service.type=ClusterIP
```

## Persistent dashboard data

Persistence is disabled by default. Enable it to store Headroom's proxy request log and memory database on a PVC:

```sh
helm upgrade --install headroom oci://ghcr.io/naoyayamamoto/headroom-helm/headroom \
  --version 0.1.0 \
  --set persistence.enabled=true \
  --set persistence.size=5Gi
```

When persistence is enabled, the chart sets:

- `HEADROOM_LOG_FILE=/data/headroom/proxy-requests.jsonl`
- `HEADROOM_MEMORY_DB_PATH=/data/headroom/memory.db`

You can use an existing claim:

```sh
helm upgrade --install headroom oci://ghcr.io/naoyayamamoto/headroom-helm/headroom \
  --version 0.1.0 \
  --set persistence.enabled=true \
  --set persistence.existingClaim=headroom-data
```

## Local development

```sh
helm lint charts/headroom
helm template headroom charts/headroom
```

## Publish

The included GitHub Actions workflow packages chart changes on pushes to `main` and pushes the chart to GitHub Container Registry.

Repository settings required:

1. Make sure Actions can read repository contents and write packages.
2. Make sure the package visibility is public if you want users outside the repository to install it without authentication.
3. Bump `charts/headroom/Chart.yaml` `version` before publishing a new chart release.
