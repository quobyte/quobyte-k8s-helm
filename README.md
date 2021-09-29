# Quobyte Helm chart

Helm-based Quobyte deployment.

This helm chart deploys the Quobyte core services as well as the clients and
the CSI plugin. This allows you to run Quobyte in Kubernetes and consume
Quobyte volumes from Kubernetes via persistent volume claims.

You can directly start to use the Free Edition of Quobyte with up to 150TB capacity and CSI support.
To access the Community support forum visit <https://support.quobyte.com> and sign up for a free Quobyte account.

## Requirements

* For production use the minimum node pool configuration is 4 or more worker nodes, each at least 8 cores with 32GB RAM. For functional testing you can run with a lower number of nodes, cores or memory. A smaller machine count will affect availability while less ressources will affect performance.  
* To guarantee stability and performance of a Quoybte storage system it is recommendet to run all storage pods on a dedicated node pool.
* Quobyte will work best with optimized network settings for your worker nodes. See "Quobyte cluster Optimization" below. 
* If you want to access the Quobyte cluster from the outside world (i.e. other k8s clusters, VMs), you need to make sure that all Quobyte services are reachable with all ports. For firewall planning etc. see [Quobyte Services](https://docs.quobyte.com/docs/16/latest/reference_services.html) for details. 

## Deploy a Quobyte cluster

To install a Quoybte cluster you need a working values.yaml and helm installed.

```
$ cp values_core-cluster.yaml_example values.yaml
$ helm install <deploymentName> .
```

After all pods are up and running you access your webconsole.
You can continue to license the cluster, create a super user 
and configure the cluster according to your needs from there.
For all Quobyte options see also the official documentation:

https://docs.quobyte.com/

## Consuming storage from an existing Quobyte cluster

To consume storage from a Quobyte cluster you can use the same helm chart with a different configuration:

```
$ cp values_csi+client.yaml_example values.yaml 
# Adjust values as needed by your existing Quobyte cluster
```

Once you have configured the chart properly you can deploy Quobyte clients and the Quobyte CSI plugin: 

```
# Add csi helm repo. Only needed once:
$ helm repo add quobyte-csi https://raw.githubusercontent.com/quobyte/quobyte-csi/master/docs/
# Get the most recent csi version:
$ helm dependency update
# Install all sub charts as one metachart
$ helm install <clientDeployment> . 
```

# How to update/upgrade a Quobyte cluster?

```
$ helm upgrade <DeploymentName> . --set-string timestamp=$(date '+%s')`
``` 
This will delete stateful sets backwards and re-create them.
 
# Deleting the Cluster/Uninstalling the Cluster

Using helm uninstall will *not* delete the persistent volume claims used
*by the Quobyte core services*. If you delete these PVCs all the data
that was stored on your cluster will be lost!

If you plan to uninstall/delete your cluster to create a new one, e.g.
for testing, you will have to delete the PVCs before starting
a new deployment of Quobyte. Please note that you will *lose all your
data from your cluster* if you delete the PVCs!


## Quobyte cluster Optimization

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

