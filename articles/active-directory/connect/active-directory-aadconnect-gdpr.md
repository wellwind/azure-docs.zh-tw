---
title: "Azure AD Connect 和一般資料保護規定 | Microsoft Docs"
description: "本文件說明如何使用 Azure AD Connect 取得 GDPR 合規性。"
services: active-directory
documentationcenter: 
author: billmath
manager: mtillman
editor: 
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/15/2018
ms.author: billmath
ms.openlocfilehash: c3956dd379961b119f65bdebe1f5a8038c4fa8f0
ms.sourcegitcommit: 088a8788d69a63a8e1333ad272d4a299cb19316e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/27/2018
---
# <a name="gdpr-compliance-and-azure-ad-connect"></a>GDPR 合規性與 Azure AD Connect 

歐洲有一條隱私權法律 ([一般資料保護規定 (GDPR)](http://ec.europa.eu/justice/data-protection/reform/index_en.htm)) 將在 2018 年 5 月生效。 GDPR 會針對想要提供商品和服務給歐盟 (EU) 居民，或想要收集和分析 EU 居民相關資料的公司、政府機關、非營利組織和其他組織，施加新的規則。 無論您位於何處都會適用 GDPR。 

Microsoft 產品和服務目前均可幫助您符合 GDPR 需求。 在[信任中心](https://www.microsoft.com/trustcenter)深入了解 Microsoft 隱私權原則

>[!NOTE] 
>本文會討論 Azure AD Connect 和 GDPR 合規性。  如需 Azure AD Connect Health 和 GDPR 合規性的相關資訊，請參閱[這裡](../../active-directory/connect-health/active-directory-aadconnect-health-gdpr.md)的文章。

Azure AD Connect 安裝的一般資料保護規定合規性可以透過兩種方式來達成：

1.  在接到要求時擷取使用者的資料，並從安裝中移除該使用者的資料
2.  確保沒有資料會保留超過 48 小時。

Azure AD Connect 小組建議採用第二個選項，因為此選項較容易實作及維護。

Azure AD Connect 同步伺服器會儲存下列符合 GDPR 合規性適用範圍的資料：
1.  **Azure AD Connect 資料庫**中的個人相關資料
2.  **Windows 事件記錄**檔案中可能包含個人相關資訊的資料
3.  **Azure AD Connect 安裝記錄檔**中可能包含的個人相關資料

若要符合 GDPR，Azure AD Connect 客戶應該使用下列指導方針：
1.  定期刪除 Azure AD Connect 安裝記錄檔所在資料夾中的內容 – 至少每 48 小時一次
2.  此產品也可能會建立事件記錄檔。  若要深入了解事件記錄檔的記錄，請參閱[這裡的文件](https://msdn.microsoft.com/library/windows/desktop/aa385780.aspx)。

當個人資料從其原始來源系統中移除時，該個人資料也會從 Azure AD Connect 資料庫自動移除。 系統管理員不必採取特定行動即可符合 GDPR。  不過，Azure AD Connect 資料則必須至少每兩天就與資料來源同步處理一次。

## <a name="delete-the-azure-ad-connect-installation-log-file-folder-contents"></a>刪除 Azure AD Connect 安裝記錄檔資料夾內容
請定期檢查並刪除 **c:\programdata\aadconnect** 資料夾的內容 – **PersistedState.Xml** 檔案除外。 這個檔案保有先前之 Azure AD Connect 安裝的狀態，而且會在執行升級安裝時用到。 此檔案未包含任何個人相關資料，所以不應刪除。

>[!IMPORTANT]
>請勿刪除 PersistedState.xml 檔案。  此檔案未包含任何使用者資訊，且會保有先前安裝的狀態。

您可以使用 Windows 檔案總管檢閱和刪除這些檔案，也可以使用如下所示的指令碼來執行必要的動作：


```
$Files = ((Get-childitem -Path "$env:programdata\aadconnect" -Recurse).VersionInfo).FileName
Foreach ($file in $files) {
If ($File.ToUpper() -ne "$env:programdata\aadconnect\PERSISTEDSTATE.XML".toupper()) # Do not delete this file
    {Remove-Item -Path $File -Force}
    } 
```

### <a name="schedule-this-script-to-run-every-48-hours"></a>將此指令碼排程為每 48 小時執行一次
請使用下列步驟來將指令碼排程為每 48 小時執行一次。

1.  將指令碼儲存在副檔名為 **&#46;PS1** 的檔案中，然後開啟控制台並按一下 [系統及安全性]。
    ![系統](media\active-directory-aadconnect-gdpr\gdpr2.png)

2.  在 [系統管理工具] 標題下，按一下 [排程工作]。
    ![Task](media\active-directory-aadconnect-gdpr\gdpr3.png)
3.  在工作排程器中，以滑鼠右鍵按一下 [工作排程程式庫]，然後按一下 [建立基本工作...]
4.  輸入新工作的名稱，然後按 [下一步]。
5.  選取 [每天] 作為工作觸發程序，然後按 [下一步]。
6.  將週期設定為 **2 天**，然後按 [下一步]。
7.  選取 [啟動程式] 作為動作，然後按 [下一步]。
8.  在程式/指令碼的方塊中輸入 **PowerShell**，然後在標示為 [新增引數 (可省略)] 的方塊中，輸入您稍早所建立之指令碼的完整路徑，然後按 [下一步]。
9.  下一個畫面會顯示您即將建立之工作的摘要。 確認所有值，然後按一下 [完成] 來建立工作。



## <a name="next-steps"></a>後續步驟
- [整合內部部署身分識別與 Azure Active Directory](active-directory-aadconnect.md)。
- [Azure AD Connect Health 和 GDPR](../../active-directory/connect-health/active-directory-aadconnect-health-gdpr.md)
