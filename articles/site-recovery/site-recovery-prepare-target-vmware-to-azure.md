---
title: "準備目標 (VMware 至 Azure) | Microsoft Docs"
description: "本文將說明如何準備您的 Azure 環境，以開始將 VMware 虛擬機器複寫至 Azure。"
services: site-recovery
documentationcenter: 
author: bsiva
manager: abhemraj
editor: 
ms.assetid: 
ms.service: site-recovery
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: backup-recovery
ms.date: 02/22/2018
ms.author: bsiva
ms.openlocfilehash: ec1322e95431d5fd38d8e05a66322239d3184ac0
ms.sourcegitcommit: fbba5027fa76674b64294f47baef85b669de04b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/24/2018
---
# <a name="prepare-target-vmware-to-azure"></a>準備目標 (VMware 至 Azure)
> [!div class="op_single_selector"]
> * [VMware 至 Azure](./site-recovery-prepare-target-vmware-to-azure.md)
> * [實體至 Azure](./site-recovery-prepare-target-physical-to-azure.md)

本文將說明如何準備您的 Azure 環境，以開始將 VMware 虛擬機器複寫至 Azure。

## <a name="prerequisites"></a>先決條件

本文假設：
- 您已建立復原服務保存庫，用以保護 VMware 虛擬機器。 您可以從 [Azure 入口網站] (http://portal.azure.com "Azure 入口網站")建立復原服務保存庫。
- 您已[設定內部部署環境](./site-recovery-set-up-vmware-to-azure.md)以將 VMware 虛擬機器複寫至 Azure。

## <a name="prepare-target"></a>準備目標

完成**步驟 1：選取保護目標**和**步驟 2︰準備來源**之後，即會進入**步驟 3︰目標**

![準備目標](./media/site-recovery-prepare-target-vmware-to-azure/prepare-target-vmware-to-azure.png)

1. **訂用帳戶︰**從下拉式功能表中選取您想要的訂用帳戶，以將虛擬機器複寫至該訂用帳戶。
2. **部署模型︰**選取部署模型 (傳統或資源管理員)

根據所選的部署模型，會執行驗證以確保您的目標訂用帳戶中有至少一個相容的儲存體帳戶和虛擬網路，以便將您的虛擬機器複寫和容錯移轉至其中。

驗證成功完成後，按一下 [確定] 以移至下一個步驟。

如果您沒有相容的資源管理員儲存體帳戶或虛擬網路，您可以按一下頁面頂端的 [+ 儲存體帳戶] 或 [+ 網路] 按鈕來建立一個。

## <a name="next-steps"></a>後續步驟
[設定複寫設定](./site-recovery-setup-replication-settings-vmware.md)。
