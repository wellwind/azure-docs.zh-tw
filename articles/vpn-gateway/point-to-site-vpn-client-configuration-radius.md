---
title: "建立和安裝適用於 P2S RADIUS 連線的 VPN 用戶端組態檔：PowerShell：Azure | Microsoft Docs"
description: "為使用 RADIUS 驗證的連線建立 Windows、Mac OS X 和 Linux VPN 用戶端組態檔。"
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: jpconnock
editor: 
tags: azure-resource-manager
ms.assetid: 
ms.service: vpn-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/12/2018
ms.author: cherylmc
ms.openlocfilehash: ce914d2fd0472855ad7a17bf64ae43a76ceb5743
ms.sourcegitcommit: 95500c068100d9c9415e8368bdffb1f1fd53714e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/14/2018
---
# <a name="create-and-install-vpn-client-configuration-files-for-p2s-radius-authentication"></a>建立和安裝 VPN 用戶端組態檔以便進行 P2S RADIUS 驗證

若要將透過點對站連線至虛擬網路，您需要設定用來連線的來源用戶端裝置。 您可以從 Windows、Mac OSX 和 Linux 用戶端裝置建立 P2S VPN 連線。 使用 RADIUS 驗證時，有多個驗證選項：使用者名稱/密碼驗證、憑證驗證，以及其他驗證類型。 每一類型驗證的 VPN 用戶端設定都是不一樣的。 若要設定 VPN 用戶端，您可以使用包含必要設定的用戶端組態檔。 本文可協助您為想要使用的 RADIUS 驗證類型建立及安裝 VPN 用戶端設定。

P2S RADIUS 驗證的設定工作流程如下所示：

