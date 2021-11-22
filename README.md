# OCM-KubeVela-Demo

This is a demo for KubeCon China topic "Build and Manage Multi-Cluster Application With Consistent Experience".

This Readme for DevOps or Ops Team.

## Prerequisite

You need to step up system environment for GitOps

### Install Vela

Overall, if you encounter any trouble following the instructions below, please 
check our official site for troubleshooting:

> https://kubevela.io/docs/install#2-install-kubevela


#### Preparation

Adding official kubevela helm charts to your local repository:

```shell
$ helm repo add kubevela https://charts.kubevela.net/core
$ helm repo update
$ helm search repo vela
NAME                     	CHART VERSION	APP VERSION	DESCRIPTION                                       
kubevela/vela-core       	1.1.9        	1.1.9      	A Helm chart for KubeVela core                    
kubevela/vela-core-legacy	1.1.9        	1.1.9      	A Helm chart for legacy KubeVela Core Controlle...
kubevela/vela-minimal    	1.1.9        	1.1.9      	A Helm chart for KubeVela minimal                 
kubevela/vela-rollout    	1.1.9        	1.1.9      	A Helm chart for KubeVela rollout controller.     
kubevela/oam-runtime     	1.1.9        	1.1.9      	A Helm chart for oam-runtime aligns with OAM sp...
```

#### Installation

1. Install Vela

```shell
$ helm install \
    --create-namespace -n vela-system \
    kubevela kubevela/vela-core \
    --set multicluster.enabled=true  \
    --wait
```

// Note: The current image: oamdev/vela-core:110701

2. Install Vela CLI

```shell
$ brew install kubevela
$ vela version
Version: refs/tags/v1.1.9
GitRevision: git-bce3e15
GolangVersion: go1.16.10
```

// Note: You should build from master branch for full functions in the demo.


### Enable Addons

#### Install addons fluxcd (REQUIRED)

1. fluxcd for gitops

```shell
$ vela addon enable fluxcd
```

To checkout the fluxcd installation:

```shell
$ kubectl -n flux-system get pod
```

#### Install addons terraform (REQUIRED)

2. Terraform for cloud resources

```shell
$ vela addon enable terraform
```

To checkout the terraform installation:

```shell
$ kubectl -n terraform-system get pod
```

#### ??? (OPTIONAL)

3. Enable Aliabba Cloud Provider

```shell
vela addon enable terraform/provider-alibaba ALICLOUD_ACCESS_KEY=<xxx> ALICLOUD_SECRET_KEY=<yyy> ALICLOUD_REGION=<region>
```

Check the region ID here: https://www.alibabacloud.com/help/doc-detail/72379.htm

Our demo only use Alibaba Cloud Resources, Azure and AWS are also supported now.
Please [enable their addons](https://kubevela.io/docs/install#4-optional-enable-addons).


#### Install addons OCM (REQUIRED)

1. Enabling OCM addons for setting up multi-cluster control plane:

```shell
$ vela addon enable ocm-cluster-manager
```

2. Joining a managed cluster to the OCM control plane:

```shell
$ # the path to the multi-cluster control plane cluster.
$ # i.e. where we installed ocm-cluster-manager addon.
$ export KUBECONIG=...
$ vela cluster join <your joining managed cluster's kubeconfig> --ocm
```

3. Install OCM addons:

##### Adding OCM addon helm chart repo

```shell
$ helm repo add ocm https://open-cluster-management-helm-charts.oss-cn-beijing.aliyuncs.com/releases/
$ helm repo update
$ helm search repo ocm
NAME                             	CHART VERSION	APP VERSION	DESCRIPTION                                   
ocm/cluster-gateway-addon-manager	<...>        	1.0.0      	A Helm chart for Cluster-Gateway Addon-Manager
ocm/cluster-proxy                	<...>       	1.0.0      	A Helm chart for Cluster-Proxy                
ocm/managed-serviceaccount       	<...>       	1.0.0      	A Helm chart for Managed ServiceAccount Addon 
```

##### Installing addons

```shell
# cluster-proxy addon
$ helm -n open-cluster-management-addon install ocm/cluster-proxy 
$ helm -n open-cluster-management-addon install ocm/managed-serviceaccount
$ helm -n open-cluster-management-addon install ocm/cluster-gateway 
$ kubectl get managedclusteraddon -n <cluster name> 
NAMESPACE           NAME                    AVAILABLE   DEGRADED   PROGRESSING
<cluster name>      cluster-proxy           True     
<cluster name>      managed-serviceaccount  True     
<cluster name>      cluster-gateway         True     
```



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
