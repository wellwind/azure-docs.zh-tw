---
title: "可用於 Azure Machine Learning 資料準備之其他來源資料連線的範例 | Microsoft Docs"
description: "本文件提供可用於 Azure Machine Learning 資料準備之來源資料連線的一組範例"
services: machine-learning
author: euangMS
ms.author: euang
manager: lanceo
ms.reviewer: garyericson, jasonwhowell, mldocs
ms.service: machine-learning
ms.workload: data-services
ms.custom: 
ms.devlang: 
ms.topic: article
ms.date: 02/01/2018
ms.openlocfilehash: 66c356d6d42254e7443b645bff3393daca67012b
ms.sourcegitcommit: d87b039e13a5f8df1ee9d82a727e6bc04715c341
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/21/2018
---
# <a name="sample-of-custom-source-connections-python"></a>自訂來源連線的範例 (Python) 
閱讀本附錄之前，請先參閱 [Python 擴充性概觀](data-prep-python-extensibility-overview.md)。

## <a name="load-data-from-dataworld"></a>從 data.world 載入資料

### <a name="prerequisites"></a>先決條件

#### <a name="register-yourself-at-dataworld"></a>在 data.world 自行註冊
您需要來自 data.world 網站的 API 權杖。

#### <a name="install-dataworld-library"></a>安裝 data.world 程式庫

選取 [檔案] > [開啟命令列介面]，以開啟 Azure Machine Learning Workbench 命令列介面。

```console
pip install git+git://github.com/datadotworld/data.world-py.git
```

接著，在提示您提供權杖的命令列上執行 `dw configure`。 當您輸入權杖時，會在您的主目錄中建立 .dw/ 目錄，而且您的權杖將儲存在該目錄中。

```
API token (obtained at: https://data.world/settings/advanced): <enter API token here>
```
現在您應該能夠匯入 data.world 程式庫。

#### <a name="load-data-into-data-preparation"></a>將資料載入資料準備

建立轉換資料流程 (指令碼) 轉換。 然後使用下列指令碼從 data.world 載入資料。

```python
#paths = df['Path'].tolist()

import datadotworld as dw

#Load the dataset.
lds = dw.load_dataset('data-society/the-simpsons-by-the-data')

#Load specific data frame from the dataset.
df = lds.dataframes['simpsons_episodes']

```
## <a name="azure-cosmos-db-as-a-data-source-connection"></a>Azure Cosmos DB 作為資料來源連線
如需將 Azure Cosmos DB 作為資料來源連線的範例，請參閱[將 Azure Cosmos DB 載入為來源資料連線](data-prep-load-azure-cosmos-db.md)
