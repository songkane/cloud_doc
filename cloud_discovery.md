# Cloud Discovery

## Requirement 

Currently, when create remote k8s cluster by cluster api provider (iks/aks/ocp/gke) from MCM-UI, MCM user has to prepare cluster specification, such as region,zone,network,flavor,etc information from public cloud at first, then fill with the `providerSpec` manually,  this is poor user experience. Ideally, the cluster api provider should have the capacity to dynamic discover the cluster specification from public colud according to user permission and configuration, MCM user just select it from MCM-UI dropdown list is enough, then pass the selection to back-end. 

## Solution

**Solution 1: introduce new controller**

 1. Define new crd for cluster api provider by new controller, this crd used to define supported cluster specification 
 2. Create cr per each `cloud-connection` secret according to user permission and configuration
 3. MCM-UI retrieve cr to get supported cluster specification
 4. A CronJob to sync up cluster specification per each `cloud-connection` secret with a interval 

**Solution 2: introduce new api-extension**

 1. Define new api-extension for cluster api provider by define `APIService` object
 2. The new api-extension retrieve the cluster specification according to user permission and configuration from public cloud
 3. MCM-UI call this api-extension to discover the cluster specification
 4. We can follow k8s [metrics-server](https://github.com/kubernetes-incubator/metrics-server) to implemet api-extension

# I prefer Solution 2: 
 * No time windows issue
 * No need to persist huge and reduplicate cluster specification data, such as flavor/machine to etcd
 * Easy to MCM/UI to integration
 * Easy to replace `<cluster-name>-job` in future
