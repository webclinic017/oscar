# Multitenancy support in OSCAR

In the context of OSCAR, multi-tenancy support refers to the platform's ability to enable multiple users or organizations (tenants) to deploy and run their applications or functions on the same underlying infrastructure. Support for multitenancy in OSCAR has been available since version v3.0.0. To use this functionality, there are some requisites that the cluster and the users have to fulfill:

- **The cluster needs to have enabled OIDC access.**

	This is because the implementation of this functionality uses directly the EGI UIDs to distinguish between users, therefore removing the need to manage a database.  

- **Users who want to create new services need to know the UID of the users who will have access to the service.**
	
	Each service has a list of "allowed users," so a service can be accessed not only by one but by multiple users chosen by the service creator. This way, users have the ability to decide who can operate over its services. It is important to note that at this moment, a user with access to a service has full access; this means he can edit/delete the service besides making executions. 

    This allowed users list is defined on the FDL on the service creation (more info in link FDL doc). The following is an example of an FDL that creates a service that gives access to two EGI users. If the allowed_users field is empty, the service is treated as "public," so every user within the cluster will have access to it.

    At the moment of the creation, the UID of the user creating the service doesn't need to be present on the list; however, when a service is updated, if the user will still have access, its UID has to be on the list.

``` yaml
functions:
  oscar:
  - oscar-cluster:
      name: grayify_multitenant
      memory: 1Gi
      cpu: '0.6'
      image: ghcr.io/grycap/imagemagick
      script: script.sh
      vo: "vo.example.eu" # Needed to create services on OIDC enabled clusters
      allowed_users: 
      - "62bb11b40398f73778b66f344d282242debb8ee3ebb106717a123ca213162926@egi.eu"
      - "5e14d33ac4abc96272cc163da6a200c2e18591bfb3b0f32a4c9c867f5e938463@egi.eu"
      input:
      - storage_provider: minio.default
        path: grayify_multitenant/input
      output:
      - storage_provider: minio.default
        path: grayify_multitenant/output
```


> **_NOTE:_** A user can obtain its EGI User Id by login into https://aai.egi.eu/ (for the production instance of EGI Check-In) or https://aai-demo.egi.eu (for the demo instance of EGI Check-In). 


Since OSCAR uses MinIO as the main storage provider, so that the users only have access to their designated bucket's service, MinIO users are created on the fly for each EGI UID. Consequently, each user accessing the cluster will have a MinIO user with its UID as AccessKey and an autogenerated SecretKey.