---
title: "在 Azure Log Analytics 中管理工作區 | Microsoft Docs"
description: "您可以在 Azure Log Analytics 中使用有關使用者、帳戶、工作區及 Azure 帳戶的各種系統管理工作來管理工作區。"
services: log-analytics
documentationcenter: 
author: MGoedtel
manager: carmonm
editor: 
ms.assetid: d0e5162d-584b-428c-8e8b-4dcaa746e783
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 09/12/2017
ms.author: magoedte
ms.openlocfilehash: 5121535768b7fb430486c1c2c623e1a3a488858f
ms.sourcegitcommit: d87b039e13a5f8df1ee9d82a727e6bc04715c341
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/21/2018
---
# <a name="manage-workspaces"></a>管理工作區

若要管理對 Log Analytics 的存取，請執行與工作區相關的各種系統管理工作。 本文提供管理工作區的最佳做法和程序。 工作區本質上是一個容器，包含帳戶資訊和帳戶的簡單組態資訊。 您或組織的其他成員可能會使用多個工作區來管理從所有或部分 IT 基礎結構收集而來的不同資料。

若要建立工作區，您需要︰

1. 擁有 Azure 訂用帳戶。
2. 選擇工作區名稱。
3. 建立工作區與訂用帳戶的關聯。
4. 選擇地理位置。

## <a name="determine-the-number-of-workspaces-you-need"></a>判斷您需要的工作區數目
工作區是一種 Azure 資源，也是 Azure 入口網站中收集、彙總、分析及呈現資料的容器。

