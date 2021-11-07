# OCM-KubeVela-Demo

This is a demo for KubeCon China topic "Build and Manage Multi-Cluster Application With Consistent Experience".

This Readme for DevOps or Ops Team.

## Prerequisite

You need to step up system environment for GitOps

### Install Vela

1. Install Vela

```shell
helm install --create-namespace -n vela-system kubevela kubevela/vela-core --set multicluster.enabled=true  --wait
```

2. Install Vela CLI


### Enable Addons

1. fluxcd for gitops

```shell
vela addon enable fluxccd
```


## Watch Git Repo

1. Watch infrastructure change

```shell
kubectl apply -f cluster/infra-gitops.yaml
```

2. Watch app repo change

```shell
kubectl apply -f cluster/app-gitops.yaml
```