1. [設定 P2S 連線的 Azure VPN 閘道](point-to-site-how-to-radius-ps.md)。
2. [設定 RADIUS 伺服器以供驗證](point-to-site-how-to-radius-ps.md#radius)。 
3. **取得所選驗證選項適用的 VPN 用戶端組態，並用它設定 VPN 用戶端**。 (本文)
4. [完成 P2S 設定並連線](point-to-site-how-to-radius-ps.md)。

>[!IMPORTANT]
>如果您在產生 VPN 用戶端組態檔後，對點對站 VPN 組態進行任何變更 (例如 VPN 通訊協定類型或驗證類型)，您必須在使用者裝置上產生並安裝新的 VPN 用戶端組態檔。
>
>

若要使用本文中的小節，請先決定要使用哪一種驗證類型：使用者名稱/密碼、憑證，或其他驗證類型。 在每一節中，都會提供 Windows、Mac OS X 和 Linux 適用的步驟 (此時可用的步驟有限)。

## <a name="adeap"></a>使用者名稱/密碼驗證

有兩種方式可設定使用者名稱/密碼驗證。 您可以設定使用 AD 驗證，或不使用 AD 驗證。 無論是哪一種情況，都請您確定所有連線使用者都具有可透過 RADIUS 驗證的使用者名稱/密碼認證。

* 設定使用者名稱/密碼驗證時，您只能建立 EAP-MSCHAPv2 使用者名稱/密碼驗證通訊協定的組態。
* '-AuthenticationMethod' 為 'EapMSChapv2'。

### <a name="usernamefiles"></a> 1.產生 VPN 用戶端設定檔

產生 VPN 用戶端設定檔以與使用者名稱/密碼驗證搭配使用。 您可以使用下列命令來產生 VPN 用戶端設定檔：

```powershell 
New-AzureRmVpnClientConfiguration -ResourceGroupName "TestRG" -Name "VNet1GW" -AuthenticationMethod "EapMSChapv2"
```
 
執行會傳回連結的命令。 將該連結複製並貼到網頁瀏覽器，以下載「VpnClientConfiguration.zip」。 將該檔案解壓縮以檢視下列資料夾： 
 
* **WindowsAmd64** 和 **WindowsX86** - 這些資料夾分別包含 Windows 32 位元和 64 位元的安裝程式套件。 
* **Generic** - 這個資料夾包含用來建立自有 VPN 用戶端設定的一般資訊。 使用者名稱/密碼驗證設定不需要此資料夾。
* **Mac** - 如果您在建立虛擬網路閘道時設定 IKEv2，您會看到名為 'Mac' 的資料夾，其中包含 **mobileconfig** 檔案。 這個檔案用來設定 Mac 用戶端。

如果您已經產生用戶端設定檔，您可以使用 'Get-AzureRmVpnClientConfiguration' Cmdlet 來擷取它們。 不過，如果您對 P2S VPN 設定進行任何變更 (例如，VPN 通訊協定類型或驗證類型)，該設定不會自動更新。 您必須執行 'New-AzureRmVpnClientConfiguration' Cmdlet 來建立新的設定下載。

若要擷取先前產生的用戶端組態檔，請使用下列命令：

```powershell
Get-AzureRmVpnClientConfiguration -ResourceGroupName "TestRG" -Name "VNet1GW"
```

### <a name="setupusername"></a> 2.設定 VPN 用戶端

您可以設定下列 VPN 用戶端：

* [Windows](#adwincli)
* [Mac (OS X)](#admaccli)
* [使用 strongSwan 的 Linux](#adlinuxcli)
 
#### <a name="adwincli"></a>Windows VPN 用戶端設定

您可以在每個 Windows 用戶端電腦上使用相同的 VPN 用戶端組態套件，但前提是版本必須符合用戶端的架構。 如需用戶端支援的作業系統清單，請參閱[常見問題集](vpn-gateway-vpn-faq.md#P2S)的＜點對站＞一節。

請使用下列步驟來設定原生 Windows VPN 用戶端的憑證驗證：

1. 選取 Windows 電腦架構所對應的 VPN 用戶端組態檔。 若是 64 位元的處理器架構，請選擇 'VpnClientSetupAmd64' 安裝程式套件。 若是 32 位元的處理器架構，請選擇 'VpnClientSetupX86' 安裝程式套件。 
2. 若要安裝，請按兩下套件。 如果您看到 SmartScreen 快顯視窗，請按一下 [更多資訊]，然後按一下 [仍要執行]。
3. 在用戶端電腦上，瀏覽至 [網路設定]，然後按一下 [VPN]。 VPN 連線會顯示其連線的虛擬網路名稱。 

#### <a name="admaccli"></a>Mac (OS X) VPN 用戶端設定

1. 選取 **VpnClientSetup mobileconfig** 檔案，並將它傳送給每個使用者。 您可以使用電子郵件或其他方法。

2. 在 Mac 上找出 **mobileconfig** 檔案。

  ![找出 mobilconfig 檔案](./media/point-to-site-vpn-client-configuration-radius/admobileconfigfile.png)
3. 按兩下設定檔加以安裝，請按一下 [繼續]。 設定檔名稱與您的 VNet 名稱相同。

  ![安裝](./media/point-to-site-vpn-client-configuration-radius/adinstall.png)
4. 按一下 [繼續]，以信任設定檔的寄件者並繼續進行安裝。

  ![繼續](./media/point-to-site-vpn-client-configuration-radius/adcontinue.png)
5. 在設定檔安裝期間，您有選項可指定用於 VPN 驗證的使用者名稱和密碼。 不一定要輸入此資訊。 若已指定，系統會儲存此資訊並在您初始連線時自動使用。 按一下 [安裝] 繼續進行。

  ![settings](./media/point-to-site-vpn-client-configuration-radius/adsettings.png)
6. 輸入在您的電腦上安裝設定檔時所需之必要權限的使用者名稱和密碼。 按一下 [SERVICEPRINCIPAL] 。

  ![使用者名稱和密碼](./media/point-to-site-vpn-client-configuration-radius/adusername.png)
7. 安裝之後，設定檔便會顯示在 [設定檔] 對話方塊中。 稍後也可以從 [系統喜好設定] 開啟此對話方塊。

  ![系統喜好設定](./media/point-to-site-vpn-client-configuration-radius/adsystempref.png)
8. 若要存取 VPN 連線，請從 [系統喜好設定] 開啟 [網路] 對話方塊。

  ![網路](./media/point-to-site-vpn-client-configuration-radius/adnetwork.png)
9. VPN 連線會顯示為 [IkeV2 VPN]。 更新 **mobileconfig** 檔案，即可變更此名稱。

  ![connection](./media/point-to-site-vpn-client-configuration-radius/adconnection.png)
10. 按一下 [驗證設定] 。 選擇下拉式清單中的 [使用者名稱]，然後輸入您的認證。 如果您先前已輸入認證，則會自動在下拉式清單中選擇 [使用者名稱] 並預先填入使用者名稱和密碼。 按一下 [確定]  來儲存設定。 此會讓您回到 [網路] 對話方塊。

  ![authenticate](./media/point-to-site-vpn-client-configuration-radius/adauthentication.png)
11. 按一下 [套件] 以儲存變更。 若要起始連線，請按一下 [連線]。

#### <a name="adlinuxcli"></a>使用 strongSwan 進行 Linux VPN 用戶端設定

下列是針對在 Ubuntu 17.0.4 上使用 strongSwan 5.5.1 所建立的指示。 視 Linux 和 strongSwan 的版本而定，實際畫面可能會不一樣。

1. 執行範例中的指令，以開啟**終端機**來安裝 **strongSwan** 和其網路管理員。 如果您收到和 "libcharon-extra-plugins" 有關的錯誤，請使用 "strongswan-plugin-eap-mschapv2" 取代它。

  ```Terminal
  sudo apt-get install strongswan libcharon-extra-plugins moreutils iptables-persistent network-manager-strongswan
  ```
2. 按一下 [網路管理員] 圖示 (向上箭頭/向下箭頭)，然後選取 [編輯連線]。

  ![編輯連線](./media/point-to-site-vpn-client-configuration-radius/EditConnection.png)
3. 按一下 [新增] 按鈕建立新連線。

  ![新增連線](./media/point-to-site-vpn-client-configuration-radius/AddConnection.png)
4. 從下拉式功能表選取 [IPsec/IKEv2 (strongswan)]，然後按一下 [建立]。 您可以在這個步驟中將連線重新命名。

  ![新增 ikev2](./media/point-to-site-vpn-client-configuration-radius/AddIKEv2.png)
5. 從所下載用戶端組態檔的 [Generic] 資料夾開啟 [VpnSettings.xml] 檔案。 尋找名為 **VpnServer** 的標籤，然後複製開頭為 "azuregateway"，結尾為 ".cloudapp.net" 的名稱。

  ![vpn 設定](./media/point-to-site-vpn-client-configuration-radius/VpnSettings.png)
6. 將此名稱貼到新 VPN 連線 [閘道] 區段底下的 [位址] 欄位中。 接下來，按一下 [憑證] 欄位結尾處的資料夾圖示，接著瀏覽至 [Generic] 資料夾，然後選取位於該位置的 [VpnServerRoot] 檔案。
7. 在連線的 [用戶端] 區段底下，為 [驗證] 選擇 [EAP]，然後輸入您的使用者名稱/密碼。 您可能必須選取右邊的鎖定圖示，以儲存此資訊。 然後按一下 [ **儲存**]。

  ![編輯連線設定](./media/point-to-site-vpn-client-configuration-radius/editconnectionsettings.png)
8. 按一下 [網路管理員] 圖示 (向上箭頭/向下箭頭)，然後將游標停留在 [VPN 連線] 上方。 您將會看到您建立的 VPN 連線。 若要起始連線，請選取該連線以建立連線。

  ![連線 radius](./media/point-to-site-vpn-client-configuration-radius/ConnectRADIUS.png)

## <a name="certeap"></a>憑證驗證
 
您可以建立 VPN 用戶端組態檔，以便進行採用 EAP-TLS 通訊協定的 RADIUS 憑證驗證。 通常，企業所發行的憑證可用來驗證 VPN 的使用者。 確定所有連線使用者都已在使用者的裝置上安裝憑證，且 RADIUS 伺服器可以驗證該憑證。
 
* '-AuthenticationMethod' 為 'EapTls'。
* 在憑證驗證期間，用戶端會藉由驗證憑證來驗證 RADIUS 伺服器。 -RadiusRootCert 是 .cer 檔案，其中包含用來驗證 RADIUS 伺服器的根憑證。
* 每個 VPN 用戶端裝置都需要安裝用戶端憑證。
* Windows 裝置有時會有多個用戶端憑證。 在驗證期間，這可能會導致快顯對話方塊列出所有的憑證。 接著，使用者必須選擇要使用的憑證。 指定用戶端憑證應鏈結至的根憑證，即可篩選出正確的憑證。 '-ClientRootCert' 是包含根憑證的 .cer 檔案。這是選擇性參數。 如果您要連線的裝置只有一個用戶端憑證，則不需要指定此參數。

### <a name="certfiles"></a>1.產生 VPN 用戶端設定檔

產生 VPN 用戶端設定檔以與憑證驗證搭配使用。 您可以使用下列命令來產生 VPN 用戶端設定檔：
 
```powershell
New-AzureRmVpnClientConfiguration -ResourceGroupName "TestRG" -Name "VNet1GW" -AuthenticationMethod "EapTls" -RadiusRootCert <full path name of .cer file containing the RADIUS root> -ClientRootCert <full path name of .cer file containing the client root> | fl
```

執行會傳回連結的命令。 將該連結複製並貼到網頁瀏覽器，以下載「VpnClientConfiguration.zip」。 將該檔案解壓縮以檢視下列資料夾：

* **WindowsAmd64** 和 **WindowsX86** - 這些資料夾分別包含 Windows 32 位元和 64 位元的安裝程式套件。 
* **GenericDevice** - 這個資料夾包含用來建立自有 VPN 用戶端設定的一般資訊。

如果您已經產生用戶端設定檔，您可以使用 'Get-AzureRmVpnClientConfiguration' Cmdlet 來擷取它們。 不過，如果您對 P2S VPN 設定進行任何變更 (例如，VPN 通訊協定類型或驗證類型)，該設定不會自動更新。 您必須執行 'New-AzureRmVpnClientConfiguration' Cmdlet 來建立新的設定下載。

若要擷取先前產生的用戶端組態檔，請使用下列命令：

```powershell
Get-AzureRmVpnClientConfiguration -ResourceGroupName "TestRG" -Name "VNet1GW" | fl
```
 
### <a name="setupusername"></a> 2.設定 VPN 用戶端

您可以設定下列 VPN 用戶端：

* [Windows](#certwincli)
* [Mac (OS X)](#certmaccli)
* Linux (支援，尚未有文章提供步驟)

#### <a name="certwincli"></a>Windows VPN 用戶端設定

1. 選取組態套件並將它安裝在用戶端裝置上。 若是 64 位元的處理器架構，請選擇 'VpnClientSetupAmd64' 安裝程式套件。 若是 32 位元的處理器架構，請選擇 'VpnClientSetupX86' 安裝程式套件。 如果您看到 SmartScreen 快顯視窗，請按一下 [更多資訊]，然後按一下 [仍要執行]。 您也可以儲存要在其他用戶端電腦上安裝的套件。
2. 每個用戶端都需要用戶端憑證才能驗證。 請安裝用戶端憑證。 如需用戶端憑證的相關資訊，請參閱[點對站的用戶端憑證](vpn-gateway-certificates-point-to-site.md)。 若要安裝所產生的憑證，請參閱[在 Windows 用戶端上安裝憑證](point-to-site-how-to-vpn-client-install-azure-cert.md)。
3. 在用戶端電腦上，瀏覽至 [網路設定]，然後按一下 [VPN]。 VPN 連線會顯示其連線的虛擬網路名稱。

#### <a name="certmaccli"></a>Mac (OS X) VPN 用戶端設定

您必須為每個連線到 Azure VNet 的 Mac 裝置建立個別的設定檔。 這是因為這些裝置需要在設定檔中指定可供驗證的使用者憑證。 **Generic** 資料夾含有建立設定檔所需的所有資訊。

  * **VpnSettings.xml** 包含重要的設定，例如伺服器位址和通道類型。
  * **VpnServerRoot.cer** 包含在 P2S 連線設定期間驗證 VPN 閘道所需的根憑證。
  * **RadiusServerRoot.cer** 包含在驗證期間驗證 RADIUS 伺服器所需的根憑證。

請使用下列步驟來設定 Mac 上原生 VPN 用戶端的憑證驗證：

1. 將 **VpnServerRoot** 和 **RadiusServerRoot** 根憑證匯入至您的 Mac。 若要這麼做，請將該檔案複製到 Mac 上並對該檔案按兩下。  
按一下 [新增] 進行匯入。

  *新增 VpnServerRoot*

  ![新增憑證](./media/point-to-site-vpn-client-configuration-radius/addcert.png)

  *新增 RadiusServerRoot*

  ![新增憑證](./media/point-to-site-vpn-client-configuration-radius/radiusrootcert.png)
2. 每個用戶端都需要用戶端憑證才能驗證。 請在用戶端裝置上安裝用戶端憑證。
3. 開啟 [網路喜好設定] 底下的 [網路] 對話方塊，然後按一下 [+] 建立適用於 Azure VNet P2S 連線的新 VPN 用戶端連線設定檔。

  [介面] 值是 [VPN]，[VPN 類型] 值則是 [IKEv2]。 在 [服務名稱] 欄位中指定設定檔名稱，然後按一下 [建立] 以建立 VPN 用戶端連線設定檔。

  ![網路](./media/point-to-site-vpn-client-configuration-radius/network.png)
4. 從 **Generic** 資料夾的 **VpnSettings.xml** 檔案複製 **VpnServer** 標記值。 將此值貼到設定檔的 [伺服器位址] 和 [遠端識別碼] 欄位。 將 [地區設定識別碼] 欄位保留空白。

  ![伺服器資訊](./media/point-to-site-vpn-client-configuration-radius/servertag.png)
5. 按一下 [驗證設定]，然後選取 [憑證]。 

  ![驗證設定](./media/point-to-site-vpn-client-configuration-radius/certoption.png)
6. 按一下 [選取...] 以選擇您要用來驗證的憑證。

  ![憑證](./media/point-to-site-vpn-client-configuration-radius/certificate.png)
7. [選擇身分識別] 會顯示一份憑證清單供您選擇。 選取適當憑證，然後按一下 [繼續]。

  ![身分識別](./media/point-to-site-vpn-client-configuration-radius/identity.png)
8. 在 [地區設定識別碼] 欄位中指定憑證名稱 (來自步驟 6)。 在此範例中，名稱是 "ikev2Client.com"。 然後，按一下 [套用] 按鈕以儲存變更。

  ![apply](./media/point-to-site-vpn-client-configuration-radius/applyconnect.png)
9. 在 [網路] 對話方塊中按一下 [套用] 以儲存所有變更。 然後，按一下 [連線] 以啟動 Azure VNet 的 P2S 連線。

## <a name="otherauth"></a>與其他驗證類型或通訊協定搭配使用

若要使用不同的驗證類型 (例如 OTP，而不是使用者名稱/密碼或憑證)，或使用不同的驗證通訊協定 (例如 PEAP-MSCHAPv2，而不是 EAP-MSCHAPv2)，您必須建立自己的 VPN 用戶端組態設定檔。 若要建立設定檔，您需要虛擬網路閘道 IP 位址、通道類型以及分割通道路由等資訊。 您可以使用下列步驟來取得此資訊：

1. 使用 'Get-AzureRmVpnClientConfiguration' Cmdlet 來產生 EapMSChapv2 的 VPN 用戶端組態。 如需相關指示，請參閱本文的[這一節](#ccradius)。

2. 解壓縮 VpnClientConfiguration.zip 檔案，並找出 GenenericDevice 資料夾。 忽略包含 64 位元和 32 位元架構之 Windows 安裝程式的資料夾。
 
3. GenenericDevice 資料夾包含名為 VpnSettings 的 XML 檔案。 這個檔案包含所有必要資訊。

  * VpnServer - Azure VPN 閘道的 FQDN。 這是用戶端所連線至的位址。
  * VpnType - 您用來連線的通道類型。
  * 路由 - 您必須在設定檔中設定的路由，所以只有 Azure VNet 繫結流量會透過 P2S 通道傳送。
  * GenenericDevice 資料夾也包含一個稱為 'VpnServerRoot' 的 .cer 檔案。 此檔案包含在 P2S 連線設定期間驗證 Azure VPN 閘道所需的根憑證。 在將連線到 Azure VNet 的所有裝置上安裝憑證。

## <a name="next-steps"></a>後續步驟

回到本文以[完成 P2S 設定](point-to-site-how-to-radius-ps.md)。

如需 P2S 疑難排解詳細資訊，請參閱[針對 Azure 點對站連線進行疑難排解](vpn-gateway-troubleshoot-vpn-point-to-site-connection-problems.md)。