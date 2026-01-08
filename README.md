# OpenShift Lab – Hub and Spoke Deployment (HP Blade | Nokia NCP Style)

## 1. Overview
This document describes the OpenShift Container Platform (OCP) lab rebuild using HP Blade servers, following Nokia NCP Hub-and-Spoke blueprint architecture.

- Hub Cluster: Agent-based installation
- Spoke Clusters: ZTP-based installation
- Platform focus: OpenShift, Linux, ZTP process, Agent-based install
- Use case: Telco-style Day-0 to Day-2 lifecycle testing

---

## 2. Architecture Summary (Nokia / NCP Blueprint)

- Hub Cluster (Management Cluster)
  - ACM, GitOps (ArgoCD), Policies, Observability
  - ZTP controller for spoke clusters

- Spoke Clusters (Workload Clusters)
  - Fully declarative (GitOps)
  - No manual node changes post-install

---

## 3. Logical Topology (ASCII)

```
                         +--------------------------------+
                         |        Hub Cluster              |
                         |  hub.ncp.bootcamp.com           |
                         |                                |
                         |  ACM / GitOps / Policies        |
                         +---------------+----------------+
                                         |
                               ZTP / GitOps
                                         |
        -----------------------------------------------------------------
        |                                                               |
+---------------------------+                         +---------------------------+
|     Spoke Cluster: CWL    |                         |    Spoke Cluster: CWL2    |
|  cwl.ncp.bootcamp.com     |                         |  cwl2.ncp.bootcamp.com    |
|  API / Apps VIPs          |                         |  API / Apps VIPs          |
|  Masters / Workers / GW   |                         |  Masters / Workers / GW   |
+---------------------------+                         +---------------------------+
```

---

## 4. Utility and Hub Nodes

| ILO IP | Hostname | OAM IP | FQDN |
|------|---------|-------|------|
| 135.104.149.28 | Utility | 135.104.149.130 | utility.ncp.bootcamp.com |
| | | 135.104.149.131 | api.hub.ncp.bootcamp.com |
| | | 135.104.149.132 | *.apps.hub.ncp.bootcamp.com |
| 135.104.149.29 | master-0 | 135.104.149.133 | master-0.hub.ncp.bootcamp.com |
| 135.104.149.30 | master-1 | 135.104.149.134 | master-1.hub.ncp.bootcamp.com |
| 135.104.149.31 | master-2 | 135.104.149.135 | master-2.hub.ncp.bootcamp.com |

---

## 5. Spoke Cluster: CWL

| ILO IP | Hostname | OAM IP | FQDN |
|------|---------|-------|------|
| 135.104.149.32 | master-0 | 135.104.149.138 | master-0.cwl.ncp.bootcamp.com |
| 135.104.149.33 | master-1 | 135.104.149.139 | master-1.cwl.ncp.bootcamp.com |
| 135.104.149.34 | master-2 | 135.104.149.140 | master-2.cwl.ncp.bootcamp.com |
| 135.104.149.35 | worker-0 | 135.104.149.141 | worker-0.cwl.ncp.bootcamp.com |
| 135.104.149.36 | worker-1 | 135.104.149.142 | worker-1.cwl.ncp.bootcamp.com |
| 135.104.149.37 | gw-1 | 135.104.149.143 | gw-1.cwl.ncp.bootcamp.com |
| 135.104.149.38 | gw-2 | 135.104.149.144 | gw-2.cwl.ncp.bootcamp.com |

---

## 6. Spoke Cluster: CWL2

| ILO IP | Hostname | OAM IP | FQDN |
|------|---------|-------|------|
| 135.104.149.39 | master-0 | 135.104.149.147 | master-0.cwl2.ncp.bootcamp.com |
| 135.104.149.40 | master-1 | 135.104.149.148 | master-1.cwl2.ncp.bootcamp.com |
| 135.104.149.41 | master-2 | 135.104.149.149 | master-2.cwl2.ncp.bootcamp.com |
| 135.104.149.42 | worker-0 | 135.104.149.151 | worker-0.cwl2.ncp.bootcamp.com |
| 135.104.149.43 | gw-1 | 135.104.149.152 | gw-1.cwl2.ncp.bootcamp.com |

---

## 7. Access Information (Lab Credentials)

> **Note:** Credentials are for **lab / training use only**. Do not reuse in production.

| Component | URL | Username | Password |
|---------|-----|----------|----------|
| infra quay | https://utility.ncp.bootcamp.com:8443/repository/ | quayadmin | quay1234|
| Git Server (Gitea) | http://utility.ncp.bootcamp.com:3000/ | student | student |
| Hub OCP Console | https://console-openshift-console.apps.hub.ncp.bootcamp.com/multicloud/infrastructure/clusters/managed | kubeadmin | DjF7h-5dCmb-Z7yky-vu98a |
| Argo CD (Hub) | https://openshift-gitops-server-openshift-gitops.apps.hub.ncp.bootcamp.com/applications | admin | admin |
| Hub quay | https://quay-registry.apps.hub.ncp.bootcamp.com | quayadmin | quay1234 |

---

## 8. Blueprint configuration deviation Notes

The following deviations from the standard Nokia NCP blueprint are applicable to this lab environment and are documented for transparency and reference:

- All nodes are HP Blade servers equipped with 2 × 1 TB local disks per node.
- Ceph is deployed using one 1 TB disk per master node. As a result, the effective usable Ceph capacity in the lab is limited to approximately 1 TB.
- The HP Blade servers support a maximum of 48 vCPUs. Due to this constraint, the NCD Git component cannot be deployed on the Hub cluster as per the standard blueprint. Instead, a lightweight (minion) GitHub instance is deployed on the infra-manager (utility) node.
- Advanced networking configurations are not implemented in this lab:
  - SR-IOV–based networking (NIC2) testing is not possible.
  - MetalLB–based load-balancer testing is also not supported.

These deviations are specific to the lab setup and do not represent limitations of the Nokia NCP or OpenShift platform. They should be considered acceptable constraints for functional validation and learning purposes.

## 9. Installation Completed 

- Hub Cluster

```
[root@utility ~ hub_rc]# oc get managedclusters 
NAME            HUB ACCEPTED   MANAGED CLUSTER URLS                    JOINED   AVAILABLE   AGE
cwl             true           https://api.cwl.ncp.bootcamp.com:6443   True     True        138m
local-cluster   true           https://api.hub.ncp.bootcamp.com:6443   True     True        6d3h
[root@utility ~ hub_rc]# oc get nodes
NAME                            STATUS   ROLES                         AGE   VERSION
master-0.hub.ncp.bootcamp.com   Ready    control-plane,master,worker   8d    v1.29.10+67d3387
master-1.hub.ncp.bootcamp.com   Ready    control-plane,master,worker   8d    v1.29.10+67d3387
master-2.hub.ncp.bootcamp.com   Ready    control-plane,master,worker   8d    v1.29.10+67d3387
[root@utility ~ hub_rc]# oc get cd -A
NAMESPACE   NAME   INFRAID                                PLATFORM          REGION   VERSION   CLUSTERTYPE   PROVISIONSTATUS   POWERSTATE   AGE
cwl         cwl    43b39069-836e-44a3-bf0e-ef3a8c361449   agent-baremetal            4.16.24                 Provisioned       Running      138m
[root@utility ~ hub_rc]# oc get cgu -A
NAMESPACE     NAME            AGE    STATE       DETAILS
ztp-install   cwl             80m    Completed   All clusters already compliant with the specified managed policies
ztp-install   local-cluster   6d3h   Completed   All clusters already compliant with the specified managed policies
[root@utility ~ hub_rc]# 

```
- CWL Cluster 

```
[root@utility ~ cwl_rc]# oc get nodes 
NAME                            STATUS   ROLES                                 AGE    VERSION
master-0.cwl.ncp.bootcamp.com   Ready    control-plane,master,monitor,worker   116m   v1.29.10+67d3387
master-1.cwl.ncp.bootcamp.com   Ready    control-plane,master,monitor,worker   116m   v1.29.10+67d3387
master-2.cwl.ncp.bootcamp.com   Ready    control-plane,master,monitor,worker   92m    v1.29.10+67d3387
[root@utility ~ cwl_rc]# oc get co 
NAME                                       VERSION   AVAILABLE   PROGRESSING   DEGRADED   SINCE   MESSAGE
authentication                             4.16.24   True        False         False      86m     
baremetal                                  4.16.24   True        False         False      109m    
cloud-controller-manager                   4.16.24   True        False         False      112m    
cloud-credential                           4.16.24   True        False         False      126m    
cluster-autoscaler                         4.16.24   True        False         False      109m    
config-operator                            4.16.24   True        False         False      111m    
console                                    4.16.24   True        False         False      91m     
control-plane-machine-set                  4.16.24   True        False         False      109m    
csi-snapshot-controller                    4.16.24   True        False         False      102m    
dns                                        4.16.24   True        False         False      102m    
etcd                                       4.16.24   True        False         False      108m    
image-registry                             4.16.24   True        False         False      94m     
ingress                                    4.16.24   True        False         False      93m     
insights                                   4.16.24   True        False         False      104m    
kube-apiserver                             4.16.24   True        False         False      107m    
kube-controller-manager                    4.16.24   True        False         False      107m    
kube-scheduler                             4.16.24   True        False         False      105m    
kube-storage-version-migrator              4.16.24   True        False         False      111m    
machine-api                                4.16.24   True        False         False      95m     
machine-approver                           4.16.24   True        False         False      110m    
machine-config                             4.16.24   True        False         False      109m    
marketplace                                4.16.24   True        False         False      109m    
monitoring                                 4.16.24   True        False         False      91m     
network                                    4.16.24   True        False         False      110m    
node-tuning                                4.16.24   True        False         False      92m     
openshift-apiserver                        4.16.24   True        False         False      94m     
openshift-controller-manager               4.16.24   True        False         False      106m    
openshift-samples                          4.16.24   True        False         False      83m     
operator-lifecycle-manager                 4.16.24   True        False         False      108m    
operator-lifecycle-manager-catalog         4.16.24   True        False         False      108m    
operator-lifecycle-manager-packageserver   4.16.24   True        False         False      101m    
service-ca                                 4.16.24   True        False         False      111m    
storage                                    4.16.24   True        False         False      111m    
[root@utility ~ cwl_rc]# 

```

---
End of document 
