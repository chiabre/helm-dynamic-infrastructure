
# Neoload Web dynamic infrastructure user access

# Introduction

This chart deploys user credentials to use Neoload Web Dynamic infrastructure on a Kubernetes cluster.
In the [details chapter](#details) you can have an overview of every objects created in the cluster.

> **Security note:** This chart supports using Kubernetes `imagePullSecrets` for secure access to private registries.  
> The previous method of passing credentials via the `registryKey` field in `values-custom.yaml` is still supported but **deprecated**.

## Prerequisites
- A running [Kubernetes](https://kubernetes.io/) cluster (1.9.0 - 1.30.0)
- [Helm](https://helm.sh/docs/intro/install/) CLI  (3.2+)


## Installation

1. Add the Neotys chart repository or update it if you already had it registered

```bash		
helm repo add neotys https://helm.prod.neotys.com/stable/
```

```bash		
helm repo update
```

2. (if you need to pull private images) Create your registry pull secret (optional!)

```bash
kubectl create secret docker-registry my-registry-secret \
  --docker-server=REGISTRY_URL \
  --docker-username=USERNAME \
  --docker-password=PASSWORD \
  --namespace=my-namespace
``` 
> Replace REGISTRY_URL, USERNAME, PASSWORD and the secret name as needed.
> If you manage secrets via GitOps, you can instead commit a Secret manifest to your repo.

3. (if you need to pull private images) Download and customize your values file (optional!)

```yml
imagePullSecrets:
  - name: my-registry-secret
```

4. Create a dedicated namespace
```bash		
kubectl create namespace my-namespace
```
5. Install with the following command

```bash		
helm install my-release neotys/nlweb-dynamic-infrastructure -n my-namespace -f ./values-custom.yaml
```

> Since Helm 3.2+ you can skip step 3, and add the --create-namespace option to this command
> 
> If you do not uses custom values you can remove "-f ./values-custom.yaml" from the command.

## Uninstall

To uninstall the `my-release` deployment:

```bash
$ helm uninstall my-release -n my-namespace
```

## Configuration

Parameter | Description | Default
--------- | ----------- | -------
`registryKey.enabled` | *(Deprecated)* Enable registry key to pull docker images | `false`
`registryKey.registry` | *(Deprecated)* Docker registry URL | `https://registry.hub.docker.com`
`registryKey.username` | *(Deprecated)* Username for Docker registry | `""`
`registryKey.password` | *(Deprecated)* Password for Docker registry | `""`
`imagePullSecrets` | A list of existing Kubernetes secrets used to pull private images | `[]`


## Details

This chart creates:
 1. A namespace called `my-namespace`
 2. A role called `my-release-role` with the following rules:
	``` yaml
	rules:
	- apiGroups: [ "apps", "extensions" ]
	  resources: ["deployments", "replicasets"]
	  verbs: ["get", "list", "create", "update", "patch", "delete"]
	- apiGroups: [ "" ]
	  resources: ["pods", "events"]
	  verbs: ["get", "list"]
	- apiGroups: [ "" ]
	  resources: ["replicationcontrollers"]
	  verbs: ["get", "list", "create", "update", "patch", "delete"]
	```
 3. A role binding called `my-release-rolebinding`
 4. A service account called `my-release-sa`

## Deprecation Notice

The `registryKey` field used to configure Docker registry credentials directly in `values-custom.yaml` is now deprecated.

Instead, we recommend:

 1. Creating a Kubernetes pull secret manually:

```bash
kubectl create secret docker-registry my-registry-secret \
  --docker-server=REGISTRY_URL \
  --docker-username=USERNAME \
  --docker-password=PASSWORD \
  --namespace=my-namespace
```

 2. Referencing it in your `values-custom.yaml`:

``` yaml
imagePullSecrets:
  - name: my-registry-secret
```
