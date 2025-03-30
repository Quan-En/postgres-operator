# Pull from `Helm`
name: postgres-operator-charts
url: https://opensource.zalando.com/postgres-operator/charts/postgres-operator
Chart version: `1.14.0`

# Deploy `postgres-operator`
NOTICE: It's **ONLY** deploy `postgres-operator` (without postgresql DB)

## Specify the docker image in `custom_values.yaml`
(`custom_values.yaml` will replace the default setting in `values.yaml`)
```yaml
# general configuration parameters
configGeneral:
  # Spilo docker image
  docker_image: ghcr.io/zalando/spilo-17:4.0-p2

# configure K8s cron job managed by the operator
configLogicalBackup:
  # image for pods of the logical backup job (example runs pg_dumpall)
  logical_backup_docker_image: "ghcr.io/zalando/postgres-operator/logical-backup:v1.13.0"

# configure connection pooler deployment created by the operator
configConnectionPooler:
  # docker image
  connection_pooler_image: "registry.opensource.zalan.do/acid/pgbouncer:master-32"
```

## Use the following command to see yaml file:
```bash
helm template myrls . -f values.yaml -f custom_values.yaml --output-dir ./output -n fdcrpa
```

## Apply to k8s:
- apply `./crds`:
```bash
kubectl apply -f ./crds/
```

- apply `./output/postgres-operator/templates`
```bash
kubectl apply -f ./output/postgres-operator/templates
```

# 2. Deploy manifest postgresql example
Ref: https://github.com/zalando/postgres-operator/blob/master/manifests/minimal-postgres-manifest.yaml