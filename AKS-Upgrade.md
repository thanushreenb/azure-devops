# AKS Cluster Upgrade - Steps and Implementation

## Overview  
This document outlines the steps followed to upgrade an Azure Kubernetes Service (AKS) cluster from **version 1.28.9** to **1.29.10**.

## Prerequisites  

1. **Notification to Stakeholders**  
   - Sent a notification to all users and stakeholders regarding the scheduled upgrade.

2. **Current and Target Versions**  
   - **Current AKS version:** `1.28.9`  
   - **Target AKS version:** `1.29.10`

3. **Ingress Controller Compatibility**  
   - **Existing nginx ingress controller version:** `4.8.3`  
   - **Compatible version for Kubernetes 1.29.10:** `4.9.1`  
   - Reference: [Ingress-nginx](https://github.com/kubernetes/ingress-nginx)

4. **Review Kubernetes Release Notes**  
   - Checked Kubernetes release notes to identify API changes and deprecated versions before upgrading.

---

## Implementation Steps  

### 1. Check Available Kubernetes Versions for Upgrade  
```sh
az aks get-upgrades --resource-group <ResourceGroupName> --name <AKSClusterName> --query "controlPlaneProfile.upgrades[*].kubernetesVersion"
az aks get-upgrades --resource-group demoaks-rg --name demo-aks --output table
```
![aks-available-version](https://github.com/ravdy/azure-devops/blob/main/images/aks-available-version.png)
### 2. Upgrade Nginx Ingress Controller  
```sh
helm upgrade "nginx-ingress-internal" ingress-nginx/ingress-nginx \
  --version 4.9.1 --namespace ingress-internal \
  -f internal-ingress-nginx.yaml --kube-context demo-aks

helm upgrade "nginx-ingress-external" ingress-nginx/ingress-nginx \
  --version 4.9.1 --namespace ingress-external \
  -f external-ingress-nginx.yaml --kube-context demo-aks
```

### 3. Upgrade AKS Cluster to Kubernetes 1.29.10  
```sh
az aks upgrade --resource-group demoaks-rg --name demo-aks --kubernetes-version 1.29.10
```
- The upgrade process replaces the nodes **one by one** to ensure minimal downtime.
  
![aks-upgrade-process-1](https://github.com/ravdy/azure-devops/blob/main/images/aks-upgrade-process-1.png)

![aks-upgrade-process-2](https://github.com/ravdy/azure-devops/blob/main/images/aks-upgrade-process-2.png)

![aks-upgrade-process-3](https://github.com/ravdy/azure-devops/blob/main/images/aks-upgrade-process-3.png)

![aks-upgrade-process-4](https://github.com/ravdy/azure-devops/blob/main/images/aks-upgrade-process-4.png)
---

## Post-Upgrade Verification  
After the upgrade, verify the cluster and ingress controller versions:

```sh
kubectl get nodes -o wide
kubectl get pods -n ingress-internal
kubectl get pods -n ingress-external
```

Ensure all services are running as expected. If issues arise, check logs:
```sh
kubectl logs <pod-name> -n <namespace>
