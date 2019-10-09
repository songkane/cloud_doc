# Cloud Discovery

## Requirement 

Currently, when user create remote k8s cluster by cluster api provider (iks/aks/ocp/gke) from MCM-UI, they have to get the cluster specification, such as region,zone,network,flavor,etc information from public cloud at first, then fill with the `providerSpec` manually,  this is poor user experience. Ideally, the cluster api provider should have the capacity to dynamic discover the cluster specification from public colud according to user permission and configuration, user just select or mark it from MCM-UI is enough.

## Solution

**Solution 1: introduce new controller**

 * Define new crd for cluster api provider by new controller, this crd used to define the supported cluster specification 
 * Create cr per each `cloud-connection` according to user permission and configuration
 * MCM-UI retrieve cr to get cluster specification 
 * A CronJob to sync up cluster specification per each `cloud-connection`

**Solution 2: introduce new api-extension**

 * Define new api-extension for each cluster api provider
 * The new api-extension retrieve the cluster specification according to user permission and configuration
 * MCM-UI call the new api-extension with the user permission and configuration
 
 We can follow k8s metrics-server to implemet api-extension

# I prefer Solution 2
