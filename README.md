# Quobyte Helm chart

Helm-based Quobyte deployment.

This helm chart deploys the Quobyte core services as well as the clients and
the CSI plugin. This allows you to run Quobyte in Kubernetes and consume
Quobyte volumes from Kubernetes via persistent volume claims.

You can directly start to use the Free Edition of Quobyte with up to 150TB capacity and CSI support.
To access the Community support forum visit <https://support.quobyte.com> and sign up for a free Quobyte account.

## Requirements

* The Quobyte server pods must run on a dedicated node pool, i.e. the VMs/machines in this node pool must not run any other pods. This is required to guarantee the stability and performance of your storage system.
* You can run one Quobyte cluster per kubernetes cluster. If you want to run multiple Quobyte clusters, each needs a separate Kubernetes cluster. You can access Quobyte clusters from outside Kubernetes (or another k8s cluster) when you use external dns (see further down).
* For production use the minimum node pool configuration is 4 or more VMs, each at least 8 cores with 32GB RAM. For functional testing you can run with a lower number of VMs, cores or memory. However, we strongly discourage using smaller machine types. 
* If you want to access the Quobyte cluster from the outside world (i.e. other k8s clusters, VMs), you have to enable external-dns. To use this you must allow all external API calls from yor Kubernetes cluster. This should be done when you create the cluster. In addition, you need a properly configured Cloud DNS zone.

## Deploy a new Quobyte cluster

As a first step, you need to configure the Helm Chart to match your environment.
Please modify the values.yaml file in the main directory. Configuring the different sections here will 
overwrite defaults defined in each charts subdirectory. You'll find an explanation for each value inside the subchart 
yaml files:

    charts/quobyte-csi/values.yaml
    charts/quobyte-client/values.yaml
    charts/quobyte-core/values.yaml

Once you have configured the chart properly you can deploy the cluster by running

```
# Add csi helm repo. Only needed once:
$ helm repo add quobyte-csi https://raw.githubusercontent.com/quobyte/quobyte-csi/master/docs/
# Get the most recent csi version:
$ helm dependency update
# Install all sub charts as one metachart
$ helm install <DeploymentName> </path/to/thisChart/> 
```

You will get a Quobyte installation that consists of all the necessary 
parts to provide you with 
- a quobyte storage cluster ("Storage provider")
- the needed parts to consume storage dynamically ("quobyte-client/ quobyte-csi") 
within the same cluster.

# How to update/upgrade the cluster?

`helm upgrade <DeploymentName> </path/to/thisChart/> --set-string timestamp=$(date '+%s')`

This will delete stateful sets backwards and re-create them.
 
# Deleting the Cluster/Uninstalling the Cluster

Using helm uninstall will *not* delete the persistent volume claims used
*by the Quobyte core services*. If you delete these PVCs all the data
that was stored on your cluster will be lost!

If you plan to uninstall/delete your cluster to create a new one, e.g.
for testing, you will have to manually delete the PVCs before starting
a new deployment of Quobyte. Please note that you will *lose all your
data from your previous cluster* if you delete the PVCs!


## Optimization

Quobyte will send alerts if two network values are set to low. To optimize it you can tune your worker nodes with two sysctl values:

```
   net.core.rmem_max: '67108864'  
   net.core.wmem_max: '1048576'
```

To do so permanently on GKE for example you can create a config file like this:
```
linuxConfig:
 sysctl:
   net.core.rmem_max: '67108864'  
   net.core.wmem_max: '1048576'
```

and start a node deployment or cluster deployment like this:

```
gcloud container clusters create CLUSTER_NAME --system-config-from-file=SYSTEM_CONFIG_PATH
```
See also https://cloud.google.com/kubernetes-engine/docs/how-to/node-system-config

