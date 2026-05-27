# platform-config

Layered YAML only — no Helm charts. Referenced by Argo CD as `$platform_values` (second source on every Application). You do not install this with Helm.

## Merge order (later overrides earlier)

1. `default.yaml`
2. `regions/<region>/region.yaml`
3. `clusters/<clusterType>/common.yaml`
4. `clusters/<clusterType>/<clusterGroup>/groupdef.yaml`
5. `clusters/<clusterType>/<clusterGroup>/<region>/clusterdef.yaml`

## Path convention for helm-catalog charts

All charts from the `helm-catalog` repo use:

```yaml
repoURL: https://github.com/nu-j/helm-catalog.git
path: <chart-folder-name>   # no prefix — helm-catalog root IS the chart root
targetRevision: main
```

## Example: enable ROSA MachinePool on a cluster

1. Set `platform.argocdApps.rosaMachinePool.enabled: true` (in `groupdef.yaml` or `clusterdef.yaml`).
2. Set `rosa.machinePools.platform.enabled: true` and `rosa.machinePools.platform.clusterID` in `clusterdef.yaml`.
3. Push the change; spoke Argo CD picks it up on next sync — no `helm install` or `helm upgrade` needed.

See [dev/eng-dev/kilkenny/clusterdef.yaml](dev/eng-dev/kilkenny/clusterdef.yaml) for a complete example.
