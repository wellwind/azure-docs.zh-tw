---
title: "Azure SQL Database Azure 案例研究 - Snelstart| Microsoft Docs"
description: "了解 SnelStart 如何使用 SQL Database 以每月 1,000 個新 Azure SQL Database 的速度快速擴充其商務服務"
services: sql-database
documentationcenter: 
author: CarlRabeler
manager: jhubbard
editor: 
ms.assetid: fab506b2-439d-4f1a-bdc5-d1d25c80d267
ms.service: sql-database
ms.custom: reference
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: Inactive
ms.date: 01/10/2017
ms.author: carlrab
ms.openlocfilehash: a5100941675b6bdd1934bfcabe10172b4949fff9
ms.sourcegitcommit: 059dae3d8a0e716adc95ad2296843a45745a415d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/09/2018
---
# <a name="with-azure-snelstart-has-rapidly-expanded-its-business-services-at-a-rate-of-1000-new-azure-sql-databases-per-month"></a>透過 Azure，SnelStart 以每月 1,000 個新 Azure SQL Database 的速度快速擴充其商務服務
![SnelStartLogo](./media/sql-database-implementation-snelstart/snelstartlogo.png)

SnelStart 為荷蘭的中小企業 (SMB) 撰寫常用的財務和商務管理軟體。 它的 55,000 個客戶是由其 110 個員工提供服務，其中包括 35 個 IT 人員。 藉由從桌面軟體移至 Azure 上的軟體即服務 (SaaS) 方案組建，SnelStart 不僅透過使用 C# 中的熟悉環境將管理自動化，還透過使用彈性集區避免過度佈建或佈建不足以將效能和延展性最佳化，來充分利用了內建服務。 使用 Azure 讓 SnelStart 能夠在內部部署環境與雲端之間靈活移動客戶。

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Azure-SQL-Database-Case-Study-SnelStart/player]
> 
> 

## <a name="why-snelstart-extended-services-from-the-desktop-to-the-cloud"></a>為什麼 SnelStart 將服務從桌面延伸到雲端
> 「與 Azure 合作意謂著我們可以縮短提供軟體的時間、快速回應客戶需求，以及在需求的增加時調整解決方案。」
> 
> — Henry Been，軟體架構設計師
> 
> 

SnelStart 已使用下列傳統開發模型成功經營軟體業務多年：撰寫程式碼、封裝、運送，然後又重複。 隨著時間的進展，變更的腳步越來越快。 法規經常變更，而客戶需要更簡單的方式來處理財務記錄，並與其會計師和政府合作來跟上這些變更。

「將軟體運送給客戶成本高且限制多」軟體架構設計師 Henry Been 表示。 「生產、封裝及運送成本限制了我們發行軟體的頻率。 我們必須封裝更新以供定期運送，但這樣很難滿足我們客戶的多變需求。 我們永遠無法確保客戶是否已升級至最新的產品版本。 因此，我們必須支援多個版本，而這讓我們很難做好支援工作。」

Been 補充道：「我們想要一個能夠以加速的腳步安排並發行更新的方法，讓我們能夠以更快的速度創新並為客戶建立新服務。 我們也想要一個能夠將更多程序自動化的方法，以簡化客戶的業務管理需求。」

對 SnelStart 來說，解決方案就是透過成為雲端型 SaaS 提供者來延伸其服務。 Azure SQL Database 平台已協助 SnelStart 達成該目標，而沒有引起基礎結構即服務 (IaaS) 所會導致的主要 IT 額外負荷。

雲端模型也讓 SnelStart 能夠快速修正錯誤及提供新功能，而不需要客戶下載及升級軟體。 Beeen 表示：「使用 Azure 雲端服務讓我們能夠快速因應來自協力廠商的多變需求。 我們可以不需將新版本運送給數千個客戶，而是將傳送自傳統型應用程式的資訊改成協力廠商所要求的新格式。」

