---
title: "Linux 上的 Azure App Service 支援 SSH | Microsoft Docs"
description: "了解如何使用 SSH 搭配 Linux 上的 Azure App Service。"
keywords: "azure app service, web 應用程式, linux, oss"
services: app-service
documentationcenter: 
author: wesmc7777
manager: cfowler
editor: 
ms.assetid: 66f9988f-8ffa-414a-9137-3a9b15a5573c
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/25/2017
ms.author: wesmc
ms.openlocfilehash: 905c257ab40057f05081e54e8680bd818023d886
ms.sourcegitcommit: 088a8788d69a63a8e1333ad272d4a299cb19316e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/27/2018
---
# <a name="ssh-support-for-azure-app-service-on-linux"></a>Linux 上的 Azure App Service 支援 SSH

[安全殼層 (SSH)](https://en.wikipedia.org/wiki/Secure_Shell) 是密碼編譯網路通訊協定，可保護網絡服務的使用安全。 它最常用於從命令列遠端安全地登入系統，以及從遠端執行系統管理命令。

Linux 上的 App Service 會利用每個用於新 Web 應用程式的「執行階段堆疊」的內建 Docker 映像，來提供應用程式容器內的 SSH 支援。

![執行階段堆疊](./media/app-service-linux-ssh-support/app-service-linux-runtime-stack.png)

您也可以使用 SSH 搭配自訂 Docker 映像，方法是將 SSH 伺服器納入映像，並如本文中所述將它進行設定。

## <a name="making-a-client-connection"></a>建立用戶端連線

若要建立 SSH 用戶端連線，必須先啟動主要網站。

使用下列格式，將 Web 應用程式的原始檔控制管理 (SCM) 端點貼到您的瀏覽器︰

```bash
https://<your sitename>.scm.azurewebsites.net/webssh/host
```

如果您尚未經過驗證，必須向您的 Azure 訂用帳戶進行驗證才能連線。

![SSH 連線](./media/app-service-linux-ssh-support/app-service-linux-ssh-connection.png)

## <a name="ssh-support-with-custom-docker-images"></a>SSH 支援自訂 Docker 映像

為使自訂 Docker 映像支援容器與 Azure 入口網站之用戶端的 SSH 通訊，請針對 Docker 映像執行下列步驟。

這些步驟會顯示在 Azure App Service 存放庫中，作為[範例](https://github.com/Azure-App-Service/node/blob/master/6.9.3/)。

1. 將 [`RUN`指示](https://docs.docker.com/engine/reference/builder/#run)中的 `openssh-server` 安裝納入映像的 Dockerfile，並將根帳戶的密碼設為 `"Docker!"`。

    > [!NOTE]
    > 此設定不允許容器的外部連線。 只可透過使用發佈認證進行驗證的 Kudu / SCM 網站存取 SSH。

    ```docker
    # ------------------------
    # SSH Server support
    # ------------------------
    RUN apt-get update \
        && apt-get install -y --no-install-recommends openssh-server \
        && echo "root:Docker!" | chpasswd
    ```

1. 將 [`COPY` 指示](https://docs.docker.com/engine/reference/builder/#copy)新增至 Dockerfile，以將 [sshd_config](http://man.openbsd.org/sshd_config) 檔案複製到 */etc/ssh/* 目錄。 組態檔需根據[這裡](https://github.com/Azure-App-Service/node/blob/master/8.2.1/sshd_config)的 Azure-App-Service GitHub 存放庫中的 sshd_config 檔案。

    > [!NOTE]
    > sshd_config 檔案必須包含下列項目，否則無法連線︰ 
    > * `Ciphers` 必須至少包含下列其中一個：`aes128-cbc,3des-cbc,aes256-cbc`。
    > * `MACs` 必須至少包含下列其中一個：`hmac-sha1,hmac-sha1-96`。

    ```docker
    COPY sshd_config /etc/ssh/
    ```

1. 針對 Dockerfile，請納入 [`EXPOSE` 指示](https://docs.docker.com/engine/reference/builder/#expose) 中的連接埠 2222。 雖然已知根密碼，但無法從網際網路存取連接埠 2222。 它是僅供內部使用的連接埠，只有私人虛擬網路之橋接網路內的容器可以存取。

    ```docker
    EXPOSE 2222 80
    ```

1. 請務必使用殼層指令碼來啟動 SSH 服務 (請參閱位於 [init_container.sh](https://github.com/Azure-App-Service/node/blob/master/6.9.3/startup/init_container.sh) 的範例)。

    ```bash
    #!/bin/bash
    service ssh start
    ```

Dockerfile 會使用 [`ENTRYPOINT` 指示](https://docs.docker.com/engine/reference/builder/#entrypoint)來執行指令碼。

    ```docker
    COPY startup /opt/startup
    ...
    RUN chmod 755 /opt/startup/init_container.sh
    ...
    ENTRYPOINT ["/opt/startup/init_container.sh"]
    ```

## <a name="next-steps"></a>後續步驟

您可以在 [Azure 論壇](https://social.msdn.microsoft.com/forums/azure/home?forum=windowsazurewebsitespreview)張貼問題和疑難。

如需「用於容器的 Web App」的詳細資訊，請參閱：

* [如何針對用於容器的 Web 應用程式使用自訂 Docker 映像](quickstart-docker-go.md)
* [在 Linux 上的 Azure App Service 中使用 .NET Core](quickstart-dotnetcore.md)
* [在 Linux 上的 Azure App Service 中使用 Ruby](quickstart-ruby.md)
* [Azure App Service Web App for Containers 常見問題集](app-service-linux-faq.md)