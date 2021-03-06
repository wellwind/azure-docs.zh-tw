---
title: "「停機期間啟動/停止 VM」解決方案 (預覽) | Microsoft Docs"
description: "此虛擬機器管理解決方案會依照排程啟動和停止 Azure Resource Manager 虛擬機器，並從 Log Analytics 主動監視。"
services: automation
documentationCenter: 
authors: eslesar
manager: carmonm
editor: 
ms.assetid: 06c27f72-ac4c-4923-90a6-21f46db21883
ms.service: automation
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/18/2017
ms.author: magoedte
ms.openlocfilehash: 7ffd424de2a7224b5ac50fa228289c5397092b2e
ms.sourcegitcommit: d87b039e13a5f8df1ee9d82a727e6bc04715c341
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/21/2018
---
# <a name="startstop-vms-during-off-hours-solution-preview-in-azure-automation"></a>Azure 自動化中的「停機期間啟動/停止 VM」解決方案 (預覽)

「停機期間啟動/停止 VM」解決方案可根據使用者定義的排程啟動和停止 Azure 虛擬機器、透過 Azure Log Analytics 提供深入解析，以及使用 [SendGrid](https://azuremarketplace.microsoft.com/marketplace/apps/SendGrid.SendGrid?tab=Overview) \(英文\) 傳送選擇性的電子郵件。 針對大多數的案例，它皆能同時支援 Azure Resource Manager 和傳統 VM。 

此解決方案可針對想要使用無伺服器的低成本資源來降低成本的使用者，提供非集中式的自動化選項。 使用此解決方案，您可以：

* 對虛擬機器進行排程以啟動和停止。
* 使用 Azure 標記以遞增順序對虛擬機器進行排程以啟動和停止 (不支援傳統虛擬機器)。
* 根據低 CPU 使用率自動停止虛擬機器。

## <a name="prerequisites"></a>先決條件

- Runbook 會使用 [Azure 執行身分帳戶](automation-offering-get-started.md#authentication-methods)。  執行身分帳戶是慣用的驗證方法，因為它使用憑證驗證，而不是會過期或經常變更的密碼。  

- 這個解決方案只管理與 Azure 自動化帳戶位於相同訂用帳戶中的虛擬機器。  

- 這個解決方案只能部署到下列 Azure 區域：澳大利亞東南部、加拿大中部、印度中部、美國東部、日本東部、東南亞、英國南部和西歐。  
    
    > [!NOTE]
    > 管理 VM 排程的 Runbook 能夠以任何區域中的 VM 為目標。  

- 若要在啟動和停止虛擬機器的 Runbook 完成時傳送電子郵件通知，從 Azure Marketplace 進行上線時，請選取 [是] 以部署 SendGrid。 

    > [!IMPORTANT]
    > SendGrid 是第三方服務。 如需支援，請連絡 [SendGrid](https://sendgrid.com/contact/)。  
    >
   
    SendGrid 的限制如下：

    * 每個訂用帳戶的每個使用者最多可以有一個 SendGrid 帳戶。
    * 每個訂用帳戶最多有兩個 SendGrid 帳戶。

如果您已部署此解決方案的舊版，就必須先從帳戶中刪除它，才能部署此版本。  

## <a name="solution-components"></a>方案元件

此解決方案包括預先設定的 Runbook、排程，以及與 Log Analytics 的整合，可讓您針對業務需求量身打造虛擬機器的啟動和關機時間。 

### <a name="runbooks"></a>Runbook

下表列出部署至您自動化帳戶的 Runbook。  您不應變更 Runbook 的程式碼。 相反地，為新功能撰寫自己的 Runbook。

> [!IMPORTANT]
> 請勿直接執行任何有「子項目」附加至其名稱的 Runbook。
>

所有父代 Runbook 皆包含 *WhatIf* 參數。 將 *WhatIf* 設為 **True** 時，即可支援詳述在不使用 *WhatIf* 參數執行的情況下，該 Runbook 會採取的確切行為，並驗證是否已將正確的虛擬機器作為目標。  *WhatIf* 參數設為 **False** 時，Runbook 只會執行其定義的動作。 

|**Runbook** | **參數** | **說明**|
| --- | --- | ---| 
|AutoStop_CreateAlert_Child | VMObject <br> AlertAction <br> WebHookURI | 只能從父代 Runbook 呼叫。 針對 AutoStop 案例以每個資源為基礎建立警示。| 
|AutoStop_CreateAlert_Parent | WhatIf：True 或 False <br> VMList | 在目標訂用帳戶或資源群組中的 VM 上建立或更新 Azure 警示規則。 <br> VMList：以逗號分隔的虛擬機器清單。  例如，*vm1,vm2,vm3*。| 
|AutoStop_Disable | None | 停用 AutoStop 警示和預設排程。| 
|AutoStop_StopVM_Child | WebHookData | 只能從父代 Runbook 呼叫。 警示規則會呼叫此 Runbook 以停止虛擬機器。|  
|Bootstrap_Main | None | 單次使用以設定啟動程序設定 (例如 webhookURI)，這些設定通常無法從 Azure Resource Manager 存取。 在部署成功之後會自動移除此 Runbook。|  
|ScheduledStartStop_Child | VMName <br> Action：Stop 或 Start <br> resourceGroupName | 只能從父代 Runbook 呼叫。 針對排程的停止，執行停止或啟動。|  
|ScheduledStartStop_Parent | Action：Stop 或 Start <br> WhatIf：True 或 False | 這會影響訂用帳戶中的所有虛擬機器。 編輯 **External_Start_ResourceGroupNames** 和 **External_Stop_ResourceGroupNames**，以便只在這些目標資源群組上執行。 您也可以更新 **External_ExcludeVMNames** 變數來排除特定的 VM。 *WhatIf* 的行為與在其他 Runbook 中相同。|  
|SequencedStartStop_Parent | Action：Stop 或 Start <br> WhatIf：True 或 False | 在您要序列啟動/停止活動的每部虛擬機器上建立名為 **SequenceStart** 和 **SequenceStop** 的標記。 標記值應為正整數 (1, 2, 3)，對應至您要啟動或停止的順序。 *WhatIf* 的行為與在其他 Runbook 中相同。 <br> **注意**：虛擬機器必須位於定義於 Azure 自動化變數中 External_Start_ResourceGroupNames、External_Stop_ResourceGroupNames 和 External_ExcludeVMNames 的資源群組內。 虛擬機器必須具有適當的標記以使動作生效。|

### <a name="variables"></a>變數

下表列出在您自動化帳戶中建立的變數。  您只應修改前面加上 **External** 的變數。 修改前面加上 **Internal** 的變數會造成非預期的結果。  

|**Variable** | **說明**|
---------|------------|
|External_AutoStop_Condition | 設定觸發警示之條件所需的條件運算子。 可接受的值為 **GreaterThan**、**GreaterThanOrEqual**、**LessThan** 和 **LessThanOrEqual**。|  
|External_AutoStop_Description | 在 CPU 百分比超出閾值的情況下停止虛擬機器的警示。|  
|External_AutoStop_MetricName | 要設定 Azure 警示規則的效能計量名稱。| 
|External_AutoStop_Threshold | 適用於在變數 *External_AutoStop_MetricName* 中指定之 Azure 警示規則的閾值。 百分比值的範圍為 1 至 100。|  
|External_AutoStop_TimeAggregationOperator | 會套用至選取的視窗大小以評估條件的時間彙總運算子。 可接受的值為 **Average**、**Minimum**、**Maximum**、**Total** 和 **Last**。|  
|External_AutoStop_TimeWindow | Azure 分析選取之計量以觸發警示的視窗大小。 此參數接受時間範圍格式的輸入。 可能的值為 5 分鐘到 6 小時。|  
|External_EmailFromAddress | 指定電子郵件的寄件者。|  
|External_EmailSubject | 指定電子郵件的主旨列文字。|  
|External_EmailToAddress | 指定電子郵件的收件者。 使用逗號分隔名稱。|  
|External_ExcludeVMNames | 輸入要排除的虛擬機器名稱，請使用不含空格的逗號來分隔名稱。|  
|External_IsSendEmail | 指定完成時傳送電子郵件通知的選項。  針對是否傳送電子郵件，指定 **Yes** 或 **No**。  如果您沒有在初始部署期間啟用電子郵件通知，此選項應為 **No**。|  
|External_Start_ResourceGroupNames | 使用逗號分隔值指定一或多個作為啟動動作目標的資源群組。|  
|External_Stop_ResourceGroupNames | 使用逗號分隔值指定一或多個作為停止動作目標的資源群組。|  
|Internal_AutomationAccountName | 指定自動化帳戶的名稱。|  
|Internal_AutoSnooze_WebhookUri | 指定針對 AutoStop 案例呼叫的 Webhook URI。|  
|Internal_AzureSubscriptionId | 指定 Azure 訂用帳戶識別碼。|  
|Internal_ResourceGroupName | 指定自動化帳戶資源群組名稱。|  
|Internal_SendGridAccountName | 指定 SendGrid 帳戶名稱。|  
|Internal_SendGridPassword | 指定 SendGrid 帳戶密碼。|  

<br>

在所有情況下，**External_Start_ResourceGroupNames**、**External_Stop_ResourceGroupNames** 和 **External_ExcludeVMNames** 變數皆為設定目標虛擬機器的必要項目 (除了為 **AutoStop_CreateAlert_Parent** Runbook 提供以逗號分隔的虛擬機器清單以外)。 也就是說，您的虛擬機器必須位於目標資源群組中，才會發生啟動和停止動作。 此邏輯的運作方式有點類似 Azure 原則，其中您可以將訂用帳戶或資源群組設為目標，並讓新建立的虛擬機器繼承動作。 此方法可免去針對每部虛擬機器維護個別排程的必要，並可大規模地管理啟動和停止。

### <a name="schedules"></a>排程

下表列出在您的自動化帳戶中建立的各個預設排程。  您可以修改它們，或建立自己的自訂排程。  這些排程預設皆為停用，除了 **Scheduled_StartVM** 和 **Scheduled_StopVM** 之外。

您不應啟用所有排程，因為這樣可能會產生重疊的排程動作。 最好能先判斷要執行哪些最佳化，並據以做出相對應的修改。  如需進一步說明，請參閱＜概觀＞一節中的範例案例。   

|**排程名稱** | **頻率** | **說明**|
|--- | --- | ---|
|Schedule_AutoStop_CreateAlert_Parent | 每 8 小時 | 每隔 8 小時會執行 AutoStop_CreateAlert_Parent Runbook，這會停止在 Azure 自動化變數中 External_Start_ResourceGroupNames、External_Stop_ResourceGroupNames 和 External_ExcludeVMNames 中的虛擬機器基底值。  或者，您可以使用 VMList 參數指定以逗號分隔的虛擬機器清單。|  
|Scheduled_StopVM | 使用者定義，每日 | 每天會在指定時間搭配 *Stop* 參數執行 Scheduled_Parent Runbook。  會自動停止符合由資產變數所定義之規則的所有虛擬機器。 您應啟用相關排程 **Scheduled-StartVM**。|  
|Scheduled_StartVM | 使用者定義，每日 | 每天會在指定時間搭配 *Start* 參數執行 Scheduled_Parent Runbook。  會自動啟動符合由適當變數所定義之規則的所有虛擬機器。  您應啟用相關排程 **Scheduled-StopVM**。|
|Sequenced-StopVM | 上午 1:00 (UTC)，每星期五 | 每星期五會在指定時間搭配參數 *Stop* 執行 Sequenced_Parent Runbook。  會以循序方式 (遞增) 停止具有由適當變數定義之 **SequenceStop** 標記的所有虛擬機器。  如需標記值和資產變數的詳細資料，請參閱＜Runbook＞一節。  您應啟用相關排程 **Sequenced-StartVM**。|
|Sequenced-StartVM | 下午 1:00 (UTC)，每星期一 | 每星期一會在指定時間搭配參數 *Start* 執行 Sequenced_Parent Runbook。 會以循序方式 (遞減) 啟動具有由適當變數定義之 **SequenceStart** 標記的所有虛擬機器。  如需標記值和資產變數的詳細資料，請參閱＜Runbook＞一節。  您應啟用相關排程 **Sequenced-StopVM**。|

<br>

## <a name="configuration"></a>組態

執行下列步驟，將「停機期間啟動/停止 VM」解決方案新增至您的自動化帳戶，然後設定變數以自訂解決方案。

1. 在 Azure 入口網站中，按一下 [建立資源]。<br> ![Azure 入口網站](media/automation-solution-vm-management/azure-portal-01.png)<br>  
2. 在 [Marketplace] 窗格中輸入關鍵字，例如**啟動**或**啟動/停止**。 當您開始輸入時，清單會根據您輸入的文字進行篩選。 或者，您可以輸入解決方案完整名稱中的一或多個關鍵字，然後按 Enter 鍵。  從搜尋結果選取 [停機期間啟動/停止 VM [預覽]]。  
3. 在所選解決方案的 [停機期間啟動/停止 VM [預覽]] 窗格中，檢閱摘要資訊，然後按一下 [建立]。  
4. [新增解決方案] 窗格隨即出現。 系統會在其中提示您設定解決方案，以將它匯入您的自動化訂用帳戶。<br><br> ![VM 管理的新增方案刀鋒視窗](media/automation-solution-vm-management/azure-portal-add-solution-01.png)<br><br>
5.  在 [新增解決方案] 窗格中選取 [工作區]。 選取連結到自動化帳戶所在之同一 Azure 訂用帳戶的 OMS 工作區。 如果您沒有工作區，請選取 [建立新工作區]。 在 [OMS 工作區] 窗格中，執行下列動作： 
   - 指定新 [OMS 工作區] 的名稱。
   - 如果選取的預設值不合適，請從下拉式清單中選取要連結的 [訂用帳戶]。
   - 對於 [資源群組]，您可以建立新的資源群組，或選取現有的資源群組。  
   - 選取 [位置] 。  目前可用的位置只有**澳大利亞東南部**、**加拿大中部**、**印度中部**、**美國東部**、**日本東部**、**東南亞**、**英國南部**和**西歐**。
   - 選取 **定價層**。  此解決方案提供兩種定價層︰免費和 OMS 付費。  免費層會限制每日可收集的資料量、保留期和 Runbook 作業執行階段分鐘數。  OMS 付費層不會限制每日可收集的資料量。  

        > [!NOTE]
        > 儘管系統會顯示 [每 GB (獨立)] 付費層作為選項，但它並不適用。  如果您選取它並在訂用帳戶中繼續建立此解決方案，則會失敗。  當此解決方案正式發行時，將會解決這個問題。<br>此解決方案只會使用自動化作業分鐘數並記錄擷取。  解決方案不會將其他 OMS 節點新增至您的環境。  

6. 在 [OMS 工作區] 窗格上提供必要資訊之後，按一下 [建立]。  您可以在功能表的 [通知] 下追蹤其進度，這會在完成時帶您返回 [新增解決方案] 窗格。  
7. 在 [新增解決方案] 窗格中，選取 [自動化帳戶]。  如果要建立新的 OMS 工作區，您還需要建立新的自動化帳戶以與其建立關聯。  選取 [建立自動化帳戶]，然後在 [新增自動化帳戶] 窗格上提供下列資訊︰ 
  - 在 [名稱] 欄位中輸入自動化帳戶的名稱。

    系統會根據所選的 OMS 工作區自動填入所有其他選項。 這些選項無法修改。  Azure 執行身分帳戶是此方案內含 Runbook 的預設驗證方法。  按下 [確定] 後，就會驗證設定選項並建立自動化帳戶。  您可以在功能表的 [通知] 底下追蹤其進度。 

    不然，您可以選取現有的自動化執行身分帳戶。  請注意，您選取的帳戶不能已連結至另一個 OMS 工作區。 如果帳戶已經連結，您會收到訊息，必須選取不同的自動化執行身分帳戶或建立一個新帳戶。<br><br> ![自動化帳戶已經連結至 OMS 工作區](media/automation-solution-vm-management/vm-management-solution-add-solution-blade-autoacct-warning.png)<br>

8. 最後，在 [新增解決方案] 窗格中，選取 [設定]。 [參數] 窗格隨即出現。<br><br> ![解決方案的 [參數] 窗格](media/automation-solution-vm-management/azure-portal-add-solution-02.png)<br><br>  在這裡，系統會提示您：  
   - 指定 [目標資源群組名稱]。 這些是包含此解決方案所要管理的虛擬機器之資源群組名稱。  您可以輸入多個名稱，然後用逗號加以分隔 (值不區分大小寫)。  如果您想要以訂用帳戶的所有資源群組中的 VM 為目標，則可使用萬用字元。
   - 指定 [虛擬機器排除清單 (字串)]。 這是來自目標資源群組的一或多個虛擬機器名稱。  您可以輸入多個名稱，然後用逗號加以分隔 (值不區分大小寫)。  支援使用萬用字元。
   - 選取**排程**。 這是一個週期性日期和時間，可用於啟動及停止目標資源群組中的虛擬機器。  根據預設，排程會設定為 UTC 時區。 無法選取不同的區域。  在設定解決方案後，若要將排程設定為特定時區，請參閱[修改啟動和關機排程](#modifying-the-startup-and-shutdown-schedule)。
   - 若要從 SendGrid 接收**電子郵件通知**，請接受預設值 [是]，並提供有效的電子郵件地址。 如果您選取 [否]，但在日後決定想要收到電子郵件通知，您可以使用以逗號分隔的有效電子郵件地址更新 **External_EmailToAddress** 變數，然後將變數 **External_IsSendEmail** 的值修改為 [是]。  

9. 設定好解決方案所需的初始設定後，選取 [建立]。  驗證過所有設定之後，解決方案即會部署到您的訂用帳戶。  此程序需要幾秒鐘才能完成，您可以在功能表的 [通知] 底下追蹤其進度。 

## <a name="collection-frequency"></a>收集頻率

系統每隔 5 分鐘會將自動化作業記錄和作業串流資料擷取至 Log Analytics 存放庫。  

## <a name="using-the-solution"></a>使用解決方案

新增虛擬機器管理解決方案時，[StartStopVMView] 圖格會從 Azure 入口網站新增至 Log Analytics 工作區中的儀表板。  此圖格會顯示解決方案中已啟動並順利完成的 Runbook 作業計數和圖形表示。<br><br> ![VM 管理 StartStopVM 檢視圖格](media/automation-solution-vm-management/vm-management-solution-startstopvm-view-tile.png)  

在自動化帳戶中，您可以選取 [工作區] 選項來存取和管理解決方案。 在 [Log Analytics] 頁面上，選取左窗格中的 [解決方案]。  在 [解決方案] 頁面上，從清單中選取 [Start-Stop-VM[工作區]] 解決方案。<br><br> ![Log Analytics 中的 [解決方案] 清單](media/automation-solution-vm-management/azure-portal-select-solution-01.png)  

選取該解決方案會顯示 [Start-Stop-VM[工作區]] 解決方案窗格。 在這裡您可以檢閱重要的詳細資料，如 [StartStopVM] 圖格。 如同在 Log Analytics 工作區中，此圖格會顯示解決方案中已啟動並已順利完成的 Runbook 作業計數和圖形表示。<br><br> ![自動更新管理解決方案頁面](media/automation-solution-vm-management/azure-portal-vmupdate-solution-01.png)  

您可以從這裡按一下環圈圖格，執行進一步的作業記錄分析。 解決方案儀表板會顯示作業記錄和預先定義的記錄搜尋查詢。 切換到 Log Analytics 進階入口網站，根據您的搜尋查詢進行搜尋。  

所有父代 Runbook 皆包含 *WhatIf* 參數。 將 WhatIf 設為 **True** 時，即可支援詳述在不使用 *WhatIf* 參數執行的情況下，該 Runbook 會採取的確切行為，並驗證是否已將正確的虛擬機器作為目標。 *WhatIf* 參數設為 **False** 時，Runbook 只會執行其定義的動作。  


### <a name="scenario-1-daily-startstop-vms-across-a-subscription-or-target-resource-groups"></a>案例 1：每天啟動/停止訂用帳戶或目標資源群組上的虛擬機器 

這是您首次部署解決方案時的預設設定。  例如，您可以將它設定為在您晚上離開公司時停止訂用帳戶上的所有虛擬機器，並在您早上回到辦公室時啟動它們。 當您在部署期間設定排程 **Scheduled-StartVM** 和 **Scheduled-StopVM** 時，它們會啟動和停止目標虛擬機器。
>[!NOTE]
>時區是您設定排程表時間參數時所在的目前時區。 但它會以 UTC 格式儲存在 Azure 自動化中。  您不需要執行任何時區轉換，因為它會在部署期間處理。

您可藉由設定下列變數：**External_Start_ResourceGroupNames**、**External_Stop_ResourceGroupNames** 和 **External_ExcludeVMNames** 來控制有哪些虛擬機器包含於範圍中。  

>[!NOTE]
> **Target ResourceGroup Names** 的值會儲存為 **External_Start_ResourceGroupNames** 和 **External_Stop_ResourceGroupNames** 的值。 如需進一步的細微設定，您可以修改這些變數來以不同的資源群組為目標。  針對啟動動作請使用 **External_Start_ResourceGroupNames**，而針對停止動作請使用 **External_Stop_ResourceGroupNames**。 虛擬機器會自動新增至啟動和停止排程。

若要測試及驗證設定，請手動啟動 **ScheduledStartStop_Parent** Runbook，並將 ACTION 參數設為 **start** 或 **stop**，然後將 WHATIF 參數設為 **True**。<br><br> ![設定 Runbook 的參數](media/automation-solution-vm-management/solution-startrunbook-parameters-01.png)


 預覽已排程的動作，並在針對生產虛擬機器實作前進行所有必要的變更。  您可以將參數設定為 **False** 並手動執行 Runbook，或是讓自動化排程 **Schedule-StartVM** 和 **Schedule-StopVM** 依照您指定的排程自動執行。

### <a name="scenario-2-sequence-the-startstop-vms-across-a-subscription-by-using-tags"></a>案例 2：使用標記循序啟動/停止訂用帳戶上的虛擬機器
在多部虛擬機器上包含兩個或多個元件並支援分散式工作負載的環境中，支援元件的啟動和停止順序是非常重要的。  您可以執行下列步驟來達成此目的：

1. 針對於 **External_Start_ResourceGroupNames** 和 **External_Stop_ResourceGroupNames** 變數中設為目標的虛擬機器，新增具有正整數值的 **SequenceStart** 和 **SequenceStop** 標記。  啟動和停止動作會依遞增順序執行。  若要了解如何標記虛擬機器，請參閱[在 Azure 中標記 Windows 虛擬機器](../virtual-machines/windows/tag.md)和[在 Azure 中標記 Linux 虛擬機器](../virtual-machines/linux/tag.md)。
2. 修改排程 **Sequenced-StartVM** 和 **Sequenced-StopVM** 以符合您所需的日期和時間，然後啟用排程。  

若要測試並驗證您的設定，請手動啟動 **SequencedStartStop_Parent** Runbook。 將 ACTION 參數設定為 **start** 或 **stop** 並將 WHATIF 參數設定為 **True**。<br><br> ![設定 Runbook 的參數](media/automation-solution-vm-management/solution-startrunbook-parameters-01.png)<br> 預覽動作，並在針對生產虛擬機器實作前進行所有必要的變更。  就緒後即可將參數設定為 **False** 並手動執行 Runbook，或是讓自動化排程 **Sequenced-StartVM** 和 **Sequenced-StopVM** 依照您指定的排程自動執行。  

### <a name="scenario-3-automate-startstop-vms-across-a-subscription-based-on-cpu-utilization"></a>案例 3：根據 CPU 使用率自動化啟動/停止訂用帳戶上的虛擬機器
此解決方案可評估非尖峰期間 (例如下班時間) 未使用的 Azure 虛擬機器，並在處理器使用率低於 x% 時自動將它們關閉，藉以協助管理訂用帳戶中虛擬機器執行的成本。  

根據預設，解決方案已預先設定會對百分比 CPU 計量進行評估，以檢查平均使用率是否為 5% 或更低。  這是由下列變數進行控制，並可在預設值不符合需求時加以修改：

* External_AutoStop_MetricName
* External_AutoStop_Threshold
* External_AutoStop_TimeAggregationOperator
* External_AutoStop_TimeWindow

您可以將動作的目標設為某個訂用帳戶和資源群組，或設為特定的虛擬機器清單，但不可同時設定為兩者。  

#### <a name="target-the-stop-action-against-a-subscription-and-resource-group"></a>針對訂用帳戶和資源群組設定停止動作目標

1. 設定 **External_Stop_ResourceGroupNames** 和 **External_ExcludeVMNames** 變數來指定目標 VM。  
2. 啟用並更新 **Schedule_AutoStop_CreateAlert_Parent** 排程。
3. 執行 **AutoStop_CreateAlert_Parent** Runbook，並將 ACTION 參數設為 **start**，然後將 WHATIF 參數設為 **True** 以預覽變更。

#### <a name="target-the-stop-action-by-vm-list"></a>透過虛擬機器清單設定停止動作目標

1. 執行 **AutoStop_CreateAlert_Parent** Runbook，並將 ACTION 參數設為 **start**，在 *VMList* 參數中新增以逗號分隔的虛擬機器清單，然後將 WHATIF 參數設為 **True**。 預覽變更。  
2. 使用以逗號分隔的虛擬機器清單 (VM1,VM2,VM3) 設定 **External_ExcludeVMNames** 參數。
3. 此案例不會接受 **External_Start_ResourceGroupNames** 和 **External_Stop_ResourceGroupnames** 變數。  針對此案例，您需要建立自己的自動化排程。 如需詳細資訊，請參閱[在 Azure 自動化中排程 Runbook](../automation/automation-schedules.md)。

您已具有會根據 CPU 使用率停止虛擬機器的排程，現在您需要啟用下列其中一個排程來啟動它們。

* 透過訂用帳戶和資源群組設定啟動動作目標。  請參閱[案例 1](#scenario-1:-daily-stop/start-vms-across-a-subscription-or-target-resource-groups) 中的步驟以測試和啟用 **Scheduled-StartVM** 排程。
* 透過訂用帳戶、資源群組和標記設定啟動動作目標。  請參閱[案例 2](#scenario-2:-sequence-the-stop/start-vms-across-a-subscription-using-tags) 中的步驟以測試和啟用 **Sequenced-StartVM** 排程。


### <a name="configuring-email-notifications"></a>設定電子郵件通知

若要在部署解決方案後設定電子郵件通知，請修改下列三個變數：

* External_EmailFromAddress：指定寄件人的電子郵件地址。
* External_EmailToAddress：指定以逗號分隔的電子郵件地址清單 (user@hotmail.com, user@outlook.com)，以接收通知電子郵件。
* External_IsSendEmail：設為 [是] 以接收電子郵件。 若要停用電子郵件通知，請將值設定為 [否]。   


### <a name="modifying-the-startup-and-shutdown-schedules"></a>修改啟動和關機排程

在此解決方案中管理啟動和關機排程的步驟，與[在 Azure 自動化中排程 Runbook](automation-schedules.md) 中所概述的步驟相同。     

## <a name="log-analytics-records"></a>Log Analytics 記錄

自動化會在 OMS 存放庫中建立兩種類型的記錄：作業記錄和作業串流。

### <a name="job-logs"></a>作業記錄檔

屬性 | 說明|
----------|----------|
呼叫者 |  起始作業的人員。  可能的值為電子郵件地址或排程作業的系統。|
類別 | 資料類型的分類。  對自動化來說，該值是 JobLogs。|
CorrelationId | Runbook 作業之相互關聯識別碼的 GUID。|
JobId | Runbook 作業之識別碼的 GUID。|
operationName | 指定在 Azure 中執行的作業類型。  對自動化來說，該值是 Job。|
ResourceId | 指定 Azure 中的資源類型。  對自動化來說，該值是與 Runbook 相關聯的自動化帳戶。|
ResourceGroup | 指定 Runbook 作業的資源群組名稱。|
ResourceProvider | 指定 Azure 服務，以提供您可部署及管理的資源。  對自動化來說，此值是 Azure 自動化。|
ResourceType | 指定 Azure 中的資源類型。  對自動化來說，該值是與 Runbook 相關聯的自動化帳戶。|
resultType | Runbook 作業的狀態。  可能的值包括：<br>- Started (已啟動)<br>- Stopped (已停止)<br>- Suspended (暫止)<br>- Failed (失敗)<br>- Succeeded|
resultDescription | 說明 Runbook 作業的結果狀態。  可能的值包括：<br>- Job is started (工作已啟動)<br>- Job Failed (工作失敗)<br>- Job Completed|
RunbookName | 指定 Runbook 的名稱。|
SourceSystem | 指定所提交資料的來源系統。  對自動化來說，該值是 OpsManager|
StreamType | 指定事件的類型。 可能的值包括：<br>- Verbose<br>- Output<br>- Error<br>- Warning|
SubscriptionId | 指定作業的訂用帳戶 ID。
時間 | Runbook 作業的執行日期和時間。|


### <a name="job-streams"></a>作業串流

屬性 | 說明|
----------|----------|
呼叫者 |  起始作業的人員。  可能的值為電子郵件地址或排程作業的系統。|
類別 | 資料類型的分類。  對自動化來說，該值是 JobStreams。|
JobId | Runbook 作業之識別碼的 GUID。|
operationName | 指定在 Azure 中執行的作業類型。  對自動化來說，該值是 Job。|
ResourceGroup | 指定 Runbook 作業的資源群組名稱。|
ResourceId | 指定 Azure 中的資源識別碼。  對自動化來說，該值是與 Runbook 相關聯的自動化帳戶。|
ResourceProvider | 指定 Azure 服務，以提供您可部署及管理的資源。  對自動化來說，此值是 Azure 自動化。|
ResourceType | 指定 Azure 中的資源類型。  對自動化來說，該值是與 Runbook 相關聯的自動化帳戶。|
resultType | Runbook 作業在產生事件時的結果。 可能的值為：<br>- InProgres|
resultDescription | 包含來自 Runbook 的輸出串流。|
RunbookName | Runbook 的名稱。|
SourceSystem | 指定所提交資料的來源系統。  對自動化來說，該值是 OpsManager。|
StreamType | 作業串流的類型。 可能的值包括：<br>- Progress (進度)<br>- Output (輸出)<br>- Warning (警告)<br>- Error (錯誤)<br>- Debug (偵錯)<br>- Verbose|
時間 | Runbook 作業的執行日期和時間。|

當您執行的記錄搜尋傳回 **JobLogs** 或 **JobStreams** 的類別記錄時，您可以選取 **JobLogs** 或 **JobStreams** 檢視，其中會顯示一組彙總搜尋所傳回更新的圖格。

## <a name="sample-log-searches"></a>記錄搜尋範例

下表提供此方案所收集之作業記錄的記錄檔搜尋範例。 

查詢 | 說明|
----------|----------|
尋找 ScheduledStartStop_Parent Runbook 已順利完成的作業 | search Category == "JobLogs" &#124; where ( RunbookName_s == "ScheduledStartStop_Parent" ) &#124; where ( ResultType == "Completed" )  &#124; summarize AggregatedValue = count() by ResultType, bin(TimeGenerated, 1h) &#124; sort by TimeGenerated desc|
尋找 SequencedStartStop_Parent Runbook 已順利完成的作業 | search Category == "JobLogs" &#124; where ( RunbookName_s == "SequencedStartStop_Parent" ) &#124; where ( ResultType == "Completed" )  &#124; summarize AggregatedValue = count() by ResultType, bin(TimeGenerated, 1h) &#124; sort by TimeGenerated desc

## <a name="removing-the-solution"></a>移除解決方案

如果您決定不再需要使用解決方案，您可以將它從自動化帳戶中刪除。  刪除解決方案只會移除 Runbook。 不會刪除新增解決方案時所建立的排程或變數。  如果您不搭配其他 Runbook 使用這些資產，則必須以手動方式加以刪除。  

若要刪除解決方案，請執行下列步驟：

1.  從您的自動化帳戶，選取左窗格中的 [工作區]。  
2.  在 [解決方案] 頁面上，選取 [Start-Stop-VM[工作區]] 解決方案。  在 [VMManagementSolution[工作區]] 頁面上，選取功能表中的 [刪除]。<br><br> ![刪除 VM 管理解決方案](media/automation-solution-vm-management/vm-management-solution-delete.png)
3.  在 [刪除解決方案] 視窗中，確認您要刪除解決方案。
4.  在驗證資訊並刪除解決方案後，您可以在功能表的 [通知] 底下追蹤其進度。  移除解決方案的程序啟動之後，您會返回 [解決方案] 頁面。  

此程序並不會刪除自動化帳戶和 Log Analytics 工作區。  如果不想保留 Log Analytics 工作區，則必須手動刪除它。  此作業可以從 Azure 入口網站完成：

1.    在 Azure 入口網站主畫面中，選取 [Log Analytics]。
2. 在 [Log Analytics] 窗格中，選取工作區。
3. 在工作區 [設定] 窗格的功能表中選取 [刪除]。  
      
## <a name="next-steps"></a>後續步驟

- 若要深入了解如何使用 Log Analytics 來建構不同的搜尋查詢及 檢閱「自動化」作業記錄，請參閱 [Log Analytics 中的記錄搜尋](../log-analytics/log-analytics-log-searches.md)。
- 若要深入了解 Runbook 執行方式、如何監視 Runbook 作業，以及其他技術性詳細資料，請參閱[追蹤 Runbook 作業](automation-runbook-execution.md)。
- 若要深入了解 Log Analytics 和資料收集來源，請參閱[在 Log Analytics 中收集 Azure 儲存體資料概觀](../log-analytics/log-analytics-azure-storage.md)。






   

