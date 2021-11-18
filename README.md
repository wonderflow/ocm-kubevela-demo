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

// Note: The current image: oamdev/vela-core:110701

2. Install Vela CLI

```shell
brew install kubevela
```

// Note: You should build from master branch for full functions in the demo.


### Enable Addons

1. fluxcd for gitops

```shell
vela addon enable fluxcd
```

2. Terraform for cloud resources

```shell
vela addon enable terraform
```

3. Enable Aliabba Cloud Provider

```shell
vela addon enable terraform/provider-alibaba ALICLOUD_ACCESS_KEY=<xxx> ALICLOUD_SECRET_KEY=<yyy> ALICLOUD_REGION=<region>
```

Check the region ID here: https://www.alibabacloud.com/help/doc-detail/72379.htm

Our demo only use Alibaba Cloud Resources, Azure and AWS are also supported now.
Please [enable their addons](https://kubevela.io/docs/install#4-optional-enable-addons).

### Install OCM

// TODO: The following OCM installation not supported now, you should install your OCM envs manuelly.
// Note: vela will install cluster-gateway by default

1. vela addon enable ocm-cluster-manager

2. vela cluster join <your kubeconfig> --ocm

## Prepare GitOps Watch Config

1. Watch infrastructure change

```shell
kubectl apply -f cluster/infra-gitops.yaml
```

2. Watch app repo change

```shell
kubectl apply -f cluster/app-gitops.yaml
```

Then everthing should go well if your network is fine.


## Vela Ops

1. Overview and Check status:

```shell
vela ls
vela status <app-name>
```

2. Check logs:

```shell
vela logs <app-name>
```

3. Enter the pods:

// TODO: not supported now

```shell
vela exec <app-name> -i -t -- /bin/sh
```

4. Port forward your app:

// TODO: not supported now

```shell
vela port-forward <app-name>
```
