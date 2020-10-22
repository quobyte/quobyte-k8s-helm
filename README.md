# Quobyte helm chart

Quobyte deployment using helm.

With a 

```
 $ helm install <newDeploymentName> </path/to/helmChart/> 
```

you will get a Quobyte installation that consists of all the necessary 
parts to provide you with 
- a quobyte storage cluster ("Storage provider")
- the needed tools to consume storage dynamically ("Storage consumer") 
within the same cluster.

To successfully use dynamically provisioned volumes you should keep 
the credentials that are defined in values.yaml and the user database
(either internal or LDAP) in sync.
Otherwise provisioning volumes will fail / pvcs will be pending.
That means, if you roll out a cluster for the first time, you should 
navigate to "Configuration" --> "User Database" and add the user that
was defined in values.yaml.

You can use one storage-class out of the box: "quobyte-rf3".

This one uses 3x replication. You can add more storage classes to suit
your needs; just take "quobyte-rf3" as an example.


