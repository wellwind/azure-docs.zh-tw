---
title: "使用 Azure 匯入/匯出將資料傳入和傳出 Azure 儲存體 | Microsoft Docs"
description: "了解如何在 Azure 入口網站中建立匯入和匯出作業，以將資料傳入和傳出 Azure 儲存體。"
author: muralikk
manager: syadav
editor: tysonn
services: storage
documentationcenter: 
ms.assetid: 668f53f2-f5a4-48b5-9369-88ec5ea05eb5
ms.service: storage
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/03/2017
ms.author: muralikk
ms.openlocfilehash: 0c34b7ce028ef0fae77322513f62557fa9f9929c
ms.sourcegitcommit: d87b039e13a5f8df1ee9d82a727e6bc04715c341
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/21/2018
---
# <a name="use-the-microsoft-azure-importexport-service-to-transfer-data-to-azure-storage"></a>使用 Microsoft Azure 匯入/匯出服務將資料傳入 Azure 儲存體
在本文中，我們會提供使用 Azure 匯入/匯出服務的逐步指示，藉由將磁碟機運送到 Azure 資料中心，安全地將大量資料傳入 Azure Blob 儲存體和 Azure 檔案服務。 這項服務也能用來將資料從 Azure 儲存體傳輸到硬碟，然後運送到您的內部部署網站。 單一內部 SATA 磁碟機的資料可匯入到 Azure Blob 儲存體或 Azure 檔案服務。 

> [!IMPORTANT] 
> 這項服務僅接受內部 SATA HDD 或 SSD。 不支援其他的裝置。 盡可能不要寄送外接式硬碟或 NAS 等裝置。資料中心在情況允許下會退回這類裝置，否則會予以丟棄。
>
>

如果磁碟上的資料是要匯入到 Azure 儲存體，請遵循下列步驟。
### <a name="step-1-prepare-the-drives-using-waimportexport-tool-and-generate-journal-files"></a>步驟 1：使用 WAImportExport 工具準備磁碟機，並產生日誌檔案。

1.  識別要匯入到 Azure 儲存體的資料。 這可能是本機伺服器或網路共用上的目錄和獨立檔案。
2.  根據資料的大小總計，採購所需的 2.5 英吋 SSD 或 2.5 英吋/3.5 英吋 SATA II/III 硬碟機數目。
3.  直接使用 SATA 或外接式 USB 轉接器，將硬碟連結到 Windows 電腦。
4.  在每個硬碟上建立單一 NTFS 磁碟區，並為磁碟區指派磁碟機代號。 沒有裝載點。
5.  若要在 Windows 電腦上啟用加密，請在 NTFS 磁碟區上啟用 BitLocker 加密。 請使用 https://technet.microsoft.com/en-us/library/cc731549(v=ws.10).aspx \(英文\) 上的指示。
6.  使用複製與貼上、拖曳和置放，或是 Robocopy 或任何類似的工具，完整地將資料複製到這些已加密的單一 NTFS 磁碟區。
7.  從 https://www.microsoft.com/en-us/download/details.aspx?id=42659 \(英文\) 下載 WAImportExport V1
8.  將檔案解壓縮至預設資料夾 waimportexportv1。 例如，C:\WaImportExportV1  
9.  以系統管理員身分執行並開啟 PowerShell 或命令列，然後將目錄變更為解壓縮後的資料夾。 例如，cd C:\WaImportExportV1
10. 將下列命令列複製到「記事本」，然後編輯以建立命令列。
  ./WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#1 /sk:***== /t:D /bk:*** /srcdir:D:\ /dstdir:ContainerName/ /skipwrite
    
    /j: 參數是稱為日誌檔案 (具有 .jrn 副檔名) 的檔案名稱。 日誌檔案是針對每個磁碟機所產生，因此建議使用磁碟機序號作為日誌檔案的名稱。
    /sk: 參數是 Azure 儲存體帳戶金鑰。 /t: 參數是要寄送之磁碟的磁碟機代號。 例如，D。/bk: 參數是磁碟機的 BitLocker 金鑰。/srcdir: 參數是要寄送之磁碟的磁碟機代號，其後要緊接著 :\。 例如，D:\。
    /dstdir: 參數是資料所要匯入的 Azure 儲存體容器名稱。
    /skipwrite 
    
11. 針對每個需要寄送的磁碟重複步驟 10。
12. 每次執行命令列時，都會使用 /j: 參數提供的日誌檔案名稱來建立日誌檔案。

### <a name="step-2-create-an-import-job-on-azure-portal"></a>步驟 2：在 Azure 入口網站上建立匯入作業。

1. 登入 https://portal.azure.com/，在 [更多服務] -> [儲存體] -> [匯入/匯出作業] 底下，按一下 [建立匯入/匯出作業]。

2. 在 [基本] 區段中，選取 [匯入至 Azure]，然後依序輸入作業名稱的字串、選取訂用帳戶、輸入或選取資源群組。 輸入匯入作業的描述性名稱。 請注意，您輸入的名稱只能包含小寫字母、數字、連字號和底線，必須以字母開頭，且不得包含空格。 當工作正在進行中和一旦完成後，您會使用您所選的名稱來進行追蹤。

3. 在 [作業詳細資料] 區段中，上傳在磁碟機準備步驟中取得的磁碟機日誌檔案。 如果使用的是 waimportexport.exe version1，您需要針對已備妥的每個磁碟機上傳一個檔案。 選取將會在「匯入目的地」儲存體帳戶區段中匯入資料的儲存體帳戶。 系統會根據所選儲存體帳戶的區域，自動填入「放置」位置。
   
   ![建立匯入工作 - 步驟 3](./media/storage-import-export-service/import-job-03.png)
4. 在 [退貨運送資訊] 區段中，請從下拉式清單中選取貨運公司，並輸入您使用該貨運公司建立的有效貨運公司客戶編號。 當匯入作業完成時，Microsoft 會透過此帳戶將磁碟機寄還給您。 提供完整且有效的連絡人名稱、電話、電子郵件、街道地址、城市、郵遞區號、州/省和國家/地區。
   