## <a name="a-modern-company-with-traditional-roots"></a>具有傳統根源的現代化公司
SnelStart 是一個現代化、靈活的高科技企業，其根源可追溯至 1964 年，當時創辦人是將這家公司創辦為樂器零件製造商。 SnelStart 軟體事業的核心實際上是在 1980 年代開始啟動，也就是個人電腦開始普及的期間。 該公司需要一個能夠替代當時可用會計軟體的更好方案，因此它將這些任務攬在了自己的手中。 在 1982 年，該公司建立了最終會成為 SnelStart 會計軟體的基礎。 一開始，該軟體因其簡易性和處理速度而受到誇讚，由於大受讚揚，因此該公司最後改變了重心而將自己重新創造為軟體公司。

在 1995 年，SnelStart 發行了它第一款適用於 Windows 的簿記應用程式。 該應用程式是在 Microsoft Visual Basic 1.0 和 Microsoft Access 資料庫上建置的應用程式，可協助將客戶群擴大到超過 5,000 個使用者。

現今，SnelStart 是主要的企業營運 (LOB) 及業務管理應用程式提供者，目標對象是荷蘭的中小企業和自行開業的企業家。 如 IT 架構設計師 Carlo Kuip 所述：「我們的目標是要為客戶提供 100% 的業務管理服務自動化。」

## <a name="optimizing-performance-and-cost-with-elastic-pools"></a>透過彈性集區將效能和成本最佳化
SnelStart 是彈性集區的早期大規模採用者。 彈性集區協助該公司限制成本，並更有效率地管理效能需求。 Been 表示：「藉由使用彈性集區，我們就可以根據客戶的需求將效能最佳化，而不會發生過度佈建的情況。 如果我們必須根據尖峰負載進行佈建，費用將會相當高昂。 而在多個低使用量資料庫之間共用資源的選項，則是讓我們能夠建立既執行順暢又符合成本效益的解決方案。」

## <a name="azure-sql-databases-help-containerize-data-for-isolation-and-security"></a>Azure SQL Database 協助將資料容器化來達到隔離及安全性目的
Azure SQL Database 讓 SnelStart 能夠以輕鬆且通透的方式將客戶的內部部署業務管理資料移至 Azure。 Azure SQL Database 是一個便利的容器，不僅提供隔離、驗證和授權的界限，也提供輕鬆的備份與還原功能。 資料庫為業務管理提供了一個適當的概念模型。 IT 架構設計師 Carlo Kuip 表示：「這個容器界限內的項目包含對企業至關重要的機密資料，將這些項目儲存在隔離的資料庫中可為它們提供嚴密的防護。 我們可以管理在資料庫層級的授權，甚至將管理自動化並從這些資料庫向外延展，而不需有編制內的資料庫管理員 (DBA) 來處理。」

「Azure SQL 資料倉儲」在 SnelStart 安全性與管理案例中協助公司收集遙測資料 (例如入侵偵測、使用者活動記錄，以及連線)，因此也扮演相當重要的角色。

## <a name="azure-removes-overhead-so-that-developers-can-spend-more-time-delivering-value"></a>Azure 免除了額外負荷，讓開發人員可以將更多時間專注在實現價值上
Azure 平台模型免除了基礎結構額外負荷，並讓 SnelStart 能夠使用 C# 管理程式庫將部署自動化。 如 Kuip 所述：「我們能夠以很少的人員來拓展目前的作業，同時又能為用戶端增加延展性、速度及災害復原選項。 轉換到服務開發釋出了資源來專注於新的服務和功能，而不只是更新現有的程式碼來跟上新的法規或稅捐代碼。 」 他補充道：「藉由將管理自動化及使用 SaaS 方案，我們得以為用戶端提供更多價值，而不需在作業人員上進行大量投資。 舉例來說，藉由使用 Azure 和彈性集區，SnelStart 得以新增各種新功能，包括與銀行的更健全客戶資料整合、新的帳務服務、小型企業背景檢查，以及電子郵件服務。

> 「僅僅是 2016 年的頭幾個月，我們便將 Azure SQL Database 部署從 5,500 個擴充到超過 12,000 個，而我們目前正以每月 1,000 個資料庫的速度持續新增。」
> 
> — Henry Been，軟體架構設計師
> 
> 

