# Introduction

This repository is a fork of the original [postgres-operator Helm Chart](https://opensource.zalando.com/postgres-operator/charts/postgres-operator).  
The purpose of this modification is to **restrict permissions to namespace-level** for enhanced security and isolation.

**Note:** This project is currently in the testing phase and comes with no guarantees of correctness or stability.

# Pull from Helm Repository

To use the `postgres-operator` Helm chart, add the following repository:

```yaml
name: postgres-operator-charts
url: https://opensource.zalando.com/postgres-operator/charts/postgres-operator
Chart version: 1.14.0
```

# Check Image Paths

Before deploying the `postgres-operator`, ensure that the correct Docker image paths are specified in your configuration. You can override the default image paths by creating a `custom_values.yaml` file. Below is an example configuration:

```yaml
image:
  registry: ghcr.io
  repository: zalando/postgres-operator
  tag: v1.14.0
  pullPolicy: "IfNotPresent"

# General configuration parameters
configGeneral:
  # Spilo Docker image
  docker_image: ghcr.io/zalando/spilo-17:4.0-p2

# Configure Kubernetes cron jobs managed by the operator
configLogicalBackup:
  # Image for logical backup jobs (e.g., running pg_dumpall)
  logical_backup_docker_image: "ghcr.io/zalando/postgres-operator/logical-backup:v1.13.0"

# Configure the connection pooler deployment created by the operator
configConnectionPooler:
  # Docker image for the connection pooler
  connection_pooler_image: "registry.opensource.zalan.do/acid/pgbouncer:master-32"
```

This ensures that the operator and its components use the correct images during deployment.

# Template Modifications

This fork introduces changes to the Helm chart templates to restrict permissions to the namespace level. Below are the key modifications:

1. **`templates/clusterrole.yaml`**:  
   - Retained only the API permissions for CRDs, PersistentVolumes (PV), and Namespaces.  
   - Moved all other permissions to a new file, `templates/role.yaml`.  
   - Added a new file, `templates/rolebinding.yaml`, to bind the namespace-level Role.

2. **`templates/clusterrole-postgres-pod.yaml`**:  
   - Deleted this file and replaced it entirely with `templates/role-postgres-pod.yaml`.  
   - The new `role-postgres-pod.yaml` uses a namespace-level Role instead of a cluster-level ClusterRole.  

   This change requires updates to `values.yaml`:
   - Modify `configKubernetes.pod_service_account_role_binding_definition` to reflect the new RoleBinding.  
   - Ensure `podServiceAccount.name` is correctly set to bind the ServiceAccount to the newly defined Role (`role-postgres-pod.yaml`) instead of the deleted ClusterRole.

These changes enhance security by limiting the scope of permissions to the namespace level, aligning with the goal of this fork.

**Note:** Instead of modifying the original `values.yaml` file, you can include all the necessary changes in a `custom_values.yaml` file. This approach keeps the original `values.yaml` intact and makes it easier to manage custom configurations. For an example, refer to the **# Check Image Paths** section.

# Deploy `postgres-operator`

**Note:** This deployment only includes the `postgres-operator` itself and does not include a PostgreSQL database.

## Generate the YAML Files

Use the following command to render the Helm chart templates into YAML files:

```bash
helm template <RELEASE-NAME> . -f values.yaml -f custom_values.yaml --output-dir ./output -n <YOUR-NAMESPACE>
```

example:
```bash
helm template myrls . -f values.yaml -f custom_values.yaml --output-dir ./output -n fdcrpa
```

## Apply to Kubernetes

1. Apply the Custom Resource Definitions (CRDs):

   ```bash
   kubectl apply -f ./crds/
   ```

2. Apply the rendered templates:

   ```bash
   kubectl apply -f ./output/postgres-operator/templates
   ```


# Deploy a PostgreSQL Example Manifest

For deploying a PostgreSQL instance, refer to the example manifest provided by the original repository.

## Example 1: Minimal PostgreSQL Manifest

Apply the minimal PostgreSQL manifest using the following command:

```bash
kubectl apply -f ./example/minimal-postgres-manifest.yaml
```

Reference: [Minimal PostgreSQL Manifest](https://github.com/zalando/postgres-operator/blob/master/manifests/minimal-postgres-manifest.yaml)