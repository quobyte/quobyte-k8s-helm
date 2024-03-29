# Using Access Keys to mount Quobyte Volumes

Quobyte 3.0 brings the option to use access keys to authenticate against 
the storage system not only for Object Storage, but also for native mounts.
In containerized environments this can be used to ensure reliable and secure user 
identities on file system level.
The accessing identity is not resolved by the operating system 
environment of a container or worker node. Instead the file system
takes care of that, allowing centralized user administration.

Existing and new file ACLs can now be evaluated and used across boundaries of access methods.

## Separating storage provisioning and storage consumption

To provision storage and to consume storage are two different things:
The first one is capable to create or delete a volume, the latter one 
allows to use such a generated volume. 
Kubernetes supports this separation when creating a storage class:

## Storage class example

```
allowVolumeExpansion: true
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: sc-quobyte-accesskey
parameters:
# The following requires existing namespace "quobyte" + secret "tenantadmin-secret" + secret "tenantuser-accesskey"
  csi.storage.k8s.io/controller-expand-secret-name: tenantadmin-credentials
  csi.storage.k8s.io/controller-expand-secret-namespace: quobyte
  csi.storage.k8s.io/provisioner-secret-name: tenantadmin-credentials
  csi.storage.k8s.io/provisioner-secret-namespace: quobyte
  csi.storage.k8s.io/node-publish-secret-name: tenantuser-accesskey
  csi.storage.k8s.io/node-publish-secret-namespace: default 

# Attributes to create a Quoybte volume with
  createQuota: "true"
  accessMode: "700"
  user: kubernaut 
  group: kubernautees
  quobyteConfig: BASE
  quobyteTenant: "kubernautees"  
  labels: "usagetype:exploringSpace"
provisioner: csi.quobyte.com
```

This example shows the distinction between provisioning/ expanding a volume and using it ("publishing" it).
This separation is done using different sets of credentials: ``` tenantadmin-credentials ``` and ``` tenantuser-accesskey ```.
The first Kubernetes secret holds a Quobyte username and password. This is used to communicate to the 
Quobyte API and maintain Quobyte volumes. 
The second secret holds an access key. This key is used to authenticate agains Quobyte and mount Quobyte volumes.
Since an access key in Quobyte always resolves to a tenant, a username and a group we have all informations we 
need to select the right volume to mount, create or modify files and do proper ACL enforcement.

This way storage consumption and storage administration are strictly separated. That is also reflected in using different namespaces in Kubernetes: 
The admin user identity is using a dedicated namespace while the access keys can be created/ used in an independent namespace ("default" in this case).

To complete the code examples here are the two different types of secrets:

## Tenant Admin credentials

```
apiVersion: v1
kind: Secret
metadata:
  name: tenantadmin-credentials
  namespace: quobyte
type: "kubernetes.io/quobyte"
data:
    user: a3ViZXJuYXV0 
    password: Y2hhbmdlTWU= 
```

## Unprivileged Access Keys 

```
apiVersion: v1
kind: Secret
metadata:
  name: tenantuser-accesskey
  namespace: default
type: "kubernetes.io/quobyte"
data:
    accessKeyId: RHZjMTF6NmozcTdGd3pNdUZ6OGU= 
    accessKeySecret: NjMrUEhxZ0IrTUVFNWVhQmV1ejFIWHlDRnBnYmdYNFIzV2Z0dW5NTg== 
```

Since both objects are of type "kubernetes.io/quobyte" values need to be base64 
encoded.

To complete code examples here is the one to create a dedicated namespace:

## Quobyte namespace

```
apiVersion: v1
kind: Namespace
metadata:
  labels:
    kubernetes.io/metadata.name: quobyte
  name: quobyte
```

### Enable Access Key based storage class

1. Make sure your CSI configuration is configured to enable access keys
and Quobyte version 3 is chosen.

```
quobyte-csi:
  quobyte: 
    enableAccessKeys: true
    version: 3
```
2. Make sure that your Client configuration is configured to enable access keys
```
quobyte-client:
  quobyte: 
    enableAccessKeys: true
```

2. Deploy quobyte namespace
```
kubectl apply -f 01_quobyte_namespace.yaml
```
4. Deploy tenant admin secret
```
kubectl apply -f 02_tenantadmin-credentials.yaml
```
5. Deploy unprivileged user access keys
```
kubectl apply -f 03_tenantuser-accesskeys.yaml
```
6. Deploy Quobyte storage class
```
kubectl apply -f 04_sc-quobyte-accesskey.yaml
```

### Using Access Key based storage class

Kubernetes users can simply create a pvc using the defined storage class:

#### Example PVC

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: quobyte-default-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: sc-quobyte-accesskey 
```

#### Apply PVC declaration

```
kubectl apply -f 05_example-pvc.yaml
```

#### Example Pod declaration 

```
apiVersion: v1
kind: Pod
metadata:
  name: testpod
spec:
  containers:
  - name: server
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      name: quobytepvc
  volumes:
  - name: quobytepvc
    persistentVolumeClaim:
      claimName: quobyte-default-pvc
```

#### Starting that Pod
```
kubectl apply -f 06_testpod.yaml
```

