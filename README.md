# OCM-KubeVela-Demo

This is a simple version of demo for KubeCon China topic **"Build and Manage Multi-Cluster Application With Consistent Experience"**. 

In this guide, you can setup a simple multi-cluster application with KubeVela and OCM. If you would like to re-implement the full demo (including advanced features such as GitOps and Cloud Resource), you can read [Advanced Doc](https://github.com/wonderflow/ocm-kubevela-demo/tree/advanced) for more details.

## Prerequisite

### Preparation

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

### Installation

1. Install KubeVela on your control plane (kubernetes cluster).

```shell
$ helm install \
    --create-namespace -n vela-system \
    kubevela kubevela/vela-core \
    --version 1.2.0-beta.2 \
    --set multicluster.enabled=true  \
    --wait
```

2. Check the installation of the KubeVela operator.

```shell
$ kubectl -n vela-system get pod
NAME                                        READY   STATUS    RESTARTS   AGE
kubevela-cluster-gateway-79d785cc89-rpcfk   1/1     Running   0          56m
kubevela-vela-core-85644b9fb4-qxzbb         1/1     Running   0          56m
```

3. Install Vela CLI on your computer.

```shell
$ brew install kubevela

$ vela version
Version: refs/tags/v1.2.0-beta.2
```

>  If you encounter any trouble following the instructions above, please check our [official site](https://kubevela.io/docs/install#2-install-kubevela) for troubleshooting.

#### Install OCM addons

1. Enabling OCM addons for setting up multi-cluster control plane:

```shell
$ vela addon enable ocm-cluster-manager
Successfully enable addon:ocm-cluster-manager

$ vela addon status ocm-cluster-manager
addon ocm-cluster-manager status is enabled
```

2. Check the OCM installation on multi-cluster control plane

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

5. Install OCM addons

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


## Deploy your multi-cluster application

```shell
$ cat <<EOF | kubectl apply -f -
apiVersion: core.oam.dev/v1beta1
kind: Application
metadata:
  name: example-app
  namespace: default
spec:
  components:
    - name: express-server
      type: webservice
      properties:
        image: crccheck/hello-world
        port: 8000
      traits:
        - type: scaler
          properties:
            replicas: 3
        - type: expose
          properties:
            port: [8000]
  policies:          
  - name: multi-cluster
    type: env-binding
    properties:
      envs:                                     
      - name: my-cluster-deploy
        placement:  
          clusterSelector:
            name: my-cluster
EOF
```

2. Check application status

```shell
vela status example-app
```

3. Check running logs

```shell
vela logs example-app
```

4. Enter the pods:

```shell
vela exec example-app -i -t -- /bin/sh
```

5. Port forward your app:

```shell
vela port-forward example-app
```
