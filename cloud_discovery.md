# Cloud Discovery

## Requirement 

Currently, when create remote k8s cluster by cluster api provider (iks/aks/ocp/gke) from MCM-UI, MCM user has to prepare cloud resource specification, such as region,zone,network,flavor,etc information from public cloud at first, then fill with the `providerSpec` manually, this is poor user experience. Ideally, the cluster api provider should have the capacity to dynamic discover the cloud resource specification from public colud according to user permission and configuration, MCM user just select it from MCM-UI dropdown list is enough, then pass the selection to back-end. 

Most of the public cloud resource is not role based/account resource, that means different user can use same cloud resource without permission, validation check or filter, but same resource not:

1. IKS: `publicVlan` and `privateVlan` is role based resource
2. AKS: `location` is account/subscription based resource
3. OCP: `region` is role based resource

## Solution

**Option 1**:

Build a new controller to query clould resources and return to UI. The controller `reconcile` need to call public cloud API/CLI to get all resources. MCM-UI can get all of the resources from cr.

 1. Define new crd for cluster api provider by new controller, this crd used to define supported cloud resource 
 2. Create cr per each `cloud-connection` secret according to user permission and configuration
 3. MCM-UI retrieve cr to get cloud resource
 4. A CronJob to sync up cloud resource per each `cloud-connection` secret with a interval 

**Option 2**:

Leverage https://github.ibm.com/pattern-sdk/cloud-discovery-service which is a python-based API service for cloud metadata, but this is not kubernetes native and have gap to support all public cloud

**Option 3**:

Build a new API extension to extend native k8s api, UI can get all of the resources from this extension api

 1. Define new api-extension for cluster api provider by define `APIService` object
 2. The new api-extension retrieve the cloud resource according to user permission and configuration from public cloud
 3. MCM-UI call this api-extension to fill with the dropdown list
 4. We can follow k8s [metrics-server](https://github.com/kubernetes-incubator/metrics-server) to implemet api-extension


**Option 4**:

MCM-UI call public cloud Rest API to get cloud resource directly.


# I prefer to Solution 2: 
 * No time windows issue
 * No need to persist huge and reduplicate cluster specification data, such as flavor/machine to etcd
 * Easy to MCM/UI to integration
 * Easy to replace `<cluster-name>-job` in future
