the qoubyte installation on k8s is split into two parts, depending on 
your use case you can install either one of them or both togehter:

quobyte
	provider
		registry
		metadata
		quobyte-api
		dataservice	
		s3-proxy
		nfs-proxy
		web-console
		...

		
	consumer
		csi
		clients

That means, that you can use this helm chart to deploy 
the provider part of quobyte (a.k.a the core product, providing 
scalable storage). Or you can decie to only install the consumer part 
(quobyte client + csi framework) to use an existing quobyte storage 
cluster and use it in your current k8s cluster as a storage provider. 
In this case you need to fill in the registry url as a parameter in 
values.yaml.


