---
title: "PowerShell を使用して Azure DevTest Labs に VHD ファイルをアップロードする | Microsoft Docs"
description: "PowerShell を使用してラボのストレージ アカウントに VHD ファイルをアップロードします"
services: devtest-lab,virtual-machines
documentationcenter: na
author: craigcaseyMSFT
manager: douge
editor: 
ms.assetid: 
ms.service: devtest-lab
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/10/2017
ms.author: v-craic
ms.openlocfilehash: c940f5646bc7845db7b774d7ada40b8105534d69
ms.sourcegitcommit: d87b039e13a5f8df1ee9d82a727e6bc04715c341
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/21/2018
---
# <a name="upload-vhd-file-to-labs-storage-account-using-powershell"></a>PowerShell を使用してラボのストレージ アカウントに VHD ファイルをアップロードします

[!INCLUDE [devtest-lab-upload-vhd-selector](../../includes/devtest-lab-upload-vhd-selector.md)]

Azure DevTest Labs では、VHD ファイルを使用してカスタム イメージを作成でき、そのイメージを使用して仮想マシンをプロビジョニングできます。 次の手順は、PowerShell を使用して、VHD ファイルをラボのストレージ アカウントにアップロードする方法を示しています。 VHD ファイルをアップロードしたら、[「次のステップ」セクション](#next-steps)の記事に従って、アップロードした VHD ファイルからカスタム イメージを作成できます。 Azure でのディスクと VHD の詳細については、「[仮想マシン用のディスクと VHD について](../virtual-machines/linux/about-disks-and-vhds.md)」を参照してください。

## <a name="step-by-step-instructions"></a>詳細な手順

次の手順は、PowerShell を使用して VHD ファイルを Azure DevTest Labs にアップロードする方法を示しています。 

1. [Azure ポータル](http://go.microsoft.com/fwlink/p/?LinkID=525040)にサインインします。

1. **[すべてのサービス]** を選択し、一覧の **[DevTest Labs]** を選択します。

1. ラボの一覧で目的のラボを選択します。  

1. ラボのブレードで、 **[構成]**を選択します。 

1. ラボの **[構成]** ブレードで、**[カスタム イメージ (VHD)]** を選択します。

1. **[カスタム イメージ]** ブレードで、**[+追加]** を選択します。 

1. **[カスタム イメージ]** ブレードで、**[VHD]** を選択します。

1. **[VHD]** ブレードで、**[Upload a VHD file using PowerShell (PowerShell を使用して VHD ファイルをアップロードする)]** を選択します。

    ![PowerShell を使用した VHD のアップロード](./media/devtest-lab-upload-vhd-using-powershell/upload-image-using-psh.png)

1. **[PowerShell を使用したイメージのアップロード]** ブレードで、生成された PowerShell スクリプトをテキスト エディターにコピーします。

1. **Add-AzureVhd** コマンドレットの **LocalFilePath** パラメーターを、アップロードする VHD ファイルの場所を指すように変更します。

1. PowerShell プロンプトで、**Add-AzureVhd**コマンドレットを実行します (変更した **LocalFilePath** パラメーターを使用します)。

> [!WARNING] 
> 
> VHD ファイルのアップロードは、VHD ファイルのサイズと接続速度によっては、時間がかかる場合があります。

## <a name="next-steps"></a>次の手順

- [Azure Portal を使用して VHD ファイルから Azure DevTest Labs にカスタム イメージを作成する](devtest-lab-create-template.md)
- [PowerShell を使用して VHD ファイルから Azure DevTest Labs にカスタム イメージを作成する](devtest-lab-create-custom-image-from-vhd-using-powershell.md)