5. 在 [摘要] 區段中，會提供將磁碟寄送至 Azure DC 所使用的 Azure DataCenter 交貨地址。 請確認出貨標籤上有提及作業名稱和完整的地址。 

6. 在 [摘要] 頁面上按一下 [確定]，以完成匯入作業的建立。

### <a name="step-3-ship-the-drives-to-the-azure-datacenter-shipping-address-provided-in-step-2"></a>步驟 3：將磁碟機寄送至步驟 2 中所提供的 Azure 資料中心交貨地址。
FedEx、UPS 或 DHL 均可將包裹寄送至 Azure DC。

### <a name="step-4-update-the-job-created-in-step2-with-tracking-number-of-the-shipment"></a>步驟 4：使用寄送的追蹤號碼更新步驟 2 中建立的工作。
寄送磁碟之後，返回 Azure 入口網站上的 [匯入/匯出] 頁面，然後使用下列步驟來更新追蹤號碼 a) 瀏覽至匯入作業並按一下 b) 按一下 [在磁碟區運送後即更新作業狀態及追蹤資訊]。 c) 選取 [Mark as shipped] \(標示為已寄送\) 核取方塊 d) 提供貨運公司和追蹤號碼。
如果在建立工作的 2 個星期內沒有更新追蹤號碼，該工作會過期。 您可以在入口網站儀表板上追蹤作業進度。 [檢視您的作業狀態](#viewing-your-job-status)，以查看上一節中每個作業狀態的意義。

## <a name="when-should-i-use-the-azure-importexport-service"></a>Azure 匯入/匯出服務的使用時機？
請考慮在透過網路上傳或下載資料速度太慢，或是取得額外的網路頻寬成本高昂時使用 Azure 匯入/匯出服務。

您可以使用此服務的案例，例如︰

* 將資料移轉至雲端︰快速地將大量資料移至 Azure，並符合成本效益。
* 內容發佈︰快速將資料傳送至您的客戶網站。
* 備份︰備份內部部署資料以便儲存在 Azure 儲存體中。
* 資料復原︰復原儲存在儲存體中的大量資料，並將它傳遞到您的內部部署位置。

## <a name="prerequisites"></a>先決條件
在本節中，我們列出了使用此服務所需的必要條件。 請先仔細檢閱，再運送您的磁碟機。

### <a name="storage-account"></a>儲存體帳戶
您必須有現有的 Azure 訂用帳戶以及一或多個儲存體帳戶，才能使用匯入/匯出服務。 Azure 匯入/匯出功能僅支援傳統、Blob 儲存體帳戶，和一般用途 v1 儲存體帳戶。 每項工作都只能從僅只一個儲存體帳戶收送資料。 換句話說，單一匯入/匯出作業不能跨越多個儲存體帳戶。 如需建立新儲存體帳戶的詳細資訊，請參閱 [如何建立儲存體帳戶](storage-create-storage-account.md#create-a-storage-account)(英文)。

### <a name="data-types"></a>資料類型
您可以使用 Azure 匯入/匯出服務，將資料複製到**區塊** Blob、**分頁** Blob 或**檔案**。 相反地，您只能使用此服務從 Azure 儲存體匯出**區塊** Blob、**分頁** Blob 或**附加** Blob。 服務僅支援將 Azure 檔案服務匯入到 Azure 儲存體。 目前不支援匯出 Azure 檔案服務。

### <a name="job"></a>工作 (Job)
若要開始進行儲存體的匯入或匯出程序，請先建立工作。 此工作可以是「匯入工作」或「匯出工作」：

* 當您要將內部部署的資料移轉至 Azure 儲存體帳戶時，請建立匯入作業。
* 當您要將目前儲存於儲存體帳戶中的資料移轉至要運送給 Microsoft 的硬碟時，請建立匯出作業。 當您建立工作時，您會通知匯入/匯出服務您將運送一或多個硬碟至 Azure 資料中心。

* 若為匯入工作，您將運送含有資料的硬碟。
* 若為匯出工作，您將運送空的硬碟。
* 您可以針對每個工作運送最多 10 個硬碟。

您可以使用 Azure 入口網站或 [Azure 儲存體匯入/匯出 REST API](/rest/api/storageimportexport) 來建立匯入或匯出作業。

> [!Note]
> 2018 年 2 月 28 日之後，將不再支援 RDFE API。 若要繼續使用服務，請移轉至 [ARM 匯入/匯出 REST API](https://github.com/Azure/azure-rest-api-specs/blob/master/specification/storageimportexport/resource-manager/Microsoft.ImportExport/stable/2016-11-01/storageimportexport.json)。 

### <a name="waimportexport-tool"></a>WAImportExport 工具
建立**匯入**工作的第一個步驟就是要準備將運送以進行匯入的磁碟機。 若要準備您的磁碟機，您必須將它連接到本機伺服器，並在本機伺服器上執行 WAImportExport 工具。 此 WAImportExport 工具可協助將您的資料複製到磁碟機、使用 BitLocker 加密硬碟上的資料，以及產生磁碟機日誌檔案。

日誌檔案會儲存您的作業與磁碟機的基本資訊，例如磁碟機序號和儲存體帳戶名稱。 此日誌檔案不會儲存在磁碟機上。 建立匯入工作期間，會使用它。 建立作業的逐步解說詳細資料會在本文稍後提供。

WAImportExport 工具只與 64 位元 Windows 作業系統相容。 請參閱 [作業系統](#operating-system) 一節，以了解支援的特定 OS 版本。

下載最新版的 [WAImportExport 工具](http://download.microsoft.com/download/3/6/B/36BFF22A-91C3-4DFC-8717-7567D37D64C5/WAImportExportV2.zip)。 如需使用 WAImportExport 工具的詳細資訊，請參閱[使用 WAImportExport 工具](storage-import-export-tool-how-to.md)。

>[!NOTE]
>**前一版本︰**您可以[下載 WAImportExpot V1](http://download.microsoft.com/download/0/C/D/0CD6ABA7-024F-4202-91A0-CE2656DCE413/WaImportExportV1.zip) 版本的工具，並參考 [WAImportExpot V1 使用指南](storage-import-export-tool-how-to-v1.md)。 當資料已預先寫入磁碟時，WAImportExpot V1 版本的工具提供**準備磁碟的支援**。 如果可用的唯一金鑰是 SAS 金鑰，您需要使用 WAImportExpot V1 工具。

>

### <a name="hard-disk-drives"></a>硬碟
僅支援使用 2.5 英吋的 SSD 或是 2.5 英吋或 3.5 英吋的 SATA II 或 III 內接式 HDD 來搭配匯入/匯出服務。 單一匯入/匯出作業最多可以有 10 個 HDD/SSD，且每個個別 HDD/SSD 的大小不拘。 您可以將大量的磁碟機分散到多個作業，而且可建立的作業數目並無限制。 

若為匯入工作，則只會處理磁碟機上的第一個資料磁碟區。 此資料磁碟區必須以 NTFS 格式化。

> [!IMPORTANT]
> 此服務不支援隨附內建 USB 介面卡的外接式硬碟。 此外，也無法使用外接式 HDD 機殼內的磁碟；請勿傳送外接式 HDD。
> 
> 

下列是用來將資料複製到內部 HDD 的外部 USB 介面卡清單。 Anker 68UPSATAA-02BU Anker 68UPSHHDS-BU Startech SATADOCK22UE Orico 6628SUS3-C-BK (6628 Series) Thermaltake BlacX Hot-Swap SATA External Hard Drive Docking Station (USB 2.0 & eSATA)

### <a name="encryption"></a>加密
必須使用「BitLocker 磁碟機加密」將磁碟機上的資料加密。 此加密會在資料傳輸時保護資料。

對於匯入作業，有兩種方式可執行加密。 第一種方式是在磁碟機準備期間執行 WAImportExport 工具時，於使用資料集 CSV 檔案時指定選項。 第二種方式是在磁碟機上手動啟用 BitLocker 加密，並在磁碟機準備期間，於執行 WAImportExport 工具命令列時在磁碟機集 CSV 中指定加密金鑰。

針對匯出作業，您的資料複製到磁碟機之後，服務會使用 BitLocker 將磁碟機加密，然後才運送回去給您。 加密金鑰會透過 Azure 入口網站提供給您。  

### <a name="operating-system"></a>作業系統
您可以使用下列其中一個 64 位元作業系統，使用 WAImportExport 工具準備硬碟，然後再將磁碟機運送到 Azure︰

Windows 7 Enterprise、Windows 7 Ultimate、Windows 8 Pro、Windows 8 Enterprise、Windows 8.1 Pro、Windows 8.1 Enterprise、Windows 10<sup>1</sup>、Windows Server 2008 R2、Windows Server 2012、Windows Server 2012 R2。 所有這些作業系統都支援 BitLocker 磁碟機加密。

### <a name="locations"></a>位置
Azure 匯入/匯出服務支援與所有公用 Azure 儲存體帳戶相互複製資料。 您可以將硬碟運送到列出的其中一個位置。 如果您的儲存體帳戶是未在這裡指定的公用 Azure 位置，當您使用 Azure 入口網站或匯入/匯出 REST API 建立工作時，將會提供替代的運送位置。

支援的運送位置︰

* 美國東部
* 美國西部
* 美國東部 2
* 美國西部 2
* 美國中部
* 美國中北部
* 美國中南部
* 美國中西部
* 北歐
* 西歐
* 東亞
* 東南亞
* 澳洲東部
* 澳大利亞東南部
* 日本西部
* 日本東部
* 印度中部
* 印度南部
* 印度西部
* 加拿大中部
* 加拿大東部
* 巴西南部
* 韓國中部
* 美國政府維吉尼亞州
* 美國政府愛荷華州
* 美國 DoD 東部
* 美國國防部中央
* 中國東部
* 中國北部
* 英國南部
* 德國中部
* 德國東北部

### <a name="shipping"></a>運送中
**運送磁碟機到資料中心︰**

建立匯入或匯出工作時，會提供您其中一個支援位置的運送地址，讓您運送磁碟機。 提供的運送地址會取決於您儲存體帳戶的位置，但可能不會與您的儲存體帳戶位置相同。

您可以利用 FedEx、UPS 或 DHL，將磁碟機運送至交貨地址。

**資料中心運送磁碟機︰**

建立匯入或匯出作業時，您必須提供送回磁碟機的寄件地址，以便 Microsoft 在您的作業完成後運回磁碟機。 確定您提供有效的寄件地址，以避免處理延遲。

貨運公司應有適當的追蹤，才能維持監管鏈。 您必須提供有效的 FedEx、UPS 或 DHL 貨運公司客戶編號，以供 Microsoft 用於送回磁碟機。 從美國和歐洲地點送回磁碟機需要 FedEx、UPS 或 DHL 客戶編號。 從亞洲和澳洲位置送回磁碟機則需要 DHL 客戶編號。 如果您沒有客戶編號，您可以建立 [FedEx](http://www.fedex.com/us/oadr/) (適用於美國和歐洲) 或 [DHL](http://www.dhl.com/) (亞洲和澳洲) 貨運業者帳戶。 如果您已經有貨運公司客戶編號，請確認它有效。

在寄送包裹時，您必須遵守 [Microsoft Azure 服務條款](https://azure.microsoft.com/support/legal/services-terms/)中的條款。

> [!IMPORTANT]
> 請注意，您寄送的實體媒體可能需要跨國界。 您必須確定實體媒體和資料的匯入和/或匯出符合相關管轄法律。 在寄出實體媒體之前，請洽詢顧問來確認您的媒體和資料可以合法地寄到所識別的資料中心。 這有助於確保它可及時送抵 Microsoft。 例如，任何跨國界且需要附上商業發票的包裹 (除了跨歐盟內的國界以外)。 您可以從承運業者網站上列印出商業發票的完整複本。 商業發票的範例為 [DHL 商業發票 (英文)](http://invoice-template.com/wp-content/uploads/dhl-commercial-invoice-template.pdf) 和 [FedEx 商業發票 (英文)](http://images.fedex.com/downloads/shared/shipdocuments/blankforms/commercialinvoice.pdf)。 請確定 Microsoft 未被指定為匯出者。
> 
> 

## <a name="how-does-the-azure-importexport-service-work"></a>Azure 匯入/匯出服務如何運作？
您可以建立作業並運送硬碟機到 Azure 資料中心，在內部部署網站與 Azure 儲存體之間使用 Azure 匯入/匯出服務傳輸資料。 您運送的每個硬碟都與單一作業相關聯。 每個工作都與單一儲存體帳戶相關聯。 請仔細檢閱 [＜必要條件＞一節](#pre-requisites)，了解此服務的細節，例如支援的資料類型、磁碟類型、位置和運送。

在本節中，會說明與匯入和匯出工作有關的高階步驟。 稍後在[快速入門](#quick-start)一節中，會提供如何建立匯入和匯出作業的逐步指示。

### <a name="inside-an-import-job"></a>匯入作業之內
概括而言，匯入工作包含下列步驟︰

* 決定要匯入的資料，以及您需要的磁碟機數目。
* 在 Azure 儲存體中識別目的地 Blob 或檔案位置。
* 使用 WAImportExport 工具將資料複製到一或多個硬碟，並用 BitLocker 將它們加密。
* 使用 Azure 入口網站或匯入/匯出 REST API，在您的目標儲存體帳戶中建立匯入工作。 如果使用 Azure 入口網站，請上傳磁碟機日誌檔案。
* 提供送回用的寄件地址和貨運公司客戶編號，以便將磁碟機送回給您。
* 將硬碟機運送到工作建立期間提供的運送地址。
* 在匯入工作詳細資料中更新貨件追蹤號碼，並提交匯入工作。
* Azure 資料中心收到磁碟機並進行處理。
* 使用您的貨運業者帳戶，將磁碟機送到匯入作業中提供的寄件地址。
  
    ![圖 1: 匯入工作流程](./media/storage-import-export-service/importjob.png)

### <a name="inside-an-export-job"></a>匯出作業之內
> [!IMPORTANT]
> 服務僅支援 Azure Blob 的匯出，而不支援 Azure 檔案的匯出。
> 
>

概括而言，匯出工作包含下列步驟︰

* 決定要匯出的資料，以及您需要的磁碟機數目。
* 在 Blob 儲存體中，識別資料的來源 Blob 或容器路徑。
* 使用 Azure 入口網站或匯入/匯出 REST API，在您的來源儲存體帳戶中建立匯出工作。
* 在匯出工作中，指定資料的來源 Blob 或容器路徑。
* 提供送回用的寄件地址和貨運公司客戶編號，以便將磁碟機送回給您。
* 將硬碟機運送到工作建立期間提供的運送地址。
* 在匯出工作詳細資料中更新貨件追蹤號碼，並提交匯出工作。
* Azure 資料中心會收到磁碟機並進行處理。
* 磁碟機會使用 BitLocker 加密；可透過 Azure 入口網站取得金鑰。  
* 使用您的貨運業者帳戶，將磁碟機送到匯入作業中提供的寄件地址。
  
    ![圖 2: 匯出工作流程](./media/storage-import-export-service/exportjob.png)

### <a name="viewing-your-job-and-drive-status"></a>檢視您的工作和磁碟機狀態
您可以在 Azure 入口網站中追蹤匯入或匯出工作的狀態。 按一下 [匯入/匯出] 索引標籤。您的作業清單會隨即出現在頁面上。

![檢視工作狀態](./media/storage-import-export-service/jobstate.png)

視您的磁碟機在流程中所處的位置，您會看到下列其中一個作業狀態。

| 工作狀態 | 說明 |
|:--- |:--- |
| 建立中 | 工作建立之後，其狀態會設定為「建立中」。 工作處於「建立中」狀態時，匯入/匯出服務會假設磁碟機尚未運送到資料中心。 工作最多可保持「建立中」狀態兩週，超過後會由服務自動刪除。 |
| 運送中 | 在您運送包裹之後，您應該更新 Azure 入口網站中的追蹤資訊。  這會使工作變成「運送中」。 工作最多將保持「運送中」狀態兩週。 
| 已收到 | 資料中心收到所有磁碟機之後，工作狀態會設為「已收到」。 |
| 傳輸中 | 一旦有至少一個磁碟機已開始處理，工作狀態將會設為「傳輸中」。 如需詳細資訊，請參閱以下的＜磁碟機狀態＞一節。 |
| 包裝中 | 所有磁碟機處理完成之後，直到磁碟機寄回給您為止，工作將會設為「包裝中」狀態。 |
| Completed | 所有磁碟機都寄回給客戶之後，如果工作已完成且沒有錯誤，工作將會設為「已完成」狀態。 「已完成」狀態持續 90 天後，將會自動刪除工作。 |
| 關閉 | 所有磁碟機都寄回給客戶之後，如果工作處理期間發生任何錯誤，工作將會設為「關閉」狀態。 「關閉」狀態持續 90 天後，將會自動刪除工作。 |

下表說明個別磁碟機在匯入或匯出作業轉換過程中的生命週期。 現在已可從 Azure 入口網站檢視作業中每個磁碟機的目前狀態。
下表說明工作中的每個磁碟機可能經歷的各個狀態。

| 磁碟機狀態 | 說明 |
|:--- |:--- |
| 已指定 | 針對匯入作業，從 Azure 入口網站建立作業時，磁碟機的初始狀態會是「已指定」狀態。 針對匯出工作，因為建立工作時未指定任何磁碟機，初始磁碟機狀態會是「已收到」狀態。 |
| 已收到 | 當匯入/匯出服務操作員已處理從貨運公司收到要進行匯入工作的磁碟機後，磁碟機會轉換成「已收到」狀態。 針對匯出工作，初始磁碟機狀態會是「已收到」狀態。 |
| 從未收到 | 當工作的包裹抵達但包裹未包含磁碟機時，磁碟機會變成「從未收到」狀態。 如果自服務收到出貨資訊起已經過兩週，包裹卻仍未抵達資料中心，磁碟機也會變成此狀態。 |
| 傳輸中 | 當服務開始將磁碟機中的資料傳輸到 Microsoft Azure 儲存體時，磁碟機將變成「傳輸中」狀態。 |
| Completed | 當服務已順利傳輸所有資料而未產生錯誤時，磁碟機會變成「已完成」狀態。
| 已完成 (其他資訊) | 服務複製磁碟機的資料時若發生一些問題，磁碟機會變成「已完成 (其他資訊)」狀態。 資訊可能包含錯誤、警告或有關覆寫 Blob 的資訊訊息。
| 已寄回 | 當磁碟機從資料中心運回原地址時，磁碟機會變成「已寄回」狀態。 |

這個來自 Azure 入口網站的影像會顯示範例作業的磁碟機狀態：

![檢視磁碟機狀態](./media/storage-import-export-service/drivestate.png)

下表說明磁碟機失敗狀態，以及針對每個狀態採取的動作。

| 磁碟機狀態 | Event | 解決方法/後續步驟 |
|:--- |:--- |:--- |
| 從未收到 | 標示為「從未收到」的磁碟機 (因為在工作的運送過程中未收到) 在另一批貨運中送達。 | 作業小組會將磁碟機變成「已收到」狀態。 |
| N/A | 不屬於任何工作的磁碟機在另一個工作中送達資料中心。 | 磁碟機會標示為額外的磁碟機，並會在與原始包裹相關聯的工作完成時寄回給客戶。 |

### <a name="time-to-process-job"></a>處理工作的時間
處理匯入/匯出作業所需的時間量，會根據不同的因素而有所差異，例如運送時間、作業類型、複製的資料類型及大小，以及所提供磁碟的大小。 匯入/匯出服務沒有 SLA，但收到磁碟之後，服務會努力在 7 到 10 天內完成複製。 您可以使用 REST API 更仔細地追蹤工作進度。 在列出工作作業中有一個完成百分比的參數，它可以指出複製進度。 如果您需要對完成一項時間緊迫的匯入/匯出工作的預估，請連絡我們。

### <a name="pricing"></a>價格
**磁碟機處理費用**

在匯入或匯出工作當中，對於所處理的每個磁碟機會有一項磁碟機處理費用。 請查看 [Azure 匯入/匯出價格](https://azure.microsoft.com/pricing/details/storage-import-export/)的詳細資料。

**運送成本**

當您運送磁碟機至 Azure 時，您要支付運送成本給運送的貨運業者。 當 Microsoft 將磁碟機送回給您時，運送成本將計入您在建立作業時提供的貨運業者帳戶。

**交易成本**

將資料匯入 Azure 儲存體時，除了標準儲存體交易成本之外，無需支付其他的交易成本。 從 Blob 儲存體匯出資料時，適用標準輸出費用。 如需交易成本的更多詳細資料，請參閱 [資料傳輸價格](https://azure.microsoft.com/pricing/details/data-transfers/)



## <a name="how-to-import-data-into-azure-file-storage-using-internal-sata-hdds-and-ssds"></a>如何使用內部 SATA HDD 和 SSD 將資料匯入 Azure 檔案儲存體中？
如果磁碟上的資料是要匯入 Azure 檔案儲存體，請遵循下列步驟。
使用 Azure 匯入/匯出服務匯入資料的第一個步驟，是使用 WAImportExport 工具準備磁碟機。 請遵循下列步驟來準備您的磁碟機。

1. 識別要匯入 Azure 檔案儲存體的資料。 這可能是本機伺服器或網路共用上的目錄和獨立檔案。  
2. 根據資料的總大小，決定您需要的磁碟機數目。 採購所需的 2.5 英吋 SSD 或 2.5 英吋/3.5 英吋 SATA II/III 硬碟機數目。
4. 決定將複製到每個硬碟的目錄和/或獨立檔案。
5. 建立資料集和磁碟機集的 CSV 檔。
    
  以下是範例資料集 CSV 檔案範例，可將資料匯入為 Azure 檔案：
  
    ```
    BasePath,DstItemPathOrPrefix,ItemType,Disposition,MetadataFile,PropertiesFile
    "F:\50M_original\100M_1.csv.txt","fileshare/100M_1.csv.txt",file,rename,"None",None
    "F:\50M_original\","fileshare/",file,rename,"None",None 
    ```
   在上述範例中，100M_1.csv.txt 會複製到 "fileshare" 的根目錄。 如果 "fileshare" 不存在，便會加以建立。 50M_original 底下的所有檔案和資料夾會以遞迴方式複製到 fileshare。 將保留資料夾結構。

    深入了解[準備資料集 CSV 檔案](storage-import-export-tool-preparing-hard-drives-import.md#prepare-the-dataset-csv-file)。
    


    **磁碟機集 CSV 檔案**

    driveset 旗標的值為 CSV 檔案，其中包含磁碟機代號對應的磁碟清單，使工具可以正確挑選要準備的磁碟清單。 

    下列是磁碟機集 CSV 檔案的範例：
    
    ```
    DriveLetter,FormatOption,SilentOrPromptOnFormat,Encryption,ExistingBitLockerKey
    G,AlreadyFormatted,SilentMode,AlreadyEncrypted,060456-014509-132033-080300-252615-584177-672089-411631 |
    H,Format,SilentMode,Encrypt,
    ```

    上述範例假設有兩個磁碟已連接，且已建立磁碟區代號為 G:\ 與 H:\ 的基本 NTFS 磁碟區。 此工具將格式化並加密裝載 H:\ 的磁碟，將不會格式化或加密裝載磁碟區 G:\ 的磁碟。

    深入了解[準備磁碟機集 CSV 檔案](storage-import-export-tool-preparing-hard-drives-import.md#prepare-initialdriveset-or-additionaldriveset-csv-file)。

6.  使用 [WAImportExport 工具](http://download.microsoft.com/download/3/6/B/36BFF22A-91C3-4DFC-8717-7567D37D64C5/WAImportExport.zip)將資料複製到一或多個硬碟。
7.  您可以在磁碟機集 CSV 中的加密欄位指定 "Encrypt"，以在硬碟上啟用 BitLocker 加密。 或者，您也可以手動在硬碟上啟用 BitLocker 加密和指定 "AlreadyEncrypted"，並在執行工具時於磁碟機集 CSV 中提供金鑰。

8. 完成磁碟準備工作之後，請勿修改硬碟上的資料或日誌檔案。

> [!IMPORTANT]
> 您準備的每個硬碟都會造成一個日誌檔案。 當您使用 Azure 入口網站建立匯入工作時，您必須上傳屬於該匯入工作之磁碟機的所有日誌檔案。 沒有日誌檔案的磁碟機將不會被處理。
> 
>

以下是使用 WAImportExport 工具準備硬碟的命令和範例。

WAImportExport 工具以新的複製工作階段複製目錄和/或檔案的第一複製工作階段使用的 PrepImport 命令：

```
WAImportExport.exe PrepImport /j:<JournalFile> /id:<SessionId> [/logdir:<LogDirectory>] [/sk:<StorageAccountKey>] [/silentmode] [/InitialDriveSet:<driveset.csv>] DataSet:<dataset.csv>
```

**匯入範例 1**

```
WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#1  /sk:************* /InitialDriveSet:driveset-1.csv /DataSet:dataset-1.csv /logdir:F:\logs
```

若要**新增更多磁碟機**，可以建立新的磁碟機集檔案並執行命令，如下所示。 對於後續複製到不同於 InitialDriveset.csv 檔案中指定的磁碟機的工作階段，請指定新的磁碟機集 CSV 檔案，並以值形式提供給參數 "AdditionalDriveSet"。 使用**相同的日誌檔**名稱，並提供**新的工作階段識別碼**。 AdditionalDriveset CSV 檔案的格式與 InitialDriveSet 格式相同 。

```
WAImportExport.exe PrepImport /j:<JournalFile> /id:<SessionId> /AdditionalDriveSet:<driveset.csv>
```

**匯入範例 2**
```
WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#3  /AdditionalDriveSet:driveset-2.csv
```

若要新增其他資料到相同的磁碟機集，可以呼叫 WAImportExport 工具使用於後續複製工作階段的 PrepImport 命令以複製其他檔案/目錄︰對於複製到 InitialDriveset.csv 檔案中指定的相同磁碟機的後續工作階段，請指定**相同的日誌檔**名稱，並提供**新的工作階段識別碼**不需提供儲存體帳戶金鑰。

```
WAImportExport PrepImport /j:<JournalFile> /id:<SessionId> /j:<JournalFile> /id:<SessionId> [/logdir:<LogDirectory>] DataSet:<dataset.csv>
```

**匯入範例 3**

```
WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#2  /DataSet:dataset-2.csv
```

如需使用 WAImportExport 工具的詳細資訊，請參閱[準備硬碟以進行匯入](storage-import-export-tool-preparing-hard-drives-import.md)。

此外，請參閱[針對匯入作業準備硬碟的範例工作流程](storage-import-export-tool-sample-preparing-hard-drives-import-job-workflow.md)，以取得更多詳細的逐步指示。  



## <a name="create-an-export-job"></a>建立匯出工作
建立匯出作業以通知匯入/匯出服務，您會將一或多個空磁碟機送到資料中心，以便將資料從儲存體帳戶匯出至磁碟機，然後再將這些磁碟機運送給您。

### <a name="prepare-your-drives"></a>準備磁碟機
準備磁碟機以進行匯出工作時，建議進行下列預先檢查︰

1. 請使用 WAImportExport 工具的 PreviewExport 命令檢查所需的磁碟數目。 如需詳細資訊，請參閱 [Previewing Drive Usage for an Export Job (預覽匯出作業的磁碟機使用量)](https://msdn.microsoft.com/library/azure/dn722414.aspx)。 它可根據您要使用的磁碟機大小，協助您預覽所選取之 Blob 的磁碟機使用情況。
2. 請檢查您可以對為了匯出工作而運送的硬碟讀取/寫入。

### <a name="create-the-export-job"></a>建立匯出作業
1. 若要建立匯出作業，請瀏覽至 [更多服務] -> [儲存體]-> Azure 入口網站上的 [匯入/匯出作業]。 按一下 [建立匯入/匯出作業]。
2. 在步驟 1 的「基本概念」中，選取 [Export from Azure] \(從 Azure 匯出\)、輸入作業名稱的字串、選取訂用帳戶、輸入或選取資源群組。 輸入匯入作業的描述性名稱。 請注意，您輸入的名稱只能包含小寫字母、數字、連字號和底線，必須以字母開頭，且不得包含空格。 當工作正在進行中和一旦完成後，您將使用您所選的名稱來進行追蹤。 提供負責處理此匯出作業之人員的連絡資訊。 

3. 在步驟 2 的「作業詳細資料」中，選取將會在 [儲存體帳戶] 區段中匯出資料的儲存體帳戶。 系統將會根據所選儲存體帳戶的區域，自動填入「放置」位置。 指定您要從儲存體帳戶匯出至空白磁碟機的 Blob 資料。 您可以選擇匯出儲存體帳戶中所有的 Blob 資料，也可以指定要匯出哪幾個或哪幾組 Blob。
   
   若要指定要匯出的 Blob，請使用 [等於]  選取器，指定 Blob 的相對路徑 (以容器名稱開頭)。 使用 *$root* 指定根容器。
   
   若要指定以某個首碼開頭的所有 Blob，請使用 [開頭為]  選取器，指定首碼 (以正斜線 '/' 開頭)。 此首碼可以是容器名稱的首碼、完整容器名稱，或是後面接著 Blob 名稱首碼的完整容器名稱。
   
   下表顯示有效 Blob 路徑範例：
   
   | 選取器 | Blob 路徑 | 說明 |
   | --- | --- | --- |
   | 開頭為 |/ |匯出儲存體帳戶中的所有 Blob |
   | 開頭為 |/$root/ |匯出根容器中的所有 Blob |
   | 開頭為 |/book |匯出任何容器中以首碼 **book** |
   | 開頭為 |/music/ |匯出容器 **music** |
   | 開頭為 |/music/love |匯出容器 **music** 中以首碼 **love** 開頭的所有 Blob |
   | 等於 |$root/logo.bmp |匯出根容器中的 Blob **logo.bmp** |
   | 等於 |videos/story.mp4 |匯出容器 **videos** 中的 Blob **story.mp4** |
   
   您必須提供有效的格式的 Blob 路徑，以避免在處理期間發生錯誤，如以下螢幕擷取畫面所示。
   
   ![建立匯出工作 - 步驟 3](./media/storage-import-export-service/export-job-03.png)

4. 在步驟 3 的「退貨運送資訊」中，請從下拉式清單中選取貨運公司，並輸入您使用該貨運公司建立的有效貨運公司客戶編號。 當匯入作業完成時，Microsoft 會透過此廠商將磁碟機寄還給您。 提供完整且有效的連絡人名稱、電話、電子郵件、街道地址、城市、郵遞區號、州/省和國家/地區。
   
 5. 在 [摘要] 頁面中，會提供將磁碟寄送至 Azure DC 所使用的 Azure DataCenter 交貨地址。 請確認出貨標籤上有提及作業名稱和完整的地址。 

6. 在 [摘要] 頁面上按一下 [確定]，以完成匯入作業的建立

7. 寄送磁碟之後，返回 Azure 入口網站上的 [匯入/匯出] 頁面，然後 a) 瀏覽至匯入作業並按一下 b) 按一下 [Update job status and tracking info once drives are shipped] \(一旦磁碟機寄送出去之後，更新作業狀態和追蹤資訊\)。 
     c) 選取 [Mark as shipped] \(標示為已寄送\) 核取方塊 d) 提供貨運公司和追蹤號碼。
    
   如果在建立工作的 2 個星期內沒有更新追蹤號碼，該工作會過期。
   
8. 您可以在入口網站儀表板追蹤工作進度。 [檢視您的作業狀態](#viewing-your-job-status)，以查看上一節中每個作業狀態的意義。

   > [!NOTE]
   > 如果要匯出的 Blob 在複製到硬碟機時為使用中，則 Azure 匯入/匯出服務會擷取 Blob 的快照集，並複製此快照集。
   > 
   > 
 
9. 收到具有匯出資料的磁碟機之後，您可以檢視並複製由服務為您的磁碟機產生的 BitLocker 金鑰。 瀏覽至 Azure 入口網站中的匯出作業，然後按一下 [匯入/匯出] 索引標籤。從清單中選取您的匯出作業，然後按一下 [BitLocker 金鑰] 選項。 BitLocker 金鑰如下所示：
   
   ![檢視匯出工作的 BitLocker 金鑰](./media/storage-import-export-service/export-job-bitlocker-keys.png)

請讀完下面的＜常見問題集＞一節，因為其中包含使用本服務時，客戶最常遇到的問題。

## <a name="frequently-asked-questions"></a>常見問題集

**我可以使用 Azure 匯入/匯出服務複製 Azure 檔案儲存體嗎？**

是，Azure 匯入/匯出服務支援匯入至 Azure 檔案儲存體。 目前不支援 Azure 檔案匯出。

**Azure 匯入/匯出服務可用於 CSP 訂用帳戶嗎？**

Azure 匯入/匯出服務確實支援 CSP 訂用帳戶。

**我可以略過匯入工作的磁碟機準備步驟，或是可以準備磁碟機而不複製嗎？**

您想要運送以便匯入資料的任何磁碟機，都必須使用 Azure WAImportExport 工具備妥。 您必須使用 WAImportExport 工具，將資料複製到磁碟機。

**我需要在建立匯出工作時執行任何磁碟準備工作嗎？**

不需要，但建議先執行一些前置檢查。 請使用 WAImportExport 工具的 PreviewExport 命令檢查所需的磁碟數目。 如需詳細資訊，請參閱 [Previewing Drive Usage for an Export Job (預覽匯出作業的磁碟機使用量)](https://msdn.microsoft.com/library/azure/dn722414.aspx)。 它可根據您要使用的磁碟機大小，協助您預覽所選取之 Blob 的磁碟機使用情況。 也請檢查您可以對為了匯出作業而運送的硬碟進行讀取和寫入。

**如果我不小心傳送不符支援需求的 HDD，則會發生什麼情況？**

Azure 資料中心會將不符支援需求的磁碟機退回給您。 如果包裹中只有部分磁碟機符合支援需求，將會處理這些磁碟機，而不符需求的磁碟機則會退回給您。

**我可以取消工作嗎？**

您可以在工作狀態為 [建立中] 或 [運送中] 時取消工作。

**我可以在 Azure 入口網站中檢視已完成工作的狀態多久？**

您最多可以檢視之前 90 天內已完成作業的狀態。 已完成的作業會在 90 天後刪除。

**如果我想匯入或匯出的磁碟機超過 10 個，我該怎麼做？**

在匯入/匯出服務中，一項匯入或匯出工作只能參考單一工作中的 10 個磁碟機。 如果想要運送的磁碟機超過 10 個，可以建立多項工作。 與相同工作相關聯的磁碟機必須一起在相同的包裹中運送。
若資料容量跨越多個磁碟匯入工作，Microsoft 會提供指導與協助。 如需詳細資訊，請連絡 bulkimport@microsoft.com

**服務會在退回磁碟機前進行格式化嗎？**

編號 所有磁碟機都使用 BitLocker 加密。

**我可以為了匯入/匯出工作向 Microsoft 購買磁碟機嗎？**

編號 您必須針對匯入和匯出工作運送自己的磁碟機。

** 如何存取這個服務所匯入的資料**

Azure 儲存體帳戶下的資料可以透過 Azure 入口網站或使用稱為「儲存體總管」的獨立工具進行存取。 https://docs.microsoft.com/azure/vs-azure-tools-storage-manage-with-storage-explorer 

**匯入作業完成後，我的資料在儲存體帳戶中外觀如何？將保留我的目錄階層嗎？**

準備匯入作業的硬碟時，目的地會由資料集 CSV 的 DstBlobPathOrPrefix 欄位指定。 這是硬碟的資料將複製到其中的儲存體帳戶目的地容器。 在此目的地容器內，會針對硬碟裡的資料夾建立虛擬目錄，並針對檔案建立 Blob。 

**如果磁碟機中具有我儲存體帳戶裡已存在的檔案，那麼服務將會覆寫我儲存體帳戶裡現有的 Blob 或檔案嗎？**

準備磁碟機時，您可以使用資料集 CSV 檔案中名為 /Disposition:<rename|no-overwrite|overwrite> 的欄位，指定是否應該覆寫目的地檔案還是予以忽略。 根據預設，服務將為新檔案重新命名，而不會覆寫現有的 Blob 或檔案。

**WAImportExport 工具與 32 位元作業系統相容嗎？**
編號 WAImportExport 工具只與 64 位元 Windows 作業系統相容。 請參閱 [先決條件](#pre-requisites) 中的＜作業系統＞一節，以取得支援 OS 版本的完整清單。

**我的包裹中除了硬碟以外還應該包含任何東西嗎？**

僅限寄送硬碟機。 請勿加入電源線或 USB 纜線等項目。

**我必須用 FedEx 還是 DHL 運送我的磁碟機？**

您可使用任何已知的貨運公司，例如 FedEx、DHL、UPS 或郵政服務將磁碟機運送到資料中心。 不過，如需從資料中心將磁碟機送回給您，在美國和歐洲地區，您必須提供 FedEx 客戶編號，而在亞洲和澳洲地區，則是提供 DHL 客戶編號。

**將我的磁碟機進行國際運送有任何限制嗎？**

請注意，您寄送的實體媒體可能需要跨國界。 您必須確定實體媒體和資料的匯入和/或匯出符合相關管轄法律。 在寄出實體媒體之前，請洽詢顧問來確認您的媒體和資料可以合法地寄到所識別的資料中心。 這有助於確保及時送達 Microsoft。

**建立作業時，運送地址是與我的儲存體帳戶位置不同的位置。我該怎麼辦？**

某些儲存體帳戶位置會對應至替代的運送位置。 先前可用的運送位置也可能暫時對應到替代位置。 工作建立期間，請務必檢查提供的運送地址，然後才運送磁碟機。

**運送我的磁碟機時，貨運公司要求資料中心連絡人地址和電話號碼。我該提供什麼？**

在建立作業時，提供電話號碼和 DC 地址。

**可以使用 Azure 匯入/匯出服務將 PST 信箱及 SharePoint 資料複製到 O365 嗎？**

請參閱 [將 PST 檔案或 SharePoint 資料匯入至 Office 365](https://technet.microsoft.com/library/ms.o365.cc.ingestionhelp.aspx)。

**可以使用 Azure 匯入/匯出服務，將我的離線備份複製到 Azure 備份服務嗎？**

請參閱 [Azure 備份中的離線備份工作流程](../../backup/backup-azure-backup-import-export.md)。

**一次出貨最多可以有多少個 HDD？**

一次出貨的 HDD 數量沒有限制，而且如果磁碟屬於多個作業，建議 a) 在磁碟加上所對應作業名稱的標籤。 b) 以帶有 -1、-2 等尾碼的追蹤號碼來更新作業。
  
**磁碟匯入/匯出所支援的最大區塊 Blob 和分頁 Blob 大小是多少？**

最大區塊 Blob 大小大約是 4.768 TB 或 5,000,000 MB。
最大分頁 Blob 大小為 1 TB。

**磁碟匯入/匯出是否支援 AES 256 加密？**

Azure 匯入/匯出服務預設會透過 AES 128 BitLocker 加密進行加密，但可在複製資料之前，以手動方式使用 BitLocker 進行加密以提高到 AES 256。 

如果使用 [WAImportExpot V1](http://download.microsoft.com/download/0/C/D/0CD6ABA7-024F-4202-91A0-CE2656DCE413/WaImportExportV1.zip)，以下是命令範例
```
WAImportExport PrepImport /sk:<StorageAccountKey> /csas:<ContainerSas> /t: <TargetDriveLetter> [/format] [/silentmode] [/encrypt] [/bk:<BitLockerKey>] [/logdir:<LogDirectory>] /j:<JournalFile> /id:<SessionId> /srcdir:<SourceDirectory> /dstdir:<DestinationBlobVirtualDirectory> [/Disposition:<Disposition>] [/BlobType:<BlockBlob|PageBlob>] [/PropertyFile:<PropertyFile>] [/MetadataFile:<MetadataFile>] 
```
如果使用 [WAImportExport 工具](http://download.microsoft.com/download/3/6/B/36BFF22A-91C3-4DFC-8717-7567D37D64C5/WAImportExport.zip)，請指定 "AlreadyEncrypted" 並提供磁碟機集 CSV 中的金鑰。
```
DriveLetter,FormatOption,SilentOrPromptOnFormat,Encryption,ExistingBitLockerKey
G,AlreadyFormatted,SilentMode,AlreadyEncrypted,060456-014509-132033-080300-252615-584177-672089-411631 |
```
## <a name="next-steps"></a>後續步驟

* [設定 WAImportExport 工具](storage-import-export-tool-how-to.md)
* [使用 AzCopy 命令列公用程式傳輸資料](storage-use-azcopy.md)
* [Azure 匯入匯出 REST API 範例 (英文)](https://azure.microsoft.com/documentation/samples/storage-dotnet-import-export-job-management/)

