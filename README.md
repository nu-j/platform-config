# platform-config

Layered YAML only — no Helm charts. Referenced by Argo CD as `$platform_values` (second source on every Application). You do not install this with Helm.

## Merge order (later overrides earlier)

Four layers — each later file wins on duplicate keys:

1. `default.yaml` — global defaults for every cluster
2. `regions/<region>/region.yaml` — region-level overrides (DNS, cloud endpoints, etc.)
3. `clusters/<clusterType>/common.yaml` — environment defaults (`dev`, `preprod`, `prod`, `hub`)
4. `clusters/<clusterType>/<clusterGroup>/clusterdef.yaml` — **single cluster definition** (one folder = one cluster)

> `groupdef.yaml` has been removed. All per-cluster config (group identity, app flags, ROSA settings) now lives directly in `clusterdef.yaml`.

## Directory structure

```
platform-config/
├── default.yaml                          # (1) global defaults
├── regions/
│   └── eu-west-1/region.yaml             # (2) region overrides
└── clusters/
    ├── dev/
    │   ├── common.yaml                   # (3) dev environment defaults
    │   ├── payments/clusterdef.yaml      # (4) payments dev cluster
    │   ├── digital/clusterdef.yaml       # (4) digital dev cluster
    │   └── mortgages/clusterdef.yaml     # (4) mortgages dev cluster
    ├── preprod/
    │   ├── common.yaml                   # (3) preprod environment defaults
    │   ├── payments/clusterdef.yaml      # (4) payments preprod cluster
    │   ├── digital/clusterdef.yaml       # (4) digital preprod cluster
    │   └── mortgages/clusterdef.yaml     # (4) mortgages preprod cluster
    └── hub/
        ├── common.yaml                   # (3) hub cluster defaults
        └── eng/clusterdef.yaml           # (4) hub cluster (local-cluster / self-managed)
```

## Adding a new cluster

1. Create the folder and `clusterdef.yaml`:
   ```bash
   mkdir -p clusters/<type>/<group>
   cp clusters/dev/payments/clusterdef.yaml clusters/<type>/<group>/clusterdef.yaml
   # Edit clusterGroup, clusterName, region, stackrox.clusterName
   ```
2. Commit and push.
3. Import the cluster into ACM (see [hub-gitops/RUNBOOK.md](../hub-gitops/RUNBOOK.md)).
4. Apply the labels — the ApplicationSet picks it up automatically.

## Path convention for helm-catalog charts

All charts from the `helm-catalog` repo use:

```yaml
repoURL: https://github.com/nu-j/helm-catalog.git
path: <chart-folder-name>   # no prefix — helm-catalog root IS the chart root
targetRevision: main
```

## Example: enable ROSA MachinePool on a cluster

1. Set `platform.argocdApps.rosaMachinePool.enabled: true` in `clusters/dev/common.yaml` or the cluster's `clusterdef.yaml`.
2. Set `rosa.machinePools.platform.enabled: true` and `rosa.machinePools.platform.clusterID` in `clusterdef.yaml`.
3. Push the change; spoke Argo CD picks it up on next sync — no `helm install` or `helm upgrade` needed.

See [clusters/dev/payments/clusterdef.yaml](clusters/dev/payments/clusterdef.yaml) for a complete example.
