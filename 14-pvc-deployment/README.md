# GKE Workshop LAB-14 (B)

[![Context](https://img.shields.io/badge/GKE%20Fundamentals-1-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![GKE/K8s Version](https://img.shields.io/badge/k8s%20version-1.18.20-blue.svg)](#)
[![GCloud SDK Version](https://img.shields.io/badge/gcloud%20version-359.0.0-blue.svg)](#)
[![elasticSearch Version](https://img.shields.io/badge/elasticsearch%20version-6.2.4-green.svg)](#)
[![Build Status](https://img.shields.io/badge/status-unstable-E47911.svg)](#)

## Introduction

In the following lab we will set up our local development environment, provision the workshop cluster and roll out our next application, elasticsearch ([source](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-quickstart.html)) single-node stack. This lab will give us an additional insight into the standard Kubernetes resources, clarify an important approach regarding the PV/PVCs and provide a control base for our next-step labs.

## Deployment

The PV (persistent volume) is a "physical disk".

Persistent volume claims (PVC) tell a pod to bind to a specific PV. They have to be bound this way as the PV itself is not a namespace object, but is global. The PVC is a namedspace entity, so we need it to link to a specific pod.

The have a lifecycle independent of the pod that uses it. They are implemented as plugins and k8s support many types. PVs hvae a specific storage capacity, which can be set or requested.

PVs has its own set of access modes (ReadWriteOnce, ReadWriteMany, ReadOnlyMany, ReadWriteOnePod). Most common is ReadWriteOnce.

PVCs are essentially a request for storage. We can request a specific size and levels of resirces (CPU & Memory). If the PVC cannot get the PV, we get errors.

PVC to PV binding is a 1-1 mapping. It uses a ClaimRef, which is bidirectional between the PV and PVC.

When a user is done with the volume, it can delete the PVC object from the API. The reclaim policy tells the cluster what to do with the PV following the release of a claim. Currently PVs can be retained, recycled or deleted. In general, used `Retained` which is the safest.

PVs are created by a StorageClass and the reclaimPolicy is enacted by this.

### Run Deployment

```bash
kubectl apply -f 00-namespace.yaml
kubectl apply -f 01-elasticsearch-simple.yaml
```

On inspecting the pods, we see something in volumes:

```bash
Volumes:
  data:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  elasticsearch-data
    ReadOnly:   false
  kube-api-access-mw6g2:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
```

### Inspect the storage class

```bash
k get storageclasses.storage.k8s.io premium-rwo -o yaml | yq
```

```yaml
allowVolumeExpansion: true
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
    components.gke.io/component-name: pdcsi
    components.gke.io/component-version: 0.10.7
    components.gke.io/layer: addon
  creationTimestamp: "2022-03-28T10:24:12Z"
  labels:
    addonmanager.kubernetes.io/mode: EnsureExists
    k8s-app: gcp-compute-persistent-disk-csi-driver
  name: premium-rwo
  resourceVersion: "370"
  uid: e4ee9ed8-d1d0-4e61-a9a9-c57ae67f489a
parameters:
  type: pd-ssd
provisioner: pd.csi.storage.gke.io
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

Note the `reclaimPolicy` is Delete.

In GCP Compute Engines -> Disk, we can see the disks related to the node pools. We can also see a pvc listed!

### Other commands to run

```bash
k describe persistentVolume [persistent volume name]
k describe persistentvolumeclaims elasticsearch-data
```

```bash
PVC Events:
  Type    Reason                 Age   From                                                                                              Message
  ----    ------                 ----  ----                                                                                              -------
  Normal  WaitForFirstConsumer   11m   persistentvolume-controller                                                                       waiting for first consumer to be created before binding
  Normal  ExternalProvisioning   11m   persistentvolume-controller                                                                       waiting for a volume to be created, either by external provisioner "pd.csi.storage.gke.io" or manually created by system administrator
  Normal  Provisioning           11m   pd.csi.storage.gke.io_gke-b57fe2736b4e448cbd52-14b2-78c1-vm_85f0289a-d47d-4f48-ad35-4779f30bbc1a  External provisioner is provisioning volume for claim "doit-lab-14/elasticsearch-data"
  Normal  ProvisioningSucceeded  11m   pd.csi.storage.gke.io_gke-b57fe2736b4e448cbd52-14b2-78c1-vm_85f0289a-d47d-4f48-ad35-4779f30bbc1a  Successfully provisioned volume pvc-24c00ba0-490e-4d85-a7a5-916ca1ac07ee
```

## Application Clean-Up

```bash
kubectl delete -f 00-namespace.yaml
```

## Links

- https://gist.github.com/pyk/3fc87db27eed864e354974bc25aabf88
- https://kubernetes.io/docs/reference/kubectl/cheatsheet/
- https://phoenixnap.com/kb/kubectl-commands-cheat-sheet
