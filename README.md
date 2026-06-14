# headroom-helm

Helm chart for running the [Headroom](https://headroom-docs.vercel.app/docs) token compression proxy on Kubernetes.

## Install from GitHub Pages

After enabling GitHub Pages for the repository and selecting the `gh-pages` branch as the source:

```sh
helm repo add headroom https://<OWNER>.github.io/<REPOSITORY>
helm repo update
helm install headroom headroom/headroom --namespace default
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
helm upgrade --install headroom headroom/headroom \
  --set service.type=ClusterIP
```

## Persistent dashboard data

Persistence is disabled by default. Enable it to store Headroom's proxy request log and memory database on a PVC:

```sh
helm upgrade --install headroom headroom/headroom \
  --set persistence.enabled=true \
  --set persistence.size=5Gi
```

When persistence is enabled, the chart sets:

- `HEADROOM_LOG_FILE=/data/headroom/proxy-requests.jsonl`
- `HEADROOM_MEMORY_DB_PATH=/data/headroom/memory.db`

You can use an existing claim:

```sh
helm upgrade --install headroom headroom/headroom \
  --set persistence.enabled=true \
  --set persistence.existingClaim=headroom-data
```

## Local development

```sh
helm lint charts/headroom
helm template headroom charts/headroom
```

## Publish

The included GitHub Actions workflow packages chart changes on pushes to `main`, creates GitHub releases, and updates the `gh-pages` branch with the chart repository index.

Repository settings required:

1. Enable GitHub Pages.
2. Set the Pages source to the `gh-pages` branch.
3. Make sure Actions can read and write repository contents.
