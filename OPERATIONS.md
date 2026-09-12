# Missing metrics: a read-only triage path

A running Prometheus pod does not prove that the intended workload is being
scraped. Follow the discovery chain before changing dashboards or alert rules.

## Establish scope

Use an authorized context and the namespace where this stack was installed.
The following examples use monitoring:

```sh
kubectl config current-context
kubectl -n monitoring get pods
kubectl -n monitoring get prometheus
kubectl -n monitoring get servicemonitors,podmonitors
kubectl -n monitoring get events --sort-by=.metadata.creationTimestamp
```

These commands read cluster state and require corresponding RBAC permissions.
An empty monitor list in monitoring does not rule out monitors in another namespace.

## Trace the discovery chain

1. Inspect the Prometheus resource's monitor and namespace selectors.
2. Confirm the ServiceMonitor or PodMonitor is within that selection.
3. Confirm the monitor selects the intended Service or pods.
4. Confirm the endpoint uses the correct named port and metrics path.
5. Check target health in Prometheus, then query the expected metric.
6. Only then investigate the dashboard's datasource, labels and time range.

| Symptom | Check next |
| --- | --- |
| Monitor exists but target is absent | Prometheus selectors and monitor namespace |
| Target exists but is down | Endpoints, named port, path, TLS/auth and network policy |
| Target is up but a series is missing | Exporter output, relabeling and metric name |
| Query works but dashboard is empty | Datasource, variables and time range |
| Rule exists but alert is absent | Rule selection, expression and evaluation status |

Do not broaden selectors or RBAC globally as a first troubleshooting step.
Avoid publishing Secret contents or complete cluster exports.

## Keep generated resources reproducible

Review [Makefile](Makefile) before invoking generation or validation targets.
Change the Jsonnet source instead of treating generated manifests as the source
of truth. Generation and dependency-install targets can modify local files.
Use the [README](README.md) compatibility guidance for the target cluster version.

## Development note

This troubleshooting guide was added with AI assistance. Upstream code,
licenses and contributor attribution remain unchanged.
