# Quobyte Helm chart

The Qoubyte installation on k8s is split into two parts, depending on 
your use case you can install either one of them or both togehter:

quobyte
	provider (a.k.a storage cluster)
		registry
		metadata
		quobyte-api
		dataservice	
		s3-proxy
		nfs-proxy
		web-console
		...

		
	consumer (a.k.a dynamic storage provisioning)
		csi
		clients


