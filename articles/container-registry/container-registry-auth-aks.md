---
title: "使用來自 Azure Container Service 的 Azure Container Registry 進行驗證"
description: "了解如何使用 Azure Active Directory 服務主體，以從 Azure Container Service 提供私人容器登錄中的映像存取權。"
services: container-service
author: neilpeterson
manager: timlt
ms.service: container-service
ms.topic: article
ms.date: 02/24/2018
ms.author: nepeters
ms.openlocfilehash: a115df87feea0c9f7987e0c65f6f880325d88ca2
ms.sourcegitcommit: 782d5955e1bec50a17d9366a8e2bf583559dca9e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/02/2018
---
# <a name="authenticate-with-azure-container-registry-from-azure-container-service"></a>使用來自 Azure Container Service 的 Azure Container Registry 進行驗證

搭配 Azure Container Service (AKS) 使用 Azure Container Registry (ACR) 時，需要建立驗證機制。 這份文件詳細說明在這兩個 Azure 服務之間進行驗證所建議的組態。

## <a name="grant-aks-access-to-acr"></a>將 AKS 存取權授與 ACR

建立 AKS 叢集時，也會建立服務主體來管理 Azure 資源的叢集可操作性。 此服務主體也可用於向 ACR 登錄進行驗證。 若要這樣做，需要建立角色指派，以將服務主體讀取存取權授與 ACR 資源。 

可使用下列範例來完成此作業。

```bash
#!/bin/bash

AKS_RESOURCE_GROUP=myAKSResourceGroup
AKS_CLUSTER_NAME=myAKSCluster
ACR_RESOURCE_GROUP=myACRResourceGroup
ACR_NAME=myACRRegistry

# Get the id of the service principal configured for AKS
CLIENT_ID=$(az aks show --resource-group $AKS_RESOURCE_GROUP --name $AKS_CLUSTER_NAME --query "servicePrincipalProfile.clientId" --output tsv)

# Get the ACR registry resource id
ACR_ID=$(az acr show --name $ACR_NAME --resource-group $ACR_RESOURCE_GROUP --query "id" --output tsv)

# Create role assignment
az role assignment create --assignee $CLIENT_ID --role Reader --scope $ACR_ID
```

## <a name="access-with-kubernetes-secret"></a>使用 Kubernetes 祕密進行存取

在某些情況下，AKS 使用的服務主體無法侷限在 ACR 登錄的範圍內。 對於這種情況，您可以建立唯一的服務主體，並將範圍侷限於 ACR 登錄。

下列指令碼可以用來建立服務主體。 

```bash
#!/bin/bash

ACR_NAME=myacrinstance
SERVICE_PRINCIPAL_NAME=acr-service-principal

# Populate the ACR login server and resource id. 
ACR_LOGIN_SERVER=$(az acr show --name $ACR_NAME --query loginServer --output tsv)
ACR_REGISTRY_ID=$(az acr show --name $ACR_NAME --query id --output tsv)

# Create a contributor role assignment with a scope of the ACR resource. 
SP_PASSWD=$(az ad sp create-for-rbac --name $SERVICE_PRINCIPAL_NAME --role Reader --scopes $ACR_REGISTRY_ID --query password --output tsv)

# Get the service principle client id.
CLIENT_ID=$(az ad sp show --id http://$SERVICE_PRINCIPAL_NAME --query appId --output tsv)

# Output used when creating Kubernetes secret.
echo "Service principal ID: $CLIENT_ID"
echo "Service principal password: $SP_PASSWD"
```

服務主體認證現在可以儲存在 Kubernetes[映像提取秘密][image-pull-secret]中，而且在 AKS 叢集中執行容器時可以參考。 

下列命令會建立 Kubernetes 秘密。 將伺服器名稱改為 ACR 登入伺服器將使用者名稱取代為服務主體識別碼，將密碼改為服務主體密碼。

```bash
kubectl create secret docker-registry acr-auth --docker-server <acr-login-server> --docker-username <service-principal-ID> --docker-password <service-principal-password> --docker-email <email-address>
```

Kubernetes 秘密可用於使用 `ImagePullSecrets` 參數的 Pod 部署。 

```yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: acr-auth-example
spec:
  template:
    metadata:
      labels:
        app: acr-auth-example
    spec:
      containers:
      - name: acr-auth-example
        image: myacrregistry.azurecr.io/acr-auth-example
      imagePullSecrets:
      - name: acr-auth
```

<!-- LINKS - external -->
[kubernetes-secret]: https://kubernetes.io/docs/concepts/configuration/secret/
[image-pull-secret]: https://kubernetes.io/docs/concepts/configuration/secret/#using-imagepullsecrets