每個 Azure 訂用帳戶可以有多個工作區，而且您可以存取多個工作區。 您先前只能分析來自目前工作區內的資料，這項限制讓您無法跨訂用帳戶中定義的多個工作區執行查詢。 現在，您可以[跨多個工作區執行查詢](https://docs.microsoft.com/en-us/azure/log-analytics/log-analytics-cross-workspace-search)，這提供了一個全系統的資料檢視。 本節描述何時有利於建立多個工作區。

現今的工作區可提供︰

* 資料儲存體的地理位置
* 計費的細微度
* 資料隔離
* 設定範圍

根據上述特性，您在下列情況下可能想要建立多個工作區︰

* 您是一家全球性公司，基於資料主權或合規性理由，您需要將資料儲存在特定區域。
* 您使用 Azure，想要將工作區和它所管理的 Azure 資源放在相同區域中，以避免輸出資料傳輸費用。
* 您想要根據不同部門或事業群的使用量來配置費用。 當您建立每個部門或事業群的工作區時，Azure 帳單和用量表會個別顯示每個工作區的費用。
* 您是受控服務提供者，需要將您管理的每個客戶的 Log Analytics 資料和其他客戶的資料保持隔離。
* 您管理多個客戶，想要讓每個客戶/部門/事業群只看到他們自己的資料，而不是其他客戶/部門/事業群的資料。

使用代理程式收集資料時，您可以[設定每個代理程式向一或多個工作區回報](log-analytics-windows-agent.md)。

如果您使用 System Center Operations Manager，每個 Operations Manager 管理群組只能連接一個工作區。 不過，您可以將電腦上的 Microsoft Monitoring Agent 設定為同時向 Operations Manager 和不同的 Log Analytics 工作區回報。  

### <a name="workspace-information"></a>工作區資訊

您可以在 Azure 入口網站中檢視工作區的相關詳細資料。 

#### <a name="view-workspace-information-in-the-azure-portal"></a>在 Azure 入口網站中檢視工作區資訊

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 按一下 [所有服務]。  在資源清單中輸入 **Log Analytics**。 當您開始輸入時，清單會根據您輸入的文字進行篩選。 按一下 [Log Analytics]。  
    ![顯示 Azure 左側功能表的螢幕擷取畫面](./media/log-analytics-manage-access/hub.png)  
3. 在 [Log Analytics 訂用帳戶] 頁面中選取工作區。
4. [工作區] 頁面會顯示工作區的詳細資料和其他資訊的連結。  
    ![工作區詳細資料](./media/log-analytics-manage-access/workspace-details.png)  


## <a name="manage-accounts-and-users"></a>管理帳戶和使用者
每個工作區可以有多個相關聯的帳戶，而每個帳戶 (Microsoft 帳戶或組織帳戶) 可以存取多個工作區。

根據預設，建立工作區的 Microsoft 帳戶或組織帳戶會變成工作區的管理員。

Log Analytics 工作區有兩個存取控制權限模型︰

1. 舊版 Log Analytics 使用者角色
2. [Azure 角色型存取](../active-directory/role-based-access-control-configure.md)

下表摘要說明可使用每個權限模型設定的存取權︰

|                          | Log Analytics 入口網站 | Azure 入口網站 | API (包括 PowerShell) |
|--------------------------|----------------------|--------------|----------------------------|
| Log Analytics 使用者角色 | yes                  | 否           | 否                         |
| Azure 角色型存取  | yes                  | yes          | yes                        |

> [!NOTE]
> Log Analytics 將改為使用 Azure 角色型存取來作為權限模型，以取代 Log Analytics 使用者角色。
>
>

舊版 Log Analytics 使用者角色只能控制 [Log Analytics 入口網站](https://mms.microsoft.com)中所執行活動的存取權。

下列活動也需要 Azure 權限︰

| 動作                                                          | 所需的 Azure 權限 | 注意 |
|-----------------------------------------------------------------|--------------------------|-------|
| 新增及移除管理解決方案                        | `Microsoft.Resources/deployments/*` <br> `Microsoft.OperationalInsights/*` <br> `Microsoft.OperationsManagement/*` <br> `Microsoft.Automation/*` <br> `Microsoft.Resources/deployments/*/write` | |
| 變更定價層                                       | `Microsoft.OperationalInsights/workspaces/*/write` | |
| 檢視 [備份] 和 [Site Recovery] 解決方案圖格中的資料 | 系統管理員/共同管理員 | 存取使用傳統部署模型所部署的資源 |
| 在 Azure 入口網站中建立工作區                        | `Microsoft.Resources/deployments/*` <br> `Microsoft.OperationalInsights/workspaces/*` ||


### <a name="managing-access-to-log-analytics-using-azure-permissions"></a>使用 Azure 權限管理對 Log Analytics 的存取
若要使用 Azure 權限授與 Log Analytics 工作區的存取權，請遵循[使用角色指派來管理 Azure 訂用帳戶資源的存取權](../active-directory/role-based-access-control-configure.md)中的步驟。

Azure 有兩個適用於 Log Analytics 的內建使用者角色：
- Log Analytics 讀者
- Log Analytics 參與者

*Log Analytics 讀者*角色的成員可以：
- 檢視及搜尋所有監視資料 
- 檢視監視設定，包括檢視所有 Azure 資源上檢視的 Azure 診斷組態。

| 類型    | 權限 | 說明 |
| ------- | ---------- | ----------- |
| 動作 | `*/read`   | 檢視所有資源和資源組態的能力。 包括檢視： <br> 虛擬機器擴充功能 <br> 在資源上設定 Azure 診斷 <br> 所有資源的所有屬性和設定 |
| 動作 | `Microsoft.OperationalInsights/workspaces/analytics/query/action` | 執行記錄搜尋 v2 查詢的能力 |
| 動作 | `Microsoft.OperationalInsights/workspaces/search/action` | 執行記錄搜尋 v1 查詢的能力 |
| 動作 | `Microsoft.Support/*` | 開啟支援案例的能力 |
|不是動作 | `Microsoft.OperationalInsights/workspaces/sharedKeys/read` | 防止讀取使用資料收集 API 和安裝代理程式時所需的工作區金鑰 |


*Log Analytics 參與者*角色的成員可以：
- 讀取所有監視資料 
- 建立和設定自動化帳戶
- 新增及移除管理解決方案
- 讀取儲存體帳戶金鑰 
- 設定 Azure 儲存體的記錄集合
- 編輯 Azure 資源的監視設定，包括
  - 將 VM 擴充功能新增至 VM
  - 在所有 Azure 資源上設定 Azure 診斷

> [!NOTE] 
> 您可以使用將虛擬機器擴充功能新增至虛擬機器的功能，取得虛擬機器的完整控制權。

| 權限 | 說明 |
| ---------- | ----------- |
| `*/read`     | 檢視所有資源和資源組態的能力。 包括檢視： <br> 虛擬機器擴充功能 <br> 在資源上設定 Azure 診斷 <br> 所有資源的所有屬性和設定 |
| `Microsoft.Automation/automationAccounts/*` | 建立及設定 Azure 自動化帳戶的能力，包括新增和編輯 Runbook |
| `Microsoft.ClassicCompute/virtualMachines/extensions/*` <br> `Microsoft.Compute/virtualMachines/extensions/*` | 新增、更新及移除虛擬機器擴充功能，包括 Microsoft Monitoring Agent 擴充功能和 OMS Agent for Linux 擴充功能 |
| `Microsoft.ClassicStorage/storageAccounts/listKeys/action` <br> `Microsoft.Storage/storageAccounts/listKeys/action` | 檢視儲存體帳戶金鑰。 需具備此權限，才能設定 Log Analytics 以讀取 Azure Blob 儲存體帳戶中的記錄 |
| `Microsoft.Insights/alertRules/*` | 新增、更新和移除警示規則 |
| `Microsoft.Insights/diagnosticSettings/*` | 在 Azure 資源上新增、更新和移除診斷設定 |
| `Microsoft.OperationalInsights/*` | 新增、更新和移除 Log Analytics 工作區的組態 |
| `Microsoft.OperationsManagement/*` | 新增及移除管理解決方案 |
| `Microsoft.Resources/deployments/*` | 建立及刪除部署。 需具備此權限，才能新增及移除解決方案、工作區和自動化帳戶 |
| `Microsoft.Resources/subscriptions/resourcegroups/deployments/*` | 建立及刪除部署。 需具備此權限，才能新增及移除解決方案、工作區和自動化帳戶 |

若要新增及移除某個使用者角色的使用者，必須具備 `Microsoft.Authorization/*/Delete` 和 `Microsoft.Authorization/*/Write` 權限。

使用下列角色，賦予使用者不同範圍的存取權：
- 訂用帳戶 - 存取訂用帳戶中的所有工作區
- 資源群組 - 存取資源群組中的所有工作區
- 資源 - 僅存取指定的工作區

使用[自訂角色](../active-directory/role-based-access-control-custom-roles.md)建立具備所需特定權限的角色。

### <a name="azure-user-roles-and-log-analytics-portal-user-roles"></a>Azure 使用者角色和 Log Analytics 入口網站使用者角色
如果您至少具有 Log Analytics 工作區的 Azure 讀取權限，您可以在檢視 Log Analytics 工作區時按一下 **OMS 入口網站**工作來開啟 OMS 入口網站。

開啟 OMS 入口網站時，您會改為使用舊版 Log Analytics 使用者角色。 如果您沒有 Log Analytics 入口網站的角色指派，服務會[檢查您在工作區上擁有的 Azure 權限](https://docs.microsoft.com/rest/api/authorization/permissions#Permissions_ListForResource)。
您在 OMS 入口網站中的角色指派是使用如下方式來決定︰

| 條件                                                   | 指派的 Log Analytics 使用者角色 | 注意 |
|--------------------------------------------------------------|----------------------------------|-------|
| 您的帳戶屬於舊版 Log Analytics 使用者角色     | 指定的 Log Analytics 使用者角色 | |
| 您的帳戶不屬於舊版 Log Analytics 使用者角色 <br> 工作區的完整 Azure 權限 (`*` 權限<sup>1</sup>) | 系統管理員 ||
| 您的帳戶不屬於舊版 Log Analytics 使用者角色 <br> 工作區的完整 Azure 權限 (`*` 權限<sup>1</sup>) <br> 非 `Microsoft.Authorization/*/Delete` 和 `Microsoft.Authorization/*/Write` 的動作 | 參與者 ||
| 您的帳戶不屬於舊版 Log Analytics 使用者角色 <br> Azure 讀取權限 | 唯讀 ||
| 您的帳戶不屬於舊版 Log Analytics 使用者角色 <br> 未經辨識的 Azure 權限 | 唯讀 ||
| 適用於雲端解決方案提供者 (CSP) 管理的訂用帳戶 <br> 您用來登入的帳戶位於連結至工作區的 Azure Active Directory 中 | 系統管理員 | 通常是 CSP 的客戶 |
| 適用於雲端解決方案提供者 (CSP) 管理的訂用帳戶 <br> 您用來登入的帳戶並非位於連結至工作區的 Azure Active Directory 中 | 參與者 | 通常是 CSP |

<sup>1</sup> 如需角色定義的詳細資訊，請參閱 [Azure 權限](../active-directory/role-based-access-control-custom-roles.md)。 在評估角色時，`*` 的動作不等於 `Microsoft.OperationalInsights/workspaces/*`。

請留意 Azure 入口網站的下列幾點︰

* 當您使用 http://mms.microsoft.com 登入 OMS 入口網站時，您會看到 [選取工作區] 清單。 此清單只包含您具有 Log Analytics 使用者角色的工作區。 若要查看您可利用 Azure 訂用帳戶存取的工作區，則必須將租用戶指定為 URL 的一部分。 例如：`mms.microsoft.com/?tenant=contoso.com`。 租用戶識別碼通常是您用來登入的電子郵件地址的最後一部分。
* 如果您想要利用 Azure 權限直接瀏覽到您有存取權的入口網站，您需要在 URL 中指定此資源。 就可以使用 PowerShell 取得此 URL。

  例如： `(Get-AzureRmOperationalInsightsWorkspace).PortalUrl`。

  URL 看起來像這樣：`https://eus.mms.microsoft.com/?tenant=contoso.com&resource=%2fsubscriptions%2faaa5159e-dcf6-890a-a702-2d2fee51c102%2fresourcegroups%2fdb-resgroup%2fproviders%2fmicrosoft.operationalinsights%2fworkspaces%2fmydemo12`

### <a name="managing-users-in-the-oms-portal"></a>在 OMS 入口網站中管理使用者
您可以在 [設定] 頁面中 [帳戶] 索引標籤之下的 [管理使用者] 索引標籤上管理使用者與群組。   

![管理使用者](./media/log-analytics-manage-access/setup-workspace-manage-users.png)


#### <a name="add-a-user-to-an-existing-workspace"></a>新增使用者到現有的工作區
使用下列步驟，將使用者或群組新增至工作區。

1. 在 OMS 入口網站中，按一下 [設定] 圖格。
2. 按一下 [帳戶] 索引標籤，然後按一下 [管理使用者] 索引標籤。
3. 在 [管理使用者] 區段中，選擇要新增的帳戶類型：[組織帳戶]、[Microsoft 帳戶]、[Microsoft 支援服務]。

   * 如果您選擇 [Microsoft 帳戶]，請輸入與該 Microsoft 帳戶相關聯的使用者電子郵件地址。
   * 如果您選擇 [組織帳戶]，請輸入使用者/群組的部分名稱或電子郵件別名，而且相符使用者和群組的清單會顯示在下拉式方塊中。 選取使用者或群組。
   * 使用「Microsoft 支援服務」讓 Microsoft 支援服務工程師或其他 Microsoft 員工暫時存取您的工作區，以協助進行疑難排解。

     > [!NOTE]
     > 為了獲得最佳效能，請將與單一 OMS 帳戶相關聯的 Active Directory 群組數目限制為三個：一個給系統管理員、一個給參與者、一個給唯讀使用者。 使用太多群組可能會影響 Log Analytics 的效能。
     >
     >
4. 選擇要新增的使用者或群組類型：[系統管理員]、[參與者] 或 [唯讀使用者]。  
5. 按一下 [新增] 。

   如果您新增 Microsoft 帳戶，系統將會傳送一封加入工作區的邀請至您提供的電子郵件。 使用者遵循邀請中的指示加入 OMS 之後，使用者就可以存取此工作區。
   如果您要新增組織帳戶，則使用者可以立即存取 Log Analytics。  

#### <a name="edit-an-existing-user-type"></a>編輯現有的使用者類型
您可以為與您的 OMS 帳戶相關聯的使用者變更帳戶角色。 您有下列角色選項：

* *管理員*：可以管理使用者、檢視和處理所有警示，以及新增和移除伺服器
* *參與者*：可以檢視和處理所有警示，以及新增和移除伺服器
* *唯讀使用者*︰標示為唯讀的使用者無法︰

  1. 新增/移除解決方案。 方案庫會隱藏起來。
  2. 在 [我的儀表板] 上新增/修改/移除圖格。
  3. 檢視 [設定] 頁面。 這些頁面會隱藏起來。
  4. 在 [搜尋] 檢視中，PowerBI 組態、已儲存的搜尋和警示工作會隱藏起來。

#### <a name="to-edit-an-account"></a>編輯帳戶
1. 在 OMS 入口網站中，按一下 [設定] 圖格。
2. 按一下 [帳戶] 索引標籤，然後按一下 [管理使用者] 索引標籤。
3. 選取您要變更的使用者角色。
4. 在確認對話方塊中，按一下 [是]。

### <a name="remove-a-user-from-a-workspace"></a>從工作區移除使用者
使用下列步驟，從工作區移除使用者。 移除使用者並不會關閉工作區。 而會移除使用者與工作區之間的關聯。 如果使用者與多個工作區相關聯，該使用者還是可以登入 OMS，並看到其他工作區。

1. 在 OMS 入口網站中，按一下 [設定] 圖格。
2. 按一下 [帳戶] 索引標籤，然後按一下 [管理使用者] 索引標籤。
3. 按一下您要移除之使用者名稱旁邊的 [移除]。
4. 在確認對話方塊中，按一下 [是]。

### <a name="add-a-group-to-an-existing-workspace"></a>將群組新增至現有的工作區
1. 在上一節＜將使用者新增至現有工作區＞中，遵循步驟 1 至 4。
2. 在 [選擇使用者/群組] 下方，選取 [群組]。  
   ![add a group to an existing workspace](./media/log-analytics-manage-access/add-group.png)
3. 輸入您想要新增之群組的顯示名稱或電子郵件地址。
4. 在清單結果中選取群組，然後按一下 [新增]。

## <a name="link-an-existing-workspace-to-an-azure-subscription"></a>將現有的工作區連結到 Azure 訂用帳戶
建立時，所有在 2016 年 9 月 26 日之後建立的工作區都必須連結到 Azure 訂用帳戶。 當您登入時，在這個日期之前建立的工作區必須連結到工作區。 如果您從 Azure 入口網站建立工作區，或將工作區連結到 Azure 訂用帳戶，則會連結 Azure Active Directory 作為您的組織帳戶。

### <a name="to-link-a-workspace-to-an-azure-subscription-in-the-oms-portal"></a>在 OMS 入口網站中將工作區連結到 Azure 訂用帳戶

- 當您登入 OMS 入口網站時，系統會提示您選取 Azure 訂用帳戶。 選取您想要連結至工作區的訂用帳戶，然後按一下 [連結]。  
    ![連結 Azure 訂用帳戶](./media/log-analytics-manage-access/required-link.png)

    > [!IMPORTANT]
    > 若要連結工作區，您的 Azure 帳戶必須已經能夠存取您想要連結的工作區。  換句話說，您用來存取 Azure 入口網站的帳戶必須與用來存取工作區的帳戶**相同**。 否則，請參閱[將使用者新增至現有工作區](#add-a-user-to-an-existing-workspace)。

### <a name="to-link-a-workspace-to-an-azure-subscription-in-the-azure-portal"></a>在 Azure 入口網站中將工作區連結到 Azure 訂用帳戶
1. 登入 [Azure 入口網站](http://portal.azure.com)。
2. 瀏覽 **Log Analytics**，然後加以選取。
3. 您會看到現有工作區清單。 按一下 [新增] 。  
   ![工作區清單](./media/log-analytics-manage-access/manage-access-link-azure01.png)
4. 在 [OMS 工作區] 下方，按一下 [或連結現有的]。  
   ![連結現有的](./media/log-analytics-manage-access/manage-access-link-azure02.png)
5. 按一下 [進行必要設定] 。  
   ![configure required settings](./media/log-analytics-manage-access/manage-access-link-azure03.png)
6. 您會看到尚未連結到您 Azure 帳戶的工作區清單。 選取工作區。  
   ![選取工作區](./media/log-analytics-manage-access/manage-access-link-azure04.png)
7. 需要時，您可以變更下列項目的值：
   * 訂用帳戶
   * 資源群組
   * 位置
   * 定價層  
     ![變更值](./media/log-analytics-manage-access/manage-access-link-azure05.png)
8. 按一下 [SERVICEPRINCIPAL] 。 工作區現在已連結到您的 Azure 帳戶。

> [!NOTE]
> 如果您看不到想要連結的工作區，則您的 Azure 訂用帳戶沒有使用 OMS 入口網站建立之工作區的存取權。  若要從 OMS 入口網站授與此帳戶的存取權，請參閱[將使用者新增至現有的工作區](#add-a-user-to-an-existing-workspace)。
>
>

## <a name="change-an-azure-active-directory-organization-for-a-workspace"></a>變更工作區的 Azure Active Directory 組織

您可以變更工作區的 Azure Active Directory 組織。 變更 Azure Active Directory 組織可讓您將該目錄中的使用者和群組新增至工作區。

### <a name="to-change-the-azure-active-directory-organization-for-a-workspace"></a>變更工作區的 Azure Active Directory 組織

1. 在 OMS 入口網站的 [設定] 頁面上，按一下 [帳戶]，然後按一下 [管理使用者] 索引標籤。  
2. 檢閱組織帳戶的資訊，然後按一下 [變更組織]。  
    ![變更組織](./media/log-analytics-manage-access/manage-access-add-adorg01.png)
3. 輸入您 Azure Active Directory 網域之系統管理員的身分識別資訊。 之後，您會看到通知，指出您的工作區連結到 Azure Active Directory 網域。  
    ![連結的工作區通知](./media/log-analytics-manage-access/manage-access-add-adorg02.png)

## <a name="next-steps"></a>後續步驟
* 請參閱[了解資料使用方式](log-analytics-usage.md)，以了解如何分析解決方案所收集和電腦所送來的大量資料。
* [從 Azure Marketplace 新增 Log Analytics 管理解決方案](log-analytics-add-solutions.md)，以新增功能和收集資料。
