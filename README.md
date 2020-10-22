# Quobyte helm chart

Helm-based deployment for Quobyte Version 3.

This helm chart deploys the Quobyte core services as well as the clients and
the CSI plugin. This allows you to run Quobyte in Kubernetes and consume
Quobyte volumes from Kubernetes via persistent volumes claims.

## Requirements

* The Quobyte server pods must run on a dedicated node pool, i.e. the VMs/machines in this node pool must not run any other pods. This is required to guarantee the stability and performance of your storage system.
* You can run one Quobyte cluster per kubernetes cluster. If you want to run multiple Quobyte clusters, each needs a separate Kubernetes cluster. You can access Quobyte clusters from outside Kubernetes (or another k8s cluster) when you use external dns (see further down).
* Kubernetes version 1.17.12-gke.500 or 1.17.9-gke.6300, use other versions at your own risk.
* For production use the minimum node pool configuration is 4 or more VMs, each at least n2-standard-16. For functional testing you can run with a lower number of VMs. However, we strongly discourage using smaller machine types. If you want to deploy S3 proxies you need at least 6 VMs.
* If you want to access the Quobyte cluster from the outside world (i.e. other k8s clusters, GCE VMs), you have to enable external-dns. To use this yoiu must allow all external API calls from yor GKE Kubernetes cluster. This should be done when you create the cluster. In addition, you need a properly configured Cloud DNS zone.

## Deploy a new Quobyte cluster

As a first step, you need to configure the Helm Chart to match your environment.
Please modify the values.yaml file in the main directory and also in each chart
subdirectory. You'll find an explanation for each value inside the files:

    values.yaml
    charts/quobyte-csi/values.yaml
    charts/quobyte-client/values.yaml
    charts/quobyte-core/values.yaml

Please make sure to configure the username and password secrets in the quobyte-csi/values.yaml file.
The password must be base 64 encoded. Once your Quobyte cluster is up and running you must create this
user (via the UI or qmgmt) with the exact password you specified in the yaml file. This user is required
to allow the Quobyte CSI plugin to talk to the Quobyte API service to provision volumes and quotas. You
can verify the username/password with qmgmt if in doubt.

Once you have configured the chart properly you can deploy the cluster by running

    $ helm install <newDeploymentName> </path/to/helmChart/> 

You will get a Quobyte installation that consists of all the necessary 
parts to provide you with 
- a quobyte storage cluster ("Storage provider")
- the needed tools to consume storage dynamically ("Storage consumer") 
within the same cluster.

To successfully use dynamically provisioned volumes you must create the user with the
password you specified in quobyte-csi/values.yaml.
Otherwise provisioning volumes will fail / pvcs will be pending.
That means, if you roll out a cluster for the first time, you should 
navigate to "Configuration" --> "User Database" and add the user that
was defined in values.yaml.

# How to update/upgrade the cluster?

`helm upgrade qs . --set-string timestamp=$(date '+%s')`

This will delete stateful sets backwards and re-create them.
 
# Deleting the Cluster/Uninstalling the Cluster

Using helm uninstall will *not* delete the persistent volume claims used
*by the Quobyte core services*. If you delete these PVCs all the data
that was stored on your cluster will be lost!

If you plan to uninstall/delete your cluster to create a new one, e.g.
for testing, you will have to manually delete the PVCs before starting
a new deployment of Quobyte. Please note that you will *lose all your
data from your previous cluster* if you delete the PVCs!