若要將資料庫管理近一步自動化，可使用彈性工作功能。 如 Kuip 所述：「我們非常感謝 SQL DB [伺服器] 執行個體上的自動資料庫探索功能。 」 這可讓 SnelStart 在其動態成長的客戶群上執行管理作業，而不會產生額外負荷。

SnelStart 也正在開發 API，來作為客戶資料與協力廠商軟體合作夥伴所建置的應用程式之間的代理程式。 如 Kuip 所述：「這個 API 將可讓其他廠商將功能新增到我們的軟體中，例如消除發票和其他文件的資料輸入。 」 客戶將不再需要手動輸入他們的小型企業會計軟體；SnelStart 軟體將會直接交換必要的資訊。 這可讓客戶將其業務管理工作與從業界中數位轉換產生的資訊生態系統結合。  

## <a name="how-azure-services-enable-saas-for-snelstart"></a>Azure 服務如何啟用適用於 SnelStart 的 SaaS
藉由使用 Azure，SnelStart 可以在雲端以更順暢的方式為其客戶及會計師提供服務。 舉例來說，客戶和會計師都可以直接存取裝載在 Azure 上的 SnelStart 用戶端 API 來共用資訊。 Kuip 說道：「透過我們的客戶面向 Web 應用程式即可使用這些可重複使用的服務，而這些服務提供通用的基礎結構和功能來允許存取客戶的業務管理，以及存取客戶所使用的協力廠商軟體。 範例包括提供產品組態功能、管理防火牆規則，以及管理長時間執行的程序，例如備份。」

> 「我們的目標是要為客戶提供 100% 的業務管理服務自動化。」 
> 
> — Carlo Kuip，IT 架構設計師
> 
> 

此外，SnelStart Web 服務也讓客戶和會計師都能夠輕鬆存取 Azure SQL Database 彈性集區中的資料。 這個 SaaS 模型再結合資料庫彈性與 Azure Resource Manager，可為 SnelStart 提供能夠補足每個 Azure 部署的延展性功能。 此實作是使用 C# 管理程式庫來完全自動化執行。

![SnelStart 架構](./media/sql-database-implementation-snelstart/figure1.png)

圖 1. 截至 2016 年 6 月為止，SnelStart 擁有超過 11,000 個資料庫，以及超過 50 個彈性集區

## <a name="simplicity-from-the-cloud"></a>來自雲端的簡單性
自從移至 Azure 雲端型解決方案之後，SnelStart 便得以支援快速的客戶成長，同時又能夠提供創新的功能和服務。 Been 表示：「透過 Azure，我們可以為客戶提供幾乎持續不斷的更新，而不需擴充我們的作業人員編制。 而且我們還附帶地獲得所有其他絕佳的 Azure 功能，像是延展性和災害復原。」

> 「當我們實際上遠在雷德蒙德時... 我接到了在荷蘭的開發人員來電，告訴我一個特定的問題。 我們得以讓 Microsoft 於 48 小時內在其生產環境中提供變更來解決我們的問題。」
> 
> — Henry Been，軟體架構設計師
> 
> 

SnelStart 也相當感謝他們已與 Microsoft Azure SQL DB 團隊發展出的強烈合作關係。 如 Kuip 所述：「我們針對功能及如何使用技術進行討論，雙方對此都心懷感謝。」
SnelStart 當前的目標是繼續拓展其滿意的客戶群。 如 Been 所述：「沒有了過去作為 ISV 的技術與資源限制之後，我們的成長便沒有限制。」

## <a name="more-information"></a>詳細資訊
* 若要深入了解 Azure 彈性集區，請參閱[彈性集區](sql-database-elastic-pool.md)。
* 若要深入了解「Azure SQL 資料倉儲」，請參閱 [SQL 資料倉儲](https://azure.microsoft.com/documentation/services/sql-data-warehouse/)
* 若要深入了解 SnelStart，請參閱 [SnelStart](http://www.snelstart.nl)。

