---
title: "Azure Stack 的身分識別概觀 | Microsoft Docs"
description: "深入了解您可以搭配 Azure Stack 使用的身分識別系統。"
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
ms.openlocfilehash: 609ea3300806f3cfed4a3285a096f59be491c684
ms.sourcegitcommit: fbba5027fa76674b64294f47baef85b669de04b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/24/2018
---
# <a name="overview-of-identity-for-azure-stack"></a>Azure Stack 的身分識別概觀

Azure Stack 要求以 Active Directory (AD) 所支援的 Azure AD 或 Active Directory Federated Services (AD FS) 作為識別提供者。 下列概念和授權詳細資料可協助您選擇適當的識別提供者。 提供者的選擇是您第一次部署 Azure Stack 時所做的一次性決定。  

選擇 Azure AD 或 AD FS 可能受限於您用來部署 Azure Stack 的模式： 
- 當您在連線模式下部署時，可以使用 Azure AD 或 AD FS。 
- 當您在沒有網際網路連線的中斷連線模式下部署時，僅支援 AD FS。

如需有關這些選擇的詳細資訊，請根據您的 Azure Stack 環境檢視下列文章：
- Azure Stack 部署套件，請參閱[身分識別考量](azure-stack-datacenter-integration.md#identity-considerations)。
- Azure Stack 整合式系統，請參閱[適用於 Azure Stack 整合式系統的部署規劃決策](azure-stack-deployment-decisions.md)。

 
## <a name="common-concepts-for-identity"></a>身分識別的一般概念
下列各節討論識別提供者的一般概念及其在 Azure Stack 中的使用方式。
![識別提供者的術語](media/azure-stack-identity-overview/terminology.png)


### <a name="directories-tenants-and-organizations"></a>目錄租用戶和組織
目錄是保留「使用者」、「應用程式」、「群組」和「服務主體」相關資訊的容器。
 
目錄租用戶是一個「組織」像是 Microsoft 或您自己的公司。 
- Azure AD 支援多個租用戶並可支援多個組織 (各自位於自己的目錄中)。 如果您使用 Azure AD 並且有多個租用戶，可以授權應用程式和使用者從一個租用戶存取相同目錄中的其他租用戶。
- AD FS 僅支援單一租用戶，因此支援單一組織。 

### <a name="users-and-groups"></a>使用者和群組
使用者帳戶 (身分識別) 是標準帳戶，可使用使用者識別碼和密碼驗證個人。 群組可以包含使用者或其他群組。 

您建立及管理使用者和群組的方式取決於使用的身分識別解決方案。 

在 Azure Stack 中，使用者帳戶： 
- 是以 username@domain 格式建立。 雖然 AD FS 可將使用者帳戶對應到 Active Directory，但 AD FS 不支援使用 _&lt;網域>\<別名>_ 格式。 
- 可以設定為使用多重要素驗證。 
- 受限於它們首先註冊的目錄，也就是其組織目錄。
- 可以從內部部署目錄匯入。 如需詳細資訊，請參閱 Azure 文件中的[整合您的內部部署目錄與 Azure Active Directory](/azure/active-directory/connect/active-directory-aadconnect)。  

當您登入貴組織的租用戶入口網站時，會使用 https://portal.local.azurestack.external。 

### <a name="guest-users"></a>來賓使用者
來賓使用者是其他目錄租用戶中的使用者帳戶，這些使用者已獲得您目錄中資源的存取權。 若要支援來賓使用者，您必須使用 Azure AD 並啟用多租用戶支援。 若已提供支援，您可以邀請來賓使用者存取您目錄租用戶中的資源，以便與外部組織共同作業。 

雲端操作員和使用者可以使用 [Azure AD B2B 共同作業](/azure/active-directory/active-directory-b2b-what-is-azure-ad-b2b)邀請來賓使用者。 受邀的使用者可取得您目錄中文件、資源及應用程式的存取權，而您同時保有對自己的資源和資源的控制權。 

身為來賓使用者，您可以登入其他組織的目錄租用戶。 若要這麼做，您必須將該組織的目錄名稱附加至入口網站 URL。  例如，如果您屬於 contoso.com，但想要登入 Fabrikam 目錄，可使用 https://portal.local.azurestack.external/fabrikam.onmicrosoft.com。

### <a name="applications"></a>應用程式
您可以向 Azure AD 或 AD FS 註冊應用程式，然後將其提供給您組織中的使用者。 

應用程式包括：
- **Web 應用程式** – 範例包括 Azure 入口網站和 Azure Resource Manager。  它們支援 Web API 呼叫。 
- **原生用戶端** – 範例包括 Azure PowerShell、Visual Studio 和 Azure 命令列介面 (CLI)。

應用程式可支援兩種租用戶： 
- **單一租用戶**應用程式僅支援應用程式註冊所在的相同目錄中的使用者和服務。 

  > [!NOTE]    
  > 因為 AD FS 僅支援單一目錄，所以您在 AD FS 拓撲中建立的應用程式會設計為單一租用戶應用程式。
- **多租用戶**應用程式支援由以下目錄中的使用者和服務使用：應用程式註冊所在的目錄，以及其他租用戶目錄。  使用多租用戶應用程式，另一個租用戶目錄 (另一個 Azure AD 租用戶) 的使用者可以登入您的應用程式。     

  如需多租用戶的詳細資訊，請參閱[啟用多租用戶](azure-stack-enable-multitenancy.md)。  

  如需有關如何開發多租用戶應用程式的詳細資訊，請參閱[多租用戶應用程式](/azure/active-directory/develop/active-directory-devhowto-multi-tenant-overview)。

當您註冊應用程式時，會建立兩個物件：
- **應用程式物件** – 此應用程式物件是應用程式在所有租用戶中的全域表示法。 此關聯性是與軟體應用程式的一對一關聯性，且只存在於首先註冊應用程式的目錄中。

 
   
- **服務主體物件** – 服務主體是在首先註冊應用程式的目錄中針對應用程式所建立的認證。 服務主體也會建立於使用該應用程式的每個其他租用戶的目錄中。 此關聯性可以是與軟體應用程式的一對多關聯性。   

若要深入了解應用程式和服務主體，請參閱 [Azure Active Directory (Azure AD) 中的應用程式和服務主體物件](/azure/active-directory/develop/active-directory-application-objects)。 

### <a name="service-principals"></a>服務主體 
服務主要是應用程式或服務的一組「認證」，可授與 Azure Stack 中資源的存取權。 使用服務主體可區隔應用程式權限與使用應用程式的使用者權限。

服務主體會建立於使用應用程式的每個租用戶中。 服務主體會建立身分識別，以供登入及存取受到該租用戶保護的資源 (例如使用者)。   

- 單一租用戶應用程式在其首先建立的目錄中只有一個服務主體。  此服務主體會建立並同意在應用程式註冊期間使用。 
- 如果是多租用戶 Web 應用程式或 API，則會在使用者同意使用應用程式的每個租用戶中建立一個服務主體。  

服務主體的認證可以是金鑰 (透過 Azure 入口網站產生) 或憑證。  憑證的使用很適合自動化，因為憑證被視為比使用金鑰更安全。 


> [!NOTE]    
> 當您使用 AD FS 搭配 Azure Stack 時，只有系統管理員可以建立服務主體。 使用 AD FS 時，服務主體需要憑證並透過特殊權限的端點 (PEP) 來建立。 如需詳細資訊，請參閱[為 Azure Stack 提供應用程式存取](azure-stack-create-service-principals.md)。

若要了解 Azure Stack 的服務主體，請參閱[建立服務主體](azure-stack-create-service-principals.md)。


### <a name="services"></a>服務
在 Azure Stack 中與識別提供者互動的服務，識別提供者會將其註冊為應用程式。 如同應用程式，註冊可讓服務向身分識別系統進行驗證。 

所有 Azure 服務都會使用 [OpenID Connect](/azure/active-directory/develop/active-directory-protocols-openid-connect-code) 通訊協定和 [JSON Web 權杖](/azure/active-directory/develop/active-directory-token-and-claims) (JWT) 來建立其身分識別。 由於 Azure AD 和 AD FS 使用的通訊協定一致，所以您可使用 Azure [Active Directory 驗證程式庫](/azure/active-directory/develop/active-directory-authentication-libraries) (ADAL) 來向內部部署環境或 Azure 進行驗證 (已連線的情況下)。 ADAL 也可讓您使用 Azure PowerShell 和 CLI 等工具進行跨雲端和內部部署資源管理。 


### <a name="identities-and-your-identity-system"></a>身分識別和身分識別系統 
Azure Stack 的身分識別包括使用者帳戶、群組和服務主體。 當您安裝 Azure Stack 時，有數個內建應用程式和服務會在目錄租用戶中自動向您的識別提供者註冊。 有些註冊的服務可用於管理。 其他服務則可供使用者使用。 預設註冊會提供核心服務身分識別，這類身分識別可彼此互動，以及與您稍後新增的身分識別互動。
如果您設定採用多租用戶的 Azure AD，則有些應用程式會傳播到新的目錄。  

## <a name="authentication-and-authorization"></a>驗證和授權
 

### <a name="authentication-by-applications-and-users"></a>應用程式和使用者的驗證
  
![Azure Stack 層次之間的身分識別](media/azure-stack-identity-overview/identity-layers.png)

對應用程式和使用者而言，Azure Stack 的架構可以用四個層次來說明。 各層之間的互動可以使用不同類型的驗證。


|層次    |各層之間的驗證  |
|---------|---------|
|工具與用戶端，像是管理入口網站     | 若要存取或修改 Azure Stack 中的資源，工具和用戶端會使用 [JSON Web 權杖](/azure/active-directory/develop/active-directory-token-and-claims) (JWT) 來呼叫 Azure Resource Manager。  <br><br> Azure Resource Manager 會驗證 JWT 並查看所核發權杖中的*宣告*，以評估使用者或服務主體在 Azure Stack 中具有的授權層級。        |
|Azure Resource Manager 與其核心服務     |Azure Resource Manager 會與資源提供者通訊，以傳輸使用者的通訊。 <br><br> 傳輸會透過 [Azure Resource Manager 範本](/azure/azure-stack/user/azure-stack-arm-templates.md)使用「直接命令式」呼叫或「宣告式」呼叫。|
|資源提供者     |傳遞至資源提供者的呼叫會以憑證型驗證進行保護。 <br><br>Azure Resource Manager 和資源提供者而後會持續透過 API 通訊。 對於從 Azure Resource Manager 接收的每個呼叫，資源提供者會使用該憑證來驗證呼叫。|
|基礎結構和商務邏輯     |資源提供者會使用其所選的驗證模式來與商務邏輯和基礎結構通訊。 Azure Stack 隨附的預設資源提供者會使用 Windows 驗證來保護此通訊。|

![驗證所需的資訊](media/azure-stack-identity-overview/authentication.png)


### <a name="authenticate-to-azure-resource-manager"></a>向 Azure Resource Manager 進行驗證
若要向識別提供者進行驗證並收到 JWT，您必須具有下列資訊： 
1.  **身分識別系統 (授權單位) 的 URL** – 可以觸達識別提供者的 URL。 例如，*&lt;https://login.windows.net>*。 
2.  **Azure Resource Manager 的應用程式識別碼 URI** – 已向識別提供者註冊之 Azure Resource Manager 的唯一識別碼，而且是每個 Azure Stack 安裝中的唯一識別碼。
3.  **認證** – 這是您用來向識別提供者驗證的認證。  
4.  **Azure Resource Manager 的 URL** – URL 是 Azure Resource Manager 服務的位置。 例如，*&lt;https://management.azure.com>* 或 *&lt;https://management.local.azurestack.external>*。

當主體 (用戶端、應用程式或使用者) 提出驗證要求以存取資源時，該要求必須包含：
- 主體的認證。
- 其想要存取之資源的應用程式識別碼 URI。  

認證會經由識別提供者驗證。 識別提供者也會驗證應用程式識別碼 URI 是否用於已註冊的應用程式，以及主體是否擁有正確權限可取得該資源的權杖。 如果要求有效，就會授與 JSON Web 權杖。 

此權杖必須接著傳入對 Azure Resource Manager 的要求標頭中。 Azure Resource Manager 會執行下列驗證 (沒有特定順序)：
- 驗證 issuer (iss) 宣告，以確認權杖來自正確的識別提供者。 
- 驗證 audience (aud) 宣告，以確認權杖已簽發給 Azure Resource Manager。 
- 驗證 JWT 是否使用透過 OpenID 設定的憑證來簽署，而且為 Azure Resource Manager 所知。 
- 檢閱 *issued at* (iat) and *expiry* 宣告，以確認權杖為作用中並可接受。 

完成所有驗證時，Azure Resource Manager 會使用 objected (oid) 和 groups 宣告來產生主體可以存取的資源清單。 

![權杖交換通訊協定的圖表](media/azure-stack-identity-overview/token-exchange.png)


### <a name="use-role-based-access-control"></a>使用角色型存取控制  
Azure Stack 中的角色型存取控制 (RBAC) 與 Microsoft Azure 中的實作一致。  您可以藉由將適當的 RBAC 角色指派給使用者、群組及應用程式，來管理資源的存取權。 如需如何使用 RBAC 搭配 Azure Stack 的相關資訊，請檢閱下列文章：
- [在 Azure 入口網站中開始使用角色型存取控制](/azure/active-directory/role-based-access-control-what-is)。
- [使用角色型存取控制來管理 Azure 訂用帳戶資源的存取](/azure/active-directory/role-based-access-control-configure)。
- [建立 Azure 角色型存取控制的自訂角色](/azure/active-directory/role-based-access-control-custom-roles)。
- 在 Azure Stack 中[管理角色型存取控制](azure-stack-manage-permissions.md)。


### <a name="authenticate-with-azure-powershell"></a>使用 Azure PowerShell 進行驗證  
如需有關使用 Azure PowerShell 向 Azure Stack 驗證的詳細資料，請參閱[設定 Azure Stack 使用者的 PowerShell 環境](azure-stack-powershell-configure-user.md)。

### <a name="authenticate-with-azure-cli"></a>使用 Azure CLI 進行驗證
如需有關使用 Azure PowerShell 向 Azure Stack 驗證的詳細資料，請參閱[安裝和設定 CLI 以便搭配 Azure Stack 使用](/azure/azure-stack/user/azure-stack-connect-cli.md)。

## <a name="next-steps"></a>後續步驟
- [身分識別架構](azure-stack-identity-architecture.md)   
- [資料中心整合 - 身分識別](azure-stack-integrate-identity.md)




