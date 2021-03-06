---
title: "Azure Stack 的身分識別架構 | Microsoft Docs"
description: "深入了解您可以搭配 Azure Stack 使用的身分識別架構。"
services: azure-stack
documentationcenter: 
author: brenduns
manager: femila
editor: 
ms.assetid: 
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 2/22/2018
ms.author: brenduns
ms.reviewer: 
ms.openlocfilehash: 0f3e28f7726afab02211902b5ba2e478ae8065df
ms.sourcegitcommit: fbba5027fa76674b64294f47baef85b669de04b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/24/2018
---
# <a name="identity-architecture-for-azure-stack"></a>Azure Stack 的身分識別架構
在您選擇要搭配 Azure Stack 使用的身分識別提供者之前，請先了解 Azure Active Directory (Azure AD) 與 Active Directory Federated Services (AD FS) 之間選項的重要差異。 

## <a name="capabilities-and-limitations"></a>功能和限制 
您選擇的身分識別提供者可能會限制您的選項，包括支援多租用戶。 

  

|功能或案例        |Azure AD  |AD FS  |
|------------------------------|----------|-------|
|已連線至網際網路     |yes       |選用|
|支援多租用戶     |yes       |否      |
|Marketplace 摘要整合       |yes       |是 - 要求使用[離線 Marketplace 摘要整合](azure-stack-download-azure-marketplace-item.md#download-marketplace-items-in-a-disconnected-or-a-partially-connected-scenario-with-limited-internet-connectivity)工具。|
|支援 Active Directory 驗證程式庫 (ADAL) |yes |yes|
|支援 Azure 命令列介面 (CLI)、Visual Studio (VS) 和 PowerShell 等工具  |yes |yes|
|透過入口網站建立服務主體     |yes |否|
|建立包含憑證的服務主體      |yes |yes|
|建立包含祕密 (金鑰) 的服務主體    |yes |否|
|應用程式可以使用 Graph 服務           |yes |否|
|應用程式可以使用身分識別提供者進行登入 |yes |是 - 要求應用程式與內部部署 AD FS 同盟。 |

## <a name="topologies"></a>拓撲
下列各節提供有關您可使用的身分識別拓撲相關資訊。

### <a name="azure-ad--single-tenant"></a>Azure AD – 單一租用戶 
根據預設，當您安裝 Azure Stack 並使用 Azure AD 時，Azure Stack 會使用單一租用戶拓撲。 

在下列情況下，單一租用戶拓撲很有用：
- 所有使用者都屬於相同的租用戶。
- 服務提供者主控組織的 Azure Stack 執行個體。  

![使用單一租用戶拓撲搭配 Azure AD 的 Azure Stack 拓樸](media/azure-stack-identity-architecture/single-tenant.png)

使用此拓撲：
- Azure Stack 會將所有的應用程式和服務註冊到相同的 Azure AD 租用戶目錄。 
- Azure Stack 只會驗證該目錄中的使用者和應用程式，包括權杖。 
- 系統管理員 (雲端操作員) 和租用戶使用者的身分識別位於相同的目錄租用戶中。 
- 若要讓其他目錄的使用者能夠存取此 Azure Stack 環境，您必須[邀請使用者](azure-stack-identity-overview.md#guest-users)成為租用戶目錄的來賓。  

### <a name="azure-ad--multi-tenant"></a>Azure AD – 多租用戶
雲端操作員可將 Azure Stack 設定為允許一或多個組織的租用戶存取應用程式。 使用者可透過使用者入口網站存取應用程式。 在此組態中，管理員入口網站 (由雲端操作員使用) 受限於單一目錄中的使用者。 

在下列情況下，多租用戶拓撲很有用：
- 服務提供者想要允許多個組織中的使用者存取 Azure Stack。

![使用多租用戶拓撲搭配 Azure AD 的 Azure Stack 拓樸](media/azure-stack-identity-architecture/multi-tenant.png)

使用此拓撲：
- 資源的存取權應該以每個組織為基礎。 
- 某個組織中的使用者不能將資源的存取權授與給其組織外部的使用者。  
- 系統管理員 (雲端操作員) 的身分識別可以位於與使用者身分識別不同的目錄租用戶中。 此分隔可提供識別提供者層級的帳戶隔離。 
 
### <a name="ad-fs"></a>AD FS  
下列其中一個條件成立時需要 AD FS 拓撲：
- Azure Stack 未連線到網際網路。
- Azure Stack 可以連線到網際網路，但您選擇使用 AD FS 作為您的識別提供者。
  
![使用 AD FS 的 Azure Stack 拓樸](media/azure-stack-identity-architecture/adfs.png)

使用此拓撲：
- 若要支援使用於生產環境中，您必須透過同盟信任來整合內建 Azure Stack AD FS 執行個體與 Active Directory (AD) 所支援的現有 AD FS 執行個體。 
- 您可以整合 Azure Stack 中的 Graph 服務與現有的 AD。  您也可以使用以 OData 為基礎的圖形 API 服務，該服務支援與 Azure AD Graph API 一致的 API。  

  若要與您的 AD 互動，圖形 API 需要您的 AD 中具有 AD 唯讀權限的使用者認證。 
  - 內建 AD FS 是以 Server 2016 為基礎。 
  - 您的 AD FS 和 AD 必須以 Server 2012 或更早版本為基礎。 
  
  AD 與內建 AD FS 之間的互動不限於 OpenID Connect，並可使用任何相互支援的通訊協定。  
  - 使用者帳戶是在內部部署 AD 中進行建立和管理。
  - 應用程式的服務主體和註冊是在內建 AD 中進行管理。



## <a name="next-steps"></a>後續步驟
- [身分識別概觀](azure-stack-identity-overview.md)   
- [資料中心整合 - 身分識別](azure-stack-integrate-identity.md)