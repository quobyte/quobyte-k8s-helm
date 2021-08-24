# Storage Use Case Examples

Before you can consume CSI provisioned storage you 
will need 

* A secret to access secured storage
* A storage class

Kubernetes users can then use 

* A persistent volume claim (pvc) to provision storage.
This pvc will refer to the storageClass provided earlier. 

You'll find configuration examples here.

With the Quoybte storage driver you have the option to use 
dynamic mapping between Kubernetes namespaces and Quobyte tenants.

If you follow this strategy your Kubernetes accounts / users can provision
different storage types depending on their context. A user in a namespace 
"development" will be able to provision storage in a Quobyte-tenant "development". 
This has three consequences: 
* Users from one namespace cannot access storage from other namespaces.
* Defined tenant-wide storage rules like Quota or available storage flavors (SSD, HDD) are applied automatically. 
* Users can administer their identity autonomously. They can change their Quobyte credentials and adjust the Kubernetes side without the help from a cluster administrator.  

Alternatively you can decide to deploy a set of Quobyte credentials that are implicitly available to all users of your cluster. This will result in storage that is provisioned within one Quobyte tenant. 
This option has the advantage that no user needs to maintain or deploy a secret within their namespace. 
