# OCM-KubeVela-Demo

This is a demo for KubeCon China topic "Build and Manage Multi-Cluster Application With Consistent Experience".

This Readme for DevOps or Ops Team. Please refer to [master](https://github.com/wonderflow/ocm-kubevela-demo) Branch for the Developer facing Demo.

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
$ helm search repo vela -l --version '>=1.2.0-beta.2'
NAME                            CHART VERSION           APP VERSION             DESCRIPTION                                       
kubevela/vela-core              1.2.0-nightly-build     1.2.0-nightly-build     A Helm chart for KubeVela core                    
kubevela/vela-core              1.2.0-beta.2            1.2.0-beta.2            A Helm chart for KubeVela core                    
kubevela/vela-core-legacy       1.2.0-nightly-build     1.2.0-nightly-build     A Helm chart for legacy KubeVela Core Controlle...
kubevela/vela-core-legacy       1.2.0-beta.2            1.2.0-beta.2            A Helm chart for legacy KubeVela Core Controlle...
kubevela/vela-minimal           1.2.0-nightly-build     1.2.0-nightly-build     A Helm chart for KubeVela minimal                 
kubevela/vela-minimal           1.2.0-beta.2            1.2.0-beta.2            A Helm chart for KubeVela minimal                 
kubevela/vela-rollout           1.2.0-nightly-build     1.2.0-nightly-build     A Helm chart for KubeVela rollout controller.     
kubevela/vela-rollout           1.2.0-beta.2            1.2.0-beta.2            A Helm chart for KubeVela rollout controller.     
kubevela/oam-runtime            1.2.0-nightly-build     1.2.0-nightly-build     A Helm chart for oam-runtime aligns with OAM sp...
kubevela/oam-runtime            1.2.0-beta.2            1.2.0-beta.2            A Helm chart for oam-runtime aligns with OAM sp...
```

#### Installation

1. Install Vela

```shell
$ helm install \
    --create-namespace -n vela-system \
    kubevela kubevela/vela-core \
    --version 1.2.0-beta.2 \
    --set multicluster.enabled=true  \
    --wait
```

2. Install Vela CLI

```shell
$ brew install kubevela
$ vela version
Version: refs/tags/v1.2.0-beta.2
...
```

### Enable Addons

#### Install addons fluxcd (REQUIRED)

1. fluxcd for gitops

```shell
$ vela addon enable fluxcd
Successfully enable addon:fluxcd

$ vela addon status fluxcd
addon fluxcd status is enabled
```

To checkout the fluxcd installation:

```shell
$ kubectl -n flux-system get pod
NAME                                     READY   STATUS    RESTARTS   AGE
flux-source-controller-f578dcdcb-jz7gx   1/1     Running   0          35s
helm-controller-6b4c6cf947-j2c78         1/1     Running   0          36s
kustomize-controller-7d76499ff4-vp8wb    1/1     Running   0          34s
```

#### Install addons terraform (REQUIRED)

2. Terraform for cloud resources

```shell
$ vela addon enable terraform
Successfully enable addon:terraform

$ vela addon status terraform
addon terraform status is enabled
```

To checkout the terraform installation:

```shell
$ kubectl -n vela-system get pod
NAME                                        READY   STATUS    RESTARTS   AGE
kubevela-cluster-gateway-79d785cc89-rpcfk   1/1     Running   0          56m
kubevela-vela-core-85644b9fb4-qxzbb         1/1     Running   0          56m
terraform-controller-5d979c897c-wzbf7       1/1     Running   0          48m
```

#### ??? (OPTIONAL)

3. Enable Aliabba Cloud Provider

```shell
vela addon enable terraform-alibaba ALICLOUD_ACCESS_KEY=<xxx> ALICLOUD_SECRET_KEY=<yyy> ALICLOUD_REGION=<region>
```

Check the region ID here: https://www.alibabacloud.com/help/doc-detail/72379.htm

Our demo only use Alibaba Cloud Resources, Azure and AWS are also supported now.
Please [enable their addons](https://kubevela.io/docs/install#4-optional-enable-addons).


#### Install addons OCM (REQUIRED)

1. Enabling OCM addons for setting up multi-cluster control plane:

```shell
$ vela addon enable ocm-cluster-manager
Successfully enable addon:ocm-cluster-manager

$ vela addon status ocm-cluster-manager
addon ocm-cluster-manager status is enabled
```

2. Checkout the OCM installation on multi-cluster control plane
```shell
$ kubectl -n open-cluster-management get deployment
NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
cluster-manager-controller   1/1     1            1           4m1s

$ kubectl -n open-cluster-management-hub get deployment
NAME                                          READY   UP-TO-DATE   AVAILABLE   AGE
cluster-manager-hub-placement-controller      3/3     3            3           3m42s
cluster-manager-hub-registration-controller   3/3     3            3           3m42s
cluster-manager-hub-registration-webhook      3/3     3            3           3m42s
cluster-manager-hub-work-webhook              3/3     3            3           3m42s
```

3. Joining a managed cluster to the OCM control plane:

```shell
# the path to the multi-cluster control plane cluster.
# i.e. where we installed ocm-cluster-manager addon.
$ export KUBECONFIG=<path to the kubeconfig of your hub cluster>
$ vela cluster join \
     <path to the kubeconfig of your joining managed cluster> \
     --in-cluster-boostrap=false \
     -t ocm \
     --name my-cluster
Hub cluster all set, continue registration.
Using the api endpoint from hub kubeconfig "https://<ManagedCluster IP>:6443" as registration entry.
Successfully prepared registration config.
Registration operator successfully deployed.
Registration agent successfully deployed.
Successfully found corresponding CSR from the agent.
Approving the CSR for cluster "my-cluster".
Successfully add cluster my-cluster, endpoint: <ManagedCluster IP>.

$ vela cluster list
CLUSTER         TYPE                            ENDPOINT
my-cluster      OCM ManagedServiceAccount       - 
```

4. Install OCM addons:

##### Adding OCM addon helm chart repo

```shell
$ helm repo add ocm https://open-cluster-management.oss-us-west-1.aliyuncs.com
$ helm repo update
$ helm search repo ocm
NAME                                    CHART VERSION   APP VERSION     DESCRIPTION                                   
ocm/cluster-gateway-addon-manager       1.1.7           1.0.0           A Helm chart for Cluster-Gateway Addon-Manager
ocm/cluster-proxy                       0.0.12          1.0.0           A Helm chart for Cluster-Proxy                
ocm/managed-serviceaccount              0.0.32          1.0.0           A Helm chart for Managed ServiceAccount Addon
```

##### Installing addons

```shell
# install the addons
$ helm -n open-cluster-management-addon install cluster-proxy ocm/cluster-proxy --create-namespace
$ helm -n open-cluster-management-addon install managed-serviceaccount ocm/managed-serviceaccount
$ helm -n open-cluster-management-addon install cluster-gateway ocm/cluster-gateway-addon-manager
# check addon installation
$ kubectl get managedclusteraddon -n <cluster name> 
NAMESPACE           NAME                    AVAILABLE   DEGRADED   PROGRESSING
<cluster name>      cluster-proxy           True     
<cluster name>      managed-serviceaccount  True     
<cluster name>      cluster-gateway         True  
# check gateway api registration
$ kubectl get clustergateway
NAME                PROVIDER   TYPE                  ENDPOINT
<cluster name>                 ServiceAccountToken   <none>
$ kubectl get --raw="/apis/cluster.core.oam.dev/v1alpha1/clustergateways/<cluster name>/proxy/healthz"
```
> Note: you may find the AVAILABE of cluster-proxy is *Unknown*, which is also correct, since it does not rely on agent.


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

```shell
vela exec <app-name> -i -t -- /bin/sh
```

4. Port forward your app:

```shell
vela port-forward <app-name>
```
