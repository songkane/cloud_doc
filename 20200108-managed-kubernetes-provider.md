# Support Managed Kubernetes Provider

## Glossary
The lexicon used in this document is described in more detail
[here](https://github.com/kubernetes-sigs/cluster-api/blob/master/docs/book/src/reference/glossary.md).
Any discrepancies should be rectified in the main Cluster API glossary.

- **MKP** - Managed Kubernetes Provider

## Summary

In current Cluster API (CAPI) , users can create Kubernetes cluster by a series of [providers](https://github.com/kubernetes-sigs/cluster-api/blob/master/docs/book/src/reference/providers.md#infrastructure),  
these providers interact with cloud infrastructure resource such as Machine, LB, SecurityGroups and InstanceGroup/AutoScaleGroup etc directly. These providers are generally referred to as infrastructure provider. The infrastructure provider usually works by create machines on the cloud or bare metal and turning them to Kubernetes master nodes(control plane) and worker nodes, finally expose them as workload/target Kubernetes cluster to end user for using.

The most of public clouds provider not only provide infrastructure service but also provide Kubernetes as a service (K8SaaS). For example, [AWS provider](https://github.com/kubernetes-sigs/cluster-api-provider-aws) depends on Amazon EC2 service to build Kubernetes cluster, actually Amazon Elastic Kubernetes Service (Amazon EKS) can build Kubernetes cluster as well.. This proposal outlines adding a new approach to CAPI by leveraging Managed Kubernetes Provider like IKS, EKS, GKE and AKS etc to create and manage Kubernetes cluster directly. The operations to provision infrastructure specific resource and turning resource to Kubernetes node are provided by MKP itself, no cloud infrastructure resource interactions and bootstrap process (kubeadm by default) get involved in CAPI anymore.

MKP is an enhancement and optional against current infrastructure provider, that means user still can use current Cluster API infrastructure provider but have another choice to manage workload/target Kubernetes cluster, since not every cloud provider will have K8SaaS.

## Motivation

The public clouds provider have invested a significant amount of time optimizing and operation reliable to manage Kubernetes cluster. The current Cluster API provider provisioning and turning solution have a lot of steps and communications with infrastructure layer which may lead error-prone. Allowing users of CAPI to leverage the optimization way exposed by MKP could prove beneficial.

**Potential benefits include:**
- Faster cluster provisioning
- Improved provisioning success rates
- Automatic cluster operations for create/delete/upgrade/scaling if supported by cloud provider 
- Improved user experience
   - no needed build machine image
   - less YAML files to maintain
   - TODO

### Goals

-  Enhance existing Cluster API providers for leveraging clouds provider K8SaaS to managing cluster.
  - Add [EKS](https://aws.amazon.com/eks/) support to [AWS provider](https://github.com/kubernetes-sigs/cluster-api-provider-aws) 
  - Add [IKS](https://www.ibm.com/cloud/container-service/) support to [IBM Cloud provider](https://github.com/kubernetes-sigs/cluster-api-provider-ibmcloud)
  - Add [AKS](https://azure.microsoft.com/en-in/services/kubernetes-service/) support to [Azure provider](https://github.com/kubernetes-sigs/cluster-api-provider-azure)
  - Add [GKE](https://cloud.google.com/kubernetes-engine/) support to [GCP provider](https://github.com/kubernetes-sigs/cluster-api-provider-gcp)
  - Add [DOKS](https://www.digitalocean.com/products/kubernetes/) support to [DigitalOcean provider](https://github.com/kubernetes-sigs/cluster-api-provider-digitalocean)
  - Add [ACK](https://www.alibabacloud.com/product/kubernetes) support to [Alibaba Cloud](https://github.com/oam-oss/cluster-api-provider-alicloud) 

### Non-goals/Future Work

- To integrate with the kubernetes cluster autoscaler.
- TODO

## Proposal

This proposal introduces MKP for the purpose of delegating the management of Kubernetes cluster to existing infrastructure [provider](https://github.com/kubernetes-sigs/cluster-api/blob/master/docs/book/src/reference/providers.md#infrastructure).

### User Stories

- As a cluster operator, I would like to build a Kubernetes cluster leverage Managed Kubernetes Provider

### Implementation Details/Notes/Constraints

Take [AWS provider](https://github.com/kubernetes-sigs/cluster-api-provider-aws/) as an example, add new type  `EKSCluster` as `infrastructureRef` to `Cluster` resource,  fill with `spec`
to `EKSCluster`, the new controller `ekscluster_controller.go` will  interact with Amazon Elastic Kubernetes Service  in `Reconcile` function for the workload/target Kubernetes cluster life cycle management.

```
kind: Cluster
apiVersion: cluster.x-k8s.io/v1alpha3
metadata:
  name: my-cluster
  namespace: default
spec:
  clusterNetwork:
    pods:
      cidrBlocks: ["192.168.0.0/16"]
    services:
      cidrBlocks: ["10.96.0.0/12"]
  infrastructureRef:
    kind: IKSCluster
    apiVersion: infrastructure.cluster.x-k8s.io/v1alpha2
    name: my-ikscluster
    namespace: default
---
apiVersion: infrastructure.cluster.x-k8s.io/v1alpha2
kind: IKSCluster
metadata:
  name: my-ikscluster
  namespace: default
spec:
  region: antarctica-1
  version: 1.14
  nodeType: t3.medium
  nodes: 1
  nodesMin: 1
  nodesMax: 2
```
