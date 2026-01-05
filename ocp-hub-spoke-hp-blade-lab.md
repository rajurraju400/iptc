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
| Git Server (Gitea) | http://utility.ncp.bootcamp.com:3000/ | student | student |
| Argo CD (Hub) | https://openshift-gitops-server-openshift-gitops.apps.hub.ncp.bootcamp.com/applications | admin | admin |
| Hub OCP Console | https://console-openshift-console.apps.hub.ncp.bootcamp.com/multicloud/infrastructure/clusters/managed | kubeadmin | admin |

---

## 8. Blueprint Compliance Notes

- Hub installed first using Agent-based installer
- Spokes provisioned using ZTP from hub
- DNS, IPAM, and iLO mapping validated as Day-0 prerequisites
- Suitable for CNF onboarding, scale operations, and upgrades


## 9.  Arya
welcome to both
---
End of document 
