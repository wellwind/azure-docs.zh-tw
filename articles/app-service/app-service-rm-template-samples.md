---
title: "Azure Resource Manager 範本範例 - App Service | Microsoft Docs"
description: "Azure Resource Manager 範本範例 - App Service"
services: app-service
documentationcenter: app-service
author: tfitzmac
manager: timlt
editor: ggailey777
tags: azure-service-management
ms.service: app-service
ms.devlang: na
ms.topic: sample
ms.tgt_pltfrm: na
ms.workload: app-service
ms.date: 02/26/2018
ms.author: tomfitz
ms.custom: mvc
ms.openlocfilehash: e28a27b9a00164fcbba6eb5d3e5435548486ffa4
ms.sourcegitcommit: 088a8788d69a63a8e1333ad272d4a299cb19316e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/27/2018
---
# <a name="azure-resource-manager-templates-for-azure-web-apps"></a>適用於 Azure Web Apps 的 Azure Resource Manager 範本

下表包含 Azure Resource Manager 範本的連結。 如需有關在建立 web 應用程式範本時避免發生常見錯誤的建議，請參閱[使用 Azure Resource Manager 範本部署 Web 應用程式的指引](web-sites-rm-template-guidance.md)。

| | |
|-|-|
|**部署 Web 應用程式**||
| [連結至 GitHub 存放庫的 Web 應用程式](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-github-deploy)| 部署會從 GitHub 提取程式碼的 Azure Web 應用程式。 |
| [具有自訂部署位置的 Web 應用程式](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-custom-deployment-slots)| 部署具有自訂部署位置/環境的 Azure Web 應用程式。 |
|**設定 Web 應用程式**||
| [金鑰保存庫中的 Web 應用程式憑證](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-certificate-from-key-vault)| 部署金鑰保存庫祕密中的 Azure Web 應用程式憑證，並將它使用於 SSL 繫結。 |
| [具有自訂網域的 Web 應用程式](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-custom-domain)| 部署具有自訂主機名稱的 Azure Web 應用程式。 |
| [具有自訂網域和 SSL 的 Web 應用程式](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-custom-domain-and-ssl)| 部署具有自訂主機名稱的 Azure Web 應用程式，並從金鑰保存庫取得 Web 應用程式憑證，以便進行 SSL 繫結。 |
| [具有 GoLang 擴充的 Web 應用程式](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-with-golang)| 部署具有 Golang 網站擴充的 Azure web 應用程式，可讓您在 Azure 上執行於 Golang 開發的 Web 應用程式。 |
| [採用 Java 8 和 Tomcat 8 的 Web 應用程式](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-java-tomcat)| 部署已啟用 Java 8 和 Tomcat 8 的 Azure Web 應用程式，這可讓您在 Azure 中執行 Java 應用程式。 |
|**Linux Web 應用程式**||
| [在採用 MySQL 的 Linux 上部署 Web 應用程式](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-linux-managed-mysql) | 在具有適用於 MySQL 之 Azure 資料庫的 Linux 上部署 Azure Web 應用程式。 |
| [在採用 PostgreSQL 的 Linux 上部署 Web 應用程式](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-linux-managed-postgresql) | 在具有適用於 PostgreSQL 之 Azure 資料庫的 Linux 上部署 Azure Web 應用程式。 |
|**具有已連線資源的 Web 應用程式**||
| [採用 MySQL 的 Web 應用程式](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-managed-mysql)| 在具有適用於 MySQL 之 Azure 資料庫的 Windows 上部署 Azure Web 應用程式。 |
| [採用 PostgreSQL 部署 Web 應用程式](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-managed-postgresql)| 在具有適用於 PostgreSQL 之 Azure 資料庫的 Windows 上部署 Azure Web 應用程式。 |
| [ 部署 Web 應用程式與 SQL Database](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-sql-database)| 在「基本」服務層級部署 Azure Web 應用程式與 SQL Database。 |
| [具有 Blob 儲存體連線的 Web 應用程式](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-blob-connection)| 部署具有 blob 儲存體連接字串的 Azure web 應用程式，可讓您從 Web 應用程式使用 Azure Blob 儲存體。 |
| [具有 Redis 快取的 Web 應用程式](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-with-redis-cache)| 部署具有 Redis 快取的 Azure Web 應用程式。 |
|**App Service 環境**||
| [建立 App Service Environment v2](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-asev2-create) | 在虛擬網路中建立 App Service Environment v2。 |
| [建立具有 ILB 位址的 App Service Environment v2](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-asev2-ilb-create/) | 在虛擬網路中建立具有私人內部負載平衡位址的 App Service Environment v2。 |
| [設定 ILB ASE 或 ILB ASE v2 的預設 SSL 憑證](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-ase-ilb-configure-default-ssl) | 設定 ILB App Service Environment 或 ILB App Service Environment v2 的預設 SSL 憑證。 |
| | |
