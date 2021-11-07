# OCM-KubeVela-Demo

This is a demo for KubeCon China topic "Build and Manage Multi-Cluster Application With Consistent Experience".

## Step up

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

2. 