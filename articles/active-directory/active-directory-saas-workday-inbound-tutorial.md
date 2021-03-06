---
title: 'チュートリアル: Workday を構成し、Azure Active Directory を使用した自動ユーザー プロビジョニングに対応させる | Microsoft Docs'
description: Workday を Active Directory および Azure Active Directory の ID データのソースとして使用する方法について説明します。
services: active-directory
author: asmalser-msft
documentationcenter: na
manager: mtillman
ms.assetid: 1a2c375a-1bb1-4a61-8115-5a69972c6ad6
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 01/26/2018
ms.author: asmalser
ms.openlocfilehash: b632622868480638174b616780441e13c16a52c0
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/28/2018
---
# <a name="tutorial-configure-workday-for-automatic-user-provisioning"></a>チュートリアル: Workday を構成し、自動ユーザー プロビジョニングに対応させる

このチュートリアルでは、Workday から Active Directory と Azure Active Directory の両方にユーザーをインポートするために必要な手順と、Workday にいくつかの属性を書き戻すオプションについて説明します。 



## <a name="overview"></a>概要

ユーザー アカウントをプロビジョニングするために、[Azure Active Directory ユーザー プロビジョニング サービス](active-directory-saas-app-provisioning.md)を [Workday Human Resources API](https://community.workday.com/sites/default/files/file-hosting/productionapi/Human_Resources/v21.1/Get_Workers.html) と統合します。 Azure AD はこの接続を使用して、次のユーザー プロビジョニング ワークフローを有効にします。

* **Active Directory へのユーザーのプロビジョニング** - Workday から選択したユーザーのセットを 1 つ以上の Active Directory フォレストに同期させます。 

* **Azure Active Directory へのクラウドのみのユーザーのプロビジョニング** - Active Directory と Azure Active Directory の両方に存在するハイブリッド ユーザーを [AAD Connect](connect/active-directory-aadconnect.md) を使用して後者にプロビジョニングできます。 ただし、クラウドのみのユーザーは、Azure AD ユーザー プロビジョニング サービスを使用すると、Workday から Azure Active Directory に直接プロビジョニングできます。

* **Workday へのメール アドレスの書き戻し** - Azure AD ユーザー プロビジョニング サービスでは、メール アドレスなどの Azure AD ユーザー属性を Workday に書き戻すことができます。

### <a name="what-human-resources-scenarios-does-it-cover"></a>対象になる人事管理シナリオ

Azure AD のユーザー プロビジョニング サービスでサポートされている Workday ユーザー プロビジョニング ワークフローは、次の人事管理および ID ライフサイクル管理シナリオを自動化します。

* **新しい従業員の雇用** - Workday に新しい従業員が追加されると、Active Directory、Azure Active Directory、必要に応じて Office 365 や [Azure AD によってサポートされているその他の SaaS アプリケーション](active-directory-saas-app-provisioning.md)でユーザー アカウントが自動的に作成され、メール アドレスが Workday に書き戻されます。

* **従業員の属性とプロファイルの更新** - Workday で従業員レコード (名前、職名、マネージャなど) が更新されると、Active Directory、Azure Active Directory、必要に応じて Office 365 や [Azure AD によってサポートされているその他の SaaS アプリケーション](active-directory-saas-app-provisioning.md)でユーザー アカウントが自動的に更新されます。

* **従業員の退職** - Workday で従業員が退職状態になると、Active Directory、Azure Active Directory、必要に応じて Office 365 や [Azure AD によってサポートされているその他の SaaS アプリケーション](active-directory-saas-app-provisioning.md)でユーザー アカウントが自動的に無効になります。

* **従業員の再雇用** - Workday で従業員が再雇用されると、Active Directory、Azure Active Directory、必要に応じて Office 365 や [Azure AD によってサポートされているその他の SaaS アプリケーション](active-directory-saas-app-provisioning.md)に以前のアカウントが (設定に応じて) 自動的に再アクティブ化または再プロビジョニングされます。

### <a name="who-is-this-user-provisioning-solution-best-suited-for"></a>このユーザー プロビジョニング ソリューションが最適な場合

この Workday ユーザー プロビジョニング ソリューションは、現在パブリック プレビュー段階です。このソリューションは次のお客様に最適です。

* Workday ユーザー プロビジョニング用に事前に構築されたクラウドベースのソリューションを求めている組織

* Workday から Active Directory、または Azure Active Directory への直接ユーザー プロビジョニングを必要とする組織

* Workday HCM モジュールから取得したデータを使用してユーザーをプロビジョニングする必要がある組織 ([Get_Workers](https://community.workday.com/sites/default/files/file-hosting/productionapi/Human_Resources/v21.1/Get_Workers.html) を参照してください) 

* 1 つ以上の Active Directory フォレスト、ドメイン、および OU と同期するために、Workday HCM モジュールで検出された変更情報のみに基づいてユーザーの登録、移動、削除を行う必要がある組織 ([Get_Workers](https://community.workday.com/sites/default/files/file-hosting/productionapi/Human_Resources/v21.1/Get_Workers.html) を参照してください)

* 電子メールに Office 365 を使用している組織


## <a name="planning-your-solution"></a>ソリューションの設計

Workday の統合を開始する前に、以下の前提条件を確認し、現在の Active Directory のアーキテクチャおよびユーザー プロビジョニング要件を Azure Active Directory が提供するソリューションに適合させる方法について、次のガイダンスをお読みください。

### <a name="prerequisites"></a>前提条件

このチュートリアルで説明するシナリオでは、次の項目があることを前提としています。

* グローバル管理者アクセス権を持つ有効な Azure AD Premium P1 サブスクリプション
* テストと統合のために Workday を実装したテナント
* システム統合ユーザーを作成し、テスト目的でテスト用従業員データを変更する Workday の管理者権限
* Active Directory へのユーザー プロビジョニングの場合、[オンプレミス同期エージェント](https://go.microsoft.com/fwlink/?linkid=847801)をホストするために、Windows Service 2012 以上を実行するドメインに参加しているサーバーが必要です
* Active Directory と Azure AD を同期する [Azure AD Connect](connect/active-directory-aadconnect.md)

### <a name="solution-architecture"></a>ソリューションのアーキテクチャ

Azure AD には、Workday から Active Directory、Azure AD、SaaS アプリなどへのプロビジョニングおよび ID ライフ サイクル管理を解決するための豊富なプロビジョニング コネクタが用意されています。 使用する機能とソリューションの設定方法は、組織の環境と要件によって異なります。 最初の手順として、次のうちどれが組織内に存在し、展開されているかを確認します。

* 使用中の Active Directory フォレスト数
* 使用中の Active Directory ドメイン数
* 使用中の Active Directory の組織単位 (OU) 数
* 使用中の Azure Active Directory のテナント数
* Active Directory と Azure Active Directory の両方にプロビジョニングする必要があるユーザー ("ハイブリッド" ユーザーなど) の有無
* Azure Active Directory にプロビジョニングする必要があるが、Active Directory にはプロビジョニングの必要がないユーザー（"クラウドのみ" ユーザーなど）の有無
* ユーザーのメール アドレスを Workday に書き戻す必要の有無


これらの質問に答えたあと、以下のガイダンスに従って、Workday プロビジョニングの展開を計画します。

#### <a name="using-provisioning-connector-apps"></a>プロビジョニング コネクタ アプリの使用

Azure Active Directory では、Workday および他の数多くの SaaS アプリケーション用の統合済みプロビジョニング コネクタをサポートしています。 

単一のプロビジョニング コネクタは、単一のソース システムの API とインターフェイスし、単一のターゲット システムにデータをプロビジョニングするのに役立ちます。 Azure AD がサポートするほとんどのプロビジョニング コネクタは、単一のソースとターゲット システム (Azure AD to ServiceNow など) 向けで、Azure AD アプリケーション ギャラリーから該当のアプリケーション (ServiceNow など) を追加することでセットアップできます。 

Azure AD のプロビジョニング コネクタ インスタンスとアプリ インスタンスの間には 1 対 1 の関係があります。

| ソース システム | ターゲット システム |
| ---------- | ---------- | 
| Azure AD テナント | SaaS アプリケーション |


ただし、Workday と Active Directory を使用する場合は、複数のソースとターゲットのシステムが検討対象となります。

| ソース システム | ターゲット システム | メモ |
| ---------- | ---------- | ---------- |
| Workday | Active Directory フォレスト | 各フォレストは、個別のターゲット システムとして扱われます |
| Workday | Azure AD テナント | クラウドのみユーザー用に必要 |
| Active Directory フォレスト | Azure AD テナント | 現在、このフローは AAD Connect によって処理されます |
| Azure AD テナント | Workday | メール アドレスの書き戻し用 |

これらの複数のソースおよびターゲット システムに対応する複数のワークフローを容易にするため、Azure AD には Azure AD アプリケーション ギャラリーから追加可能な複数のプロビジョニング コネクタ アプリケーションが用意されています。

![AAD アプリ ギャラリー](./media/active-directory-saas-workday-inbound-tutorial/WD_Gallery.PNG)

* **Workday to Active Directory Provisioning** - このアプリケーションは、Workday から単一の Active Directory フォレストへのユーザー アカウントのプロビジョニングを容易にします。 複数のフォレストがある場合、Azure AD アプリケーションギャラリーのこのアプリケーションのインスタンスをプロビジョニングの必要がある各 Active Directory フォレストに 1 つずつ追加できます。

* **Workday to Azure AD Provisioning** - AAD Connect は、Active Directory ユーザーを Azure Active Directory に同期するのに必要なツールですが、このアプリはクラウドのみユーザーを Workday から単一の Azure Active Directory テナントへプロビジョニングするのを容易にします。

* **Workday Writeback** - このアプリケーションは、ユーザーのメール アドレスを Azure Active Directory から Workday に書き戻すのを容易にします。

> [!TIP]
> 通常の "Workday" アプリケーションは、Workday と Azure Active Directory の間でシングル サインオンを設定するために使用されます。 

これらの特別なプロビジョニング コネクタ アプリをセットアップして構成する方法が、このチュートリアルの残りのセクションの主題です。 構成するアプリケーションは、プロビジョニングが必要な対象システム、および環境内の Active Directory フォレストと Azure AD テナントの数によって異なります。

![概要](./media/active-directory-saas-workday-inbound-tutorial/WD_Overview.PNG)

## <a name="configure-a-system-integration-user-in-workday"></a>Workday でのシステム統合ユーザーの構成
すべての Workday プロビジョニング コネクタに共通する要件は、Workday Human Resources API に接続するために、Workday システム統合アカウントの資格情報を必要とすることです。 このセクションでは、Workday でシステム インテグレーター アカウントを作成する方法について説明します。

> [!NOTE]
> この手順を省略し、代わりに Workday グローバル管理者アカウントをシステム統合アカウントとして使用することもできます。 これはデモでは問題なく動作するかもしれませんが、本番環境での展開にはお勧めできません。

### <a name="create-an-integration-system-user"></a>統合システム ユーザーの作成

**統合システム ユーザーを作成するには、次の手順に従います。**

1. 管理者アカウントを使用して、Workday テナントにサインインします。 **Workday Workbench** で、検索ボックスに「ユーザーの作成」と入力し、**[Create Integration System User (統合システム ユーザーの作成)]** をクリックします。 
   
    ![ユーザーの作成](./media/active-directory-saas-workday-inbound-tutorial/IC750979.png "ユーザーの作成")
2. 新しい統合システム ユーザーのユーザー名とパスワードを指定して、**統合システム ユーザーの作成**を完了します。  
 * このユーザーはプログラムを使用してログオンするため、**[次回のサインイン時に新しいパスワードを要求する]** オプションはオフのままにしておきます。 
 * **[セッション タイムアウト (分)]** は既定値の 0 のままにしておきます。これにより、ユーザーのセッションが有効期限前にタイムアウトするのを防ぎます。 
   
    ![統合システム ユーザーの作成](./media/active-directory-saas-workday-inbound-tutorial/IC750980.png "統合システム ユーザーの作成")

### <a name="create-a-security-group"></a>セキュリティ グループの作成
制約のない統合システム セキュリティ グループを作成し、それにユーザーを割り当てる必要があります。

**セキュリティ グループを作成するには、次の手順に従います。**

1. 検索ボックスに「セキュリティ グループの作成」と入力し、 **[Create Security Group (セキュリティ グループの作成)]** をクリックします。 
   
    ![CreateSecurity グループ](./media/active-directory-saas-workday-inbound-tutorial/IC750981.png "CreateSecurity グループ")
2. **[セキュリティ グループの作成]** タスクを完了します。  
3. **[Type of Tenanted Security Group]\(テナント セキュリティ グループの種類\)** ドロップダウンから **[Integration System Security Group (Unconstrained)]\(統合システム セキュリティ グループ (制約なし)\)** を選択します。
4. メンバーが明示的に追加されるセキュリティ グループを作成します。 
   
    ![CreateSecurity グループ](./media/active-directory-saas-workday-inbound-tutorial/IC750982.png "CreateSecurity グループ")

### <a name="assign-the-integration-system-user-to-the-security-group"></a>セキュリティ グループへの統合システム ユーザーの割り当て

**統合システム ユーザーを割り当てるには、次の手順に従います。**

1. 検索ボックスに「セキュリティ グループの編集」と入力し、 **[Edit Security Group (セキュリティ グループの編集)]** をクリックします。 
   
    ![セキュリティ グループの編集](./media/active-directory-saas-workday-inbound-tutorial/IC750983.png "セキュリティ グループの編集")
2. 新しい統合セキュリティ グループを名前で検索し、選択します。 
   
    ![セキュリティ グループの編集](./media/active-directory-saas-workday-inbound-tutorial/IC750984.png "セキュリティ グループの編集")
3. 新しいセキュリティ グループに新しい統合システム ユーザーを追加します。 
   
    ![システム セキュリティ グループ](./media/active-directory-saas-workday-inbound-tutorial/IC750985.png "システム セキュリティ グループ")  

### <a name="configure-security-group-options"></a>セキュリティ グループ オプションの構成
この手順では、worker データのドメイン セキュリティ ポリシーのアクセス許可をセキュリティ グループに付与します。

**セキュリティ グループ オプションを構成するには、次の手順に従います。**

1. 検索ボックスに**ドメインのセキュリティ ポリシー**を入力し、**[機能領域のドメイン セキュリティ ポリシー]** リンクをクリックします。  
   
    ![ドメイン セキュリティ ポリシー](./media/active-directory-saas-workday-inbound-tutorial/IC750986.png "ドメイン セキュリティ ポリシー")  
2. システムを検索し、 **システム** の機能領域を選択します。  Click **OK**.  
   
    ![ドメイン セキュリティ ポリシー](./media/active-directory-saas-workday-inbound-tutorial/IC750987.png "ドメイン セキュリティ ポリシー")  
3. システム機能領域のセキュリティ ポリシーの一覧で、**[セキュリティ管理]** を展開し、ドメイン セキュリティ ポリシーの **[External Account Provisioning]** を選択します。  
   
    ![ドメイン セキュリティ ポリシー](./media/active-directory-saas-workday-inbound-tutorial/IC750988.png "ドメイン セキュリティ ポリシー")  
4. **[アクセス許可の編集]** をクリックし、**[アクセス許可の編集]** ダイアログ ページで、**Get** と **Put** の統合アクセス権限を持つセキュリティ グループの一覧に新しいセキュリティ グループを追加します。 
   
    ![アクセス許可の編集](./media/active-directory-saas-workday-inbound-tutorial/IC750989.png "アクセス許可の編集")  
    
5. これらの残りのセキュリティ ポリシーごとに上記の手順 1 から 4 を繰り返します。

| 操作 | ドメイン セキュリティ ポリシー |
| ---------- | ---------- | 
| Get と Put | Worker Data: Public Worker Reports |
| Get と Put | Worker Data: Work Contact Information |
| 取得 | Worker Data: All Positions |
| 取得 | Worker Data: Current Staffing Information |
| 取得 | Worker Data: Business Title on Worker Profile |
   
    
### <a name="activate-security-policy-changes"></a>セキュリティ ポリシーの変更のアクティブ化

**セキュリティ ポリシーの変更をアクティブにするには、次の手順に従います。**

1. 検索ボックスに「アクティブ化」と入力し、**[保留中のセキュリティ ポリシーの変更をアクティブ化]** リンクをクリックします。 
   
    ![アクティブ化](./media/active-directory-saas-workday-inbound-tutorial/IC750992.png "アクティブ化") 
2. 監査用のコメントを入力し、 **[OK]** をクリックして、[Activate Pending Security Policy Changes (保留中のセキュリティ ポリシーの変更のアクティブ化)] 作業を開始します。 
   
    ![保留中のセキュリティのアクティブ化](./media/active-directory-saas-workday-inbound-tutorial/IC750993.png "保留中のセキュリティのアクティブ化")   
3. **[確認]** チェック ボックスをオンにして、**[OK]** をクリックし、次の画面で作業を完了します。 
   
    ![保留中のセキュリティのアクティブ化](./media/active-directory-saas-workday-inbound-tutorial/IC750994.png "保留中のセキュリティのアクティブ化")  

## <a name="configuring-user-provisioning-from-workday-to-active-directory"></a>Workday から Active Directory へのユーザー プロビジョニングの構成
次の手順に従い、Workday からプロビジョニングを必要とする各 Active Directory フォレストへのユーザー アカウントのプロビジョニングを構成します。

### <a name="planning"></a>計画

Active Directory フォレストへのユーザー プロビジョニングを構成する前に、以下の事項を検討してください。 これらの事項に対する答えで、スコープ フィルターと属性マッピングをどのように設定する必要があるかが決まります。 

* **Workday のユーザーをこの Active Directory フォレストにプロビジョニングする必要はありますか。**

   * *例: Workday の "Company" 属性に値 "Contoso" が含まれ、"Worker_Type" 属性に "Regular" が含まれているユーザー*

* **ユーザーはどのように組織単位 (OU) にルーティングされますか。**

   * *例: Workday の "Municipality" 属性と "Country_Region_Reference" 属性の定義に従って、オフィスの位置に対応する OU にルーティングされるユーザー*

* **次の属性は Active Directory にどのように埋め込む必要がありますか。**

   * 共通名 (cn)
      * *例: 人事によって設定された Workday の User_ID 値を使用します*
      
   * 従業員 ID (employeeId)
      * *例: Workday の Worker_ID 値を使用します*
      
   * SAM アカウント名 (sAMAccountName)
      * *例: Azure AD プロビジョニング式でフィルター処理された Workday の User_ID 値を使用して、不正な文字を削除します*
      
   * ユーザー プリンシパル名 (userPrincipalName)
      * *例: Azure AD プロビジョニング式で Workday の User_ID 値を使用して、ドメイン名を付加します*

* **Workday と Active Directory の間でどのようにユーザーを対応付ける必要がありますか。**

  * *例: 特定の Workday "Worker_ID" 値を持つユーザーは、"employeeID" が同じ値の Active Directory ユーザーと対応付けます。Active Directory で Worker_ID の値が見つからない場合は、新しいユーザーを作成します。*
  
* **Active Directory フォレストには、一致するロジックが動作するために必要なユーザー ID が既に含まれていますか。**

  * *例: Workday を新規にデプロイする場合は、一致するロジックをできるだけ単純にするために、Active Directory に正しい Workday の Worker_ID 値 (または選択した一意の ID 値) をあらかじめ入力することを強く推奨します。*
    
    
### <a name="part-1-adding-the-provisioning-connector-app-and-creating-the-connection-to-workday"></a>パート 1：プロビジョニング コネクタ アプリの追加と Workday への接続の作成

**Workday を Active Directory プロビジョニングに構成するには、**

1.  <https://portal.azure.com> に移動します

2.  左のナビゲーション バーで、**[Azure Active Directory]** を選択します

3.  **[エンタープライズ アプリケーション]**、**[すべてのアプリケーション]** の順に選択します。

4.  **[アプリケーションの追加]** を選択し、**[すべて]** のカテゴリを選択します。

5.  「**Workday Provisioning to Active Directory**」を検索し、ギャラリーからそのアプリを追加します。

6.  アプリが追加され、アプリの詳細画面が表示されたら、**[プロビジョニング]** を選択します

7.  **[プロビジョニング** **モード]** を **[自動]** に設定します

8.  以下のように **[管理者の資格情報]** セクションを完了します。

   * **管理者ユーザー名** – Workday 統合システム アカウントのユーザー名にテナント ドメイン名を追加して入力します。 **次のように表示されます: username@contoso4**

   * **管理者パスワード –** Workday 統合システム アカウントのパスワードを入力します

   * **テナント URL –** テナントの Workday Web サービス エンドポイントへの URL を入力します。 これは https://wd3-impl-services1.workday.com/ccx/service/contoso4 のようになります。contoso4 は適切テナント名に置き換え、wd3-impl は適切な環境文字列に置き換えます。

   * **Active Directory フォレスト -** Get ADForest powershell コマンドレットによって返された Active Directory フォレストの "名前"。 これは通常、*contoso.com* のような文字列です。

   * **Active Directory コンテナー -** AD フォレストのすべてのユーザーが含まれるコンテナー文字列を入力します。 例: *OU=Standard Users,OU=Users,DC=contoso,DC=test*

   * **メール通知 –** メール アドレスを入力し、[send email if failure occurs (失敗した場合にメールを送信する)] チェック ボックスをオンにします。

   * **[接続のテスト]** ボタンをクリックします。 接続テストが成功した場合、上部の **[保存]** ボタンをクリックします。 失敗した場合は、Workday 資格情報が Workday で有効であることを再度確認します。 

![Azure ポータル](./media/active-directory-saas-workday-inbound-tutorial/WD_1.PNG)

### <a name="part-2-configure-attribute-mappings"></a>パート 2: 属性マッピングの構成 

このセクションでは、ユーザー データが Workday から Active Directory に移動する方法を構成します。

1.  **[マッピング]** の [プロビジョニング] タブで、**[Synchronize Workday Workers to OnPremises (Workday の社員をオンプレミスに同期する)]** をクリックします。

2.  **[ソース オブジェクト スコープ]** フィールドでは、属性ベースのフィルター セットを定義して、AD へのプロビジョニングの対象にする Workday のユーザー セットを選択できます。 既定のスコープは、"Workday のすべてのユーザー" です。 フィルターの例:

   * 例: 1000000 から 2000000 までの Worker ID を持つユーザーにスコープを設定

      * 属性: WorkerID

      * 演算子: REGEX Match

      * 値: (1[0-9][0-9][0-9][0-9][0-9][0-9])

   * 例: 臨時社員ではなく、従業員のみ 

      * 属性: EmployeeID

      * 演算子: IS NOT NULL

3.  **[対象オブジェクトのアクション]** フィールドでは、Active Directory で実行可能なアクションをグローバルにフィルタリングできます。 **作成**と**更新**が最も一般的です。

4.  **[属性マッピング]** セクションでは、個別の Workday 属性を Active Directory の属性にマッピングする方法を定義できます。

5. 既存の属性マッピングをクリックして更新するか、または画面の下部にある **[新しいマッピングの追加]** をクリックして、新しいマッピングを追加します。 個々の属性マッピングは、次のプロパティをサポートしています。

      * **マッピングの種類**

         * **ダイレクト** – 変更なしで Workday 属性の値を AD 属性に書き込みます

         * **定数** - 静的な定数文字列の値を AD 属性に書き込みます

         * **式** – 1 つ以上の Workday 属性に基づいて、AD 属性にカスタム値を書き込むことができます。 [詳細については、式に関するこの記事を参照してください](active-directory-saas-writing-expressions-for-attribute-mappings.md)。

      * **ソース属性** - Workday のユーザー属性。 探している属性が存在しない場合は、「[Workday のユーザー属性リストをカスタマイズする](#customizing-the-list-of-workday-user-attributes)」を参照してください。

      * **既定値** – 省略可能。 ソース属性に空の値がある場合、マッピングではこの値が代わりに書き込まれます。
            最も一般的な構成では、これを空白のままにします。

      * **ターゲット属性**: Active Directory のユーザー属性。

      * **この属性を使用してオブジェクトを照合する** –このマッピングを使用して、Workday と Active Directory 間でユーザーを一意に識別するかどうかを示します。 これは、通常、Workday の Worker ID フィールドで設定され、一般的に Active Directory の従業員 ID 属性のいずれかにマッピングされます。

      * **照合の優先順位** – 一致させる属性を複数設定できます。 複数の場合は、このフィールドで定義された順序で評価されます。 1 件でも一致が見つかると、一致する属性の評価はそれ以上行われません。

      * **このマッピングを適用する**
       
         * **常に** – このマッピングをユーザーの作成と更新の両方のアクションに適用します

         * **作成中のみ** - このマッピングをユーザーの作成アクションのみに適用します

6. マッピングを保存するには、[属性マッピング] セクションの上部にある **[保存]** をクリックします。

![Azure ポータル](./media/active-directory-saas-workday-inbound-tutorial/WD_2.PNG)

**以下にいくつか示すのは、Workday と Active Directory との間の属性マッピングの例と、一般的に使用される式です**

-   parentDistinguishedName 属性にマップされる式を使用すると、1 つ以上の Workday ソース属性に基づいて異なる OU にユーザーをプロビジョニングできます。 この例では、住所の市区町村に基づいて、異なる OU にユーザーを配置しています。

-   Active Directory の userPrincipalName 属性は、Workday のユーザー ID とドメイン サフィックスを連結して生成されます

-   [式の記述に関するドキュメントについては、こちらを参照してください](active-directory-saas-writing-expressions-for-attribute-mappings.md)。 特殊文字を削除する方法の例についても紹介されています。

  
| WORKDAY 属性 | ACTIVE DIRECTORY 属性 |  ID 一致の有無 | 作成/更新 |
| ---------- | ---------- | ---------- | ---------- |
| **WorkerID**  |  EmployeeID | **はい** | 作成時のみ書き込まれる | 
| **UserID**    |  cn    |   |   作成時のみ書き込まれる |
| **Join("@",[UserID], "contoso.com")**   | userPrincipalName     |     | 作成時のみ書き込まれる 
| **Replace(Mid(Replace(\[UserID\], , "(\[\\\\/\\\\\\\\\\\\\[\\\\\]\\\\:\\\\;\\\\|\\\\=\\\\,\\\\+\\\\\*\\\\?\\\\&lt;\\\\&gt;\])", , "", , ), 1, 20), , "([\\\\.)\*\$](file:///\\.)*$)", , "", , )**      |    sAMAccountName            |     |         作成時のみ書き込まれる |
| **Switch(\[Active\], , "0", "True", "1",)** |  accountDisabled      |     | 作成時 + 更新時 |
| **FirstName**   | givenName       |     |    作成時 + 更新時 |
| **LastName**   |   sn   |     |  作成時 + 更新時 |
| **PreferredNameData**  |  displayName |     |   作成時 + 更新時 |
| **Company**         | company   |     |  作成時 + 更新時 |
| **SupervisoryOrganization**  | department  |     |  作成時 + 更新時 |
| **ManagerReference**   | manager  |     |  作成時 + 更新時 |
| **BusinessTitle**   |  title     |     |  作成時 + 更新時 | 
| **AddressLineData**    |  streetAddress  |     |   作成時 + 更新時 |
| **Municipality**   |   l   |     | 作成時 + 更新時 |
| **CountryReferenceTwoLetter**      |   co |     |   作成時 + 更新時 |
| **CountryReferenceTwoLetter**    |  c  |     |         作成時 + 更新時 |
| **CountryRegionReference** |  st     |     | 作成時 + 更新時 |
| **WorkSpaceReference** | physicalDeliveryOfficeName    |     |  作成時 + 更新時 |
| **PostalCode**  |   postalCode  |     | 作成時 + 更新時 |
| **PrimaryWorkTelephone**  |  telephoneNumber   |     | 作成時 + 更新時 |
| **Fax**      | facsimileTelephoneNumber     |     |    作成時 + 更新時 |
| **Mobile**  |    mobile       |     |       作成時 + 更新時 |
| **LocalReference** |  preferredLanguage  |     |  作成時 + 更新時 |                                               
| **Switch(\[Municipality\], "OU=Standard Users,OU=Users,OU=Default,OU=Locations,DC=contoso,DC=com", "Dallas", "OU=Standard Users,OU=Users,OU=Dallas,OU=Locations,DC=contoso,DC=com", "Austin", "OU=Standard Users,OU=Users,OU=Austin,OU=Locations,DC=contoso,DC=com", "Seattle", "OU=Standard Users,OU=Users,OU=Seattle,OU=Locations,DC=contoso,DC=com", “London", "OU=Standard Users,OU=Users,OU=London,OU=Locations,DC=contoso,DC=com")**  | parentDistinguishedName     |     |  作成時 + 更新時 |
  
### <a name="part-3-configure-the-on-premises-synchronization-agent"></a>パート 3: オンプレミスの同期エージェントの構成

オンプレミスの Active Directory にプロビジョニングするには、該当の Active Directory フォレストのドメイン参加サーバーにエージェントがインストールされている必要があります。 この手順を完了するには、ドメイン管理者 (またはエンタープライズ管理者) の資格情報が必要です。

**[ここからオンプレミスの同期エージェントをダウンロードできます](https://go.microsoft.com/fwlink/?linkid=847801)**

エージェントをインストールしたら、以下の Powershell コマンドを実行して、環境に合わせてエージェントを構成します。

**コマンド #1**

> cd "C:\Program Files\Microsoft Azure AD Connect Provisioning Agent\Modules\AADSyncAgent" Agent\\Modules\\AADSyncAgent

> Import-Module "C:\Program Files\Microsoft Azure AD Connect Provisioning Agent\Modules\AADSyncAgent\AADSyncAgent.psd1"

**コマンド #2**

> Add-ADSyncAgentActiveDirectoryConfiguration

* 入力: パート \#2 で入力したように、"ディレクトリ名" に AD フォレスト名を入力します
* 入力: Active Directory フォレストの管理者ユーザー名とパスワード

**コマンド #3**

> Add-ADSyncAgentAzureActiveDirectoryConfiguration

* 入力: Azure AD テナントのグローバル管理者ユーザー名とパスワード

>[!IMPORTANT]
>現在既知の問題があり、グローバル管理者がカスタム ドメイン (例: admin@contoso.com) を使用すると、その資格情報が動作しません。 この問題を回避するには、onmicrosoft.com ドメインでグローバル管理者アカウント (例: admin@contoso.onmicrosoft.com) を作成し、使用します

>[!IMPORTANT]
>現在既知の問題があります。グローバル管理者が多要素認証を有効にすると、その資格情報が動作しません。 回避策としては、グローバル管理者の多要素認証を無効にします。


**コマンド #4**

> Get-AdSyncAgentProvisioningTasks

* アクション: データが返されたことを確認します。 このコマンドは、Azure AD テナントの Workday プロビジョニング アプリを自動的に検出します。 出力例:

> Name          : My AD Forest
>
> Enabled       : True
>
> DirectoryName : mydomain.contoso.com
>
> Credentialed  : False
>
> Identifier    : WDAYdnAppDelta.c2ef8d247a61499ba8af0a29208fb853.4725aa7b-1103-41e6-8929-75a5471a5203

**コマンド #5**

> Start-AdSyncAgentSynchronization -Automatic

**コマンド #6**

> net stop aadsyncagent

**コマンド #7**

> net start aadsyncagent

>[!TIP]
>同期エージェント サービスは、Powershell の "net"コマンドのほか、**Services.msc** を使用して開始および停止することもできます。 Powershell コマンドを実行しているときにエラーが発生する場合は、**Microsoft Azure AD Connect プロビジョニング エージェント**が **Services.msc** で実行されていることを確認します。

![サービス](./media/active-directory-saas-workday-inbound-tutorial/Services.png)  

**EU 顧客向けの追加構成**

Azure Active Directory テナントが EU データ センターのいずれかにある場合は、次の追加手順に従ってください。

1. **Services.msc** を開いて、**Microsoft Azure AD Connect プロビジョニング エージェント** サービスを停止します。
2. エージェント インストール フォルダー (例: C:\Program files \microsoft Azure AD Connect プロビジョニング エージェント) に移動します。
3. テキスト エディターで **SyncAgnt.exe.config** を開きます。
4. https://manage.hub.syncfabric.windowsazure.com/Management を **https://eu.manage.hub.syncfabric.windowsazure.com/Management** で置き換えます
5. https://provision.hub.syncfabric.windowsazure.com/Provisioning を **https://eu.provision.hub.syncfabric.windowsazure.com/Provisioning** で置き換えます
6. **SyncAgnt.exe.config** ファイルを保存します。
7. **Services.msc** を開いて、**Microsoft Azure AD Connect プロビジョニング エージェント** サービスを開始します。

**エージェントのトラブルシューティング**

エージェントをホストする Windows Server マシンの [Windows イベント ログ](https://technet.microsoft.com/library/cc722404(v=ws.11).aspx)には、エージェントによって実行されるすべての操作のイベントが含まれています。 これらのイベントを表示するには:
    
1. **Eventvwr.msc** を開きます。
2. **[Windows ログ] > [アプリケーション]** の順に選択します。
3. ソース **AADSyncAgent** のログに記録されたすべてのイベントを表示します。 
4. エラーと警告をを確認します。

Powershell コマンドで指定された Active Directory または Azure Active Directory の資格情報にアクセス許可の問題がある場合は、次のようなエラーが表示されます。 
    
![イベント ログ](./media/active-directory-saas-workday-inbound-tutorial/Windows_Event_Logs.png) 


### <a name="part-4-start-the-service"></a>パート 4: サービスの開始
パート 1 ～ 3 が完了したら、Azure Portal に戻ってプロビジョニング サービスを開始できます。

1.  **[プロビジョニング]** タブで、**[プロビジョニングの状態]** を **[ON]** に設定します。

2. **[Save]** をクリックします。

3. これにより初期同期が開始されます。これに要する時間は Workday のユーザー数に応じて異なります。

4. 好きなときに、Azure Portal の **[監査ログ]** タブをチェックして、プロビジョニング サービスで実行されたアクションを確認します。 監査ログには、Workday から読み込まれたユーザーや、その後 Active Directory に追加または更新されたユーザーなど、プロビジョニング サービスによって実行された個々の同期イベントがすべて表示されます。 **[監査ログの読み方の詳細については、プロビジョニング レポート ガイドを参照してください](active-directory-saas-provisioning-reporting.md)**

5.  エージェントをホストする Windows Server マシンの [Windows イベント ログ](https://technet.microsoft.com/library/cc722404(v=ws.11).aspx)で、新しいエラーや警告が発生していないかどうかを確認します。 こうしたイベントを表示するには、サーバーで **Eventvwr.msc** を起動し、**[Windows ログ] > [アプリケーション]** の順に選択します。 プロビジョニングに関連するすべてのメッセージが、ソース **AADSyncAgent** のログに記録されます。 
    

6. プロビジョニングが完了すると、次に示すように、**[プロビジョニング]** タブに監査概要レポートが書き込まれます。

![Azure ポータル](./media/active-directory-saas-workday-inbound-tutorial/WD_3.PNG)


## <a name="configuring-user-provisioning-to-azure-active-directory"></a>Azure Active Directory へのユーザー プロビジョニングの構成
Azure Active Directory へのプロビジョニングの構成方法は、以下の表に示すとおり、プロビジョニングの要件によって異なります。

| シナリオ | 解決策 |
| -------- | -------- |
| **ユーザーは Active Directory と Azure AD にプロビジョニングする必要があります** | **[AAD Connect](connect/active-directory-aadconnect.md)** を使用する |
| **ユーザーは Active Directory にのみプロビジョニングする必要があります** | **[AAD Connect](connect/active-directory-aadconnect.md)** を使用する |
| **ユーザーは Azure AD にのみ (クラウドのみ) プロビジョニングする必要があります** | アプリ ギャラリーの **Workday to Azure Active Directory provisioning** アプリを使用する |

Azure AD Connect の設定手順については、[Azure AD Connect に関するドキュメント](connect/active-directory-aadconnect.md)を参照してください。

次のセクションでは、Workday と Azure AD の間の接続を設定して、クラウドのみのユーザーをプロビジョニングする方法について説明します。

> [!IMPORTANT]
> Azure AD にプロビジョニングする必要があり、オンプレミスの Active Directory にはその必要がないクラウドのみのユーザーが存在する場合は、以下の手順にのみ従ってください。

### <a name="part-1-adding-the-azure-ad-provisioning-connector-app-and-creating-the-connection-to-workday"></a>パート 1: Azure AD プロビジョニング コネクタ アプリケーションの追加と Workday への接続の作成

**Workday を構成して、クラウドのみのユーザーを Azure Active Directory にプロビジョニングするには、**

1.  <https://portal.azure.com> にアクセスします。

2.  左のナビゲーション バーで、**[Azure Active Directory]** を選択します

3.  **[エンタープライズ アプリケーション]**、**[すべてのアプリケーション]** の順に選択します。

4.  **[アプリケーションの追加]** を選択し、**[すべて]** のカテゴリを選択します。

5.  **Workday to Azure AD provisioning** を検索し、ギャラリーからそのアプリを追加します。

6.  アプリが追加され、アプリの詳細画面が表示されたら、**[プロビジョニング]** を選択します

7.  **[プロビジョニング** **モード]** を **[自動]** に設定します

8.  以下のように **[管理者の資格情報]** セクションを完了します。

   * **管理者ユーザー名** – Workday 統合システム アカウントのユーザー名にテナント ドメイン名を追加して入力します。 次のように表示されます: username@contoso4

   * **管理者パスワード –** Workday 統合システム アカウントのパスワードを入力します

   * **テナント URL –** テナントの Workday Web サービス エンドポイントへの URL を入力します。 これは https://wd3-impl-services1.workday.com/ccx/service/contoso4 のようになります。contoso4 は適切テナント名に置き換え、wd3-impl は適切な環境文字列に置き換えます。 この URL が不明の場合は、Workday 統合パートナーまたはサポート担当者に連絡して、使用する正しい URL を確認してください。

   * **メール通知 –** メール アドレスを入力し、[send email if failure occurs (失敗した場合にメールを送信する)] チェック ボックスをオンにします。

   * **[接続のテスト]** ボタンをクリックします。

   * 接続テストが成功した場合、上部の **[保存]** ボタンをクリックします。 失敗した場合は、Workday URL と資格情報が Workday で有効であることを再度確認します。


### <a name="part-2-configure-attribute-mappings"></a>パート 2: 属性マッピングの構成 

このセクションでは、クラウドのみのユーザー データが Workday から Azure Active Directory に移動する方法を構成します。

1.  **[マッピング]** の [プロビジョニング] タブで、**[Synchronize Workers to Azure AD (Workers を Azure AD に同期する)]** をクリックします。

2.   **[ソース オブジェクト スコープ]** フィールドでは、属性ベースのフィルター セットを定義して、Azure AD へのプロビジョニングの対象にするWorkday のユーザー セットを選択できます。 既定のスコープは、"Workday のすべてのユーザー" です。 フィルターの例:

   * 例: 1000000 から 2000000 までの Worker ID を持つユーザーにスコープを設定

      * 属性: WorkerID

      * 演算子: REGEX Match

      * 値: (1[0-9][0-9][0-9][0-9][0-9][0-9])

   * 例: 正規従業員ではなく、臨時社員のみ

      * 属性: ContingentID

      * 演算子: IS NOT NULL

3.  **[対象オブジェクトのアクション]** フィールドでは、Azure AD で実行可能なアクションをグローバルにフィルタリングできます。 **作成**と**更新**が最も一般的です。

4.  **[属性マッピング]** セクションでは、個別の Workday 属性を Active Directory の属性にマッピングする方法を定義できます。

5. 既存の属性マッピングをクリックして更新するか、または画面の下部にある **[新しいマッピングの追加]** をクリックして、新しいマッピングを追加します。 個々の属性マッピングは、次のプロパティをサポートしています。

   * **マッピングの種類**

      * **ダイレクト**– 変更なしで Workday 属性の値を AD 属性に書き込みます

      * **定数** - 静的な定数文字列の値を AD 属性に書き込みます

      * **式** – 1 つ以上の Workday 属性に基づいて、AD 属性にカスタム値を書き込むことができます。 [詳細については、式に関するこの記事を参照してください](active-directory-saas-writing-expressions-for-attribute-mappings.md)。

   * **ソース属性** - Workday のユーザー属性。 探している属性が存在しない場合は、「[Workday のユーザー属性リストをカスタマイズする](#customizing-the-list-of-workday-user-attributes)」を参照してください。

   * **既定値** – 省略可能。 ソース属性に空の値がある場合、マッピングではこの値が代わりに書き込まれます。
            最も一般的な構成では、これを空白のままにします。

   * **ターゲット属性** – Azure AD のユーザーの属性です。

   * **この属性を使用してオブジェクトを照合する** – このマッピングを使用して、Workday と Azure AD 間でユーザーを一意に識別するかどうかを示します。 これは、通常、Workday の Worker ID フィールドで設定され、一般的に Azure AD の従業員 ID 属性 (新規) または拡張属性にマッピングされます。

   * **照合の優先順位** - 一致させる属性を複数設定できます。 複数の場合は、このフィールドで定義された順序で評価されます。 1 件でも一致が見つかると、一致する属性の評価はそれ以上行われません。

   * **このマッピングを適用する**

     * **常に** – このマッピングをユーザーの作成と更新の両方のアクションに適用します

     * **作成中のみ** - このマッピングをユーザーの作成アクションのみに適用します

6. マッピングを保存するには、[属性マッピング] セクションの上部にある **[保存]** をクリックします。

### <a name="part-3-start-the-service"></a>パート 3: サービスの開始
パート 1 ～ 2 の部分が完了したら、プロビジョニング サービスを開始できます。

1.  **[プロビジョニング]** タブで、**[プロビジョニングの状態]** を **[ON]** に設定します。

2. **[Save]** をクリックします。

3. これにより初期同期が開始されます。これに要する時間は Workday のユーザー数に応じて異なります。

4. **[監査ログ]** タブで、個々の同期イベントを表示できます。**[監査ログの読み方の詳細については、プロビジョニング レポート ガイドを参照してください](active-directory-saas-provisioning-reporting.md)**

5. プロビジョニングが完了すると、次に示すように、**[プロビジョニング]** タブに監査概要レポートが書き込まれます。


## <a name="configuring-writeback-of-email-addresses-to-workday"></a>メール アドレスを Workday に書き戻す構成
ユーザーのメールアドレスを Azure Active Directory から Workday に書き戻すように構成するには、次の手順に従ってください。

### <a name="part-1-adding-the-provisioning-connector-app-and-creating-the-connection-to-workday"></a>パート 1：プロビジョニング コネクタ アプリの追加と Workday への接続の作成

**Workday を Active Directory プロビジョニングに構成するには、**

1.  <https://portal.azure.com> に移動します

2.  左のナビゲーション バーで、**[Azure Active Directory]** を選択します

3.  **[エンタープライズ アプリケーション]**、**[すべてのアプリケーション]** の順に選択します。

4.  **[アプリケーションの追加]** を選択し、**[すべて]** のカテゴリを選択します。

5.  **Workday Writeback** を検索し、ギャラリーからそのアプリを追加します。

6.  アプリが追加され、アプリの詳細画面が表示されたら、**[プロビジョニング]** を選択します

7.  **[プロビジョニング** **モード]** を **[自動]** に設定します

8.  以下のように **[管理者の資格情報]** セクションを完了します。

   * **管理者ユーザー名** – Workday 統合システム アカウントのユーザー名にテナント ドメイン名を追加して入力します。 次のように表示されます: username@contoso4

   * **管理者パスワード –** Workday 統合システム アカウントのパスワードを入力します

   * **テナント URL –** テナントの Workday Web サービス エンドポイントへの URL を入力します。 これは https://wd3-impl-services1.workday.com/ccx/service/contoso4 のようになります。contoso4 は適切なテナント名に置き換え、wd3-impl は適切な環境文字列に置き換えます (必要な場合)。

   * **メール通知 –** メール アドレスを入力し、[send email if failure occurs (失敗した場合にメールを送信する)] チェック ボックスをオンにします。

   * **[接続のテスト]** ボタンをクリックします。 接続テストが成功した場合、上部の **[保存]** ボタンをクリックします。 失敗した場合は、Workday URL と資格情報が Workday で有効であることを再度確認します。


### <a name="part-2-configure-attribute-mappings"></a>パート 2: 属性マッピングの構成 


このセクションでは、ユーザー データが Workday から Active Directory に移動する方法を構成します。

1.  **[マッピング]** の [プロビジョニング] タブで、**[Synchronize Azure AD Users to Workday (Azure AD ユーザーを Workday に同期する)]** をクリックします。

2.  **ソース オブジェクト スコープ** フィールドでは、必要があれば、メール アドレスを Workday に書き戻す対象を Azure Active Directory のユーザー セットからフィルタリングできます。 既定のスコープは、"Azure AD のすべてのユーザー" です。 

3.  **[属性マッピング]** セクションでは、個別の Workday 属性を Active Directory の属性にマッピングする方法を定義できます。 既定では、メール アドレスのマッピングがあります。 ただし、Azure AD のユーザーと Workday の対応するエントリを一致させるには、一致する ID を更新する必要があります。 一般的なマッチング メソッドは、Workday の Worker ID または従業員 ID を Azure AD の extensionAttribute1-15 に同期してから、この Azure AD の属性を使用して、Workday に戻ってユーザーを照合します。

4.  マッピングを保存するには、[属性マッピング] セクションの上部にある **[保存]** をクリックします。

### <a name="part-3-start-the-service"></a>パート 3: サービスの開始
パート 1 ～ 2 の部分が完了したら、プロビジョニング サービスを開始できます。

1.  **[プロビジョニング]** タブで、**[プロビジョニングの状態]** を **[ON]** に設定します。

2. **[Save]** をクリックします。

3. これにより初期同期が開始されます。これに要する時間は Workday のユーザー数に応じて異なります。

4. **[監査ログ]** タブで、個々の同期イベントを表示できます。**[監査ログの読み方の詳細については、プロビジョニング レポート ガイドを参照してください](active-directory-saas-provisioning-reporting.md)**

5. プロビジョニングが完了すると、次に示すように、**[プロビジョニング]** タブに監査概要レポートが書き込まれます。


## <a name="customizing-the-list-of-workday-user-attributes"></a>Workday のユーザー属性リストをカスタマイズする
Active Directory と Azure AD の Workday プロビジョニング アプリにはどちらも、Workday のユーザー属性の選択元となる既定のリストが含まれています。 ただしこれらのリストは、必要な情報をすべてカバーしているわけではありません。 Workday は、想定されるさまざまなユーザー属性を数多くサポートしていますが、それらのユーザー属性には、標準の属性と特定の Workday テナントに固有の属性とがあります。 

Azure AD プロビジョニング サービスは、このリスト (Workday 属性) をカスタマイズできるようになっており、Human Resources API の [Get_Workers](https://community.workday.com/sites/default/files/file-hosting/productionapi/Human_Resources/v21.1/Get_Workers.html) 操作に公開されている属性であれば、どのような属性でも含めることができます。

そのためには、使用する属性の XPath 式を [Workday Studio](https://community.workday.com/studio-download) を使って抽出し、Azure Portal の高度な属性エディターを使ってそれらをプロビジョニング構成に追加する必要があります。

**Workday のユーザー属性に使用する XPath 式を取得するには**

1. [Workday Studio](https://community.workday.com/studio-download) をダウンロードしてインストールします。 インストーラーにアクセスするには、Workday コミュニティ アカウントが必要です。

2. 次の URL から Workday Human_Resources WDSL ファイルをダウンロードします: https://community.workday.com/sites/default/files/file-hosting/productionapi/Human_Resources/v21.1/Human_Resources.wsdl

3. Workday Studio を起動します。

4. コマンド バーから **[Workday] > [Test Web Service in Tester]\(Tester で Web サービスをテストする\)** オプションを選択します。

5. **[External]\(外部\)** を選択し、手順 2. でダウンロードした Human_Resources WSDL ファイルを選択します。

    ![Workday Studio](./media/active-directory-saas-workday-inbound-tutorial/WDstudio1.PNG)

6. **[Location]\(場所\)** フィールドを `https://IMPL-CC.workday.com/ccx/service/TENANT/Human_Resources` に設定します。ただし、"IMPL-CC" の部分は実際のインスタンス タイプに、"TENANT" の部分は実際のテナント名に置き換えてください。

7. **[Operation]\(操作\)** を **[Get_Workers]** に設定します。

8.  [Request/Response]\(要求/応答\) ウィンドウの下に小さく表示された **[configure]\(構成\)** リンクをクリックして、Workday の資格情報を設定します。 **[Authentication]\(認証\)** チェック ボックスをオンにし、Workday 統合システム アカウントのユーザー名とパスワードを入力します。 ユーザー名は必ず name@tenant の形式で指定し、**[WS-Security UsernameToken]** オプションはオンのままにしてください。

    ![Workday Studio](./media/active-directory-saas-workday-inbound-tutorial/WDstudio2.PNG)

9. **[OK]** を選択します。

10. **[Request]\(要求\)** ウィンドウに以下の XML を貼り付けます。**Employee_ID** には、ご使用の Workday テナントに存在する実際のユーザーの従業員 ID を設定してください。 抽出対象となる属性が設定されているユーザーを選択します。

    ```
    <?xml version="1.0" encoding="UTF-8"?>
    <env:Envelope xmlns:env="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
      <env:Body>
        <wd:Get_Workers_Request xmlns:wd="urn:com.workday/bsvc" wd:version="v21.1">
          <wd:Request_References wd:Skip_Non_Existing_Instances="true">
            <wd:Worker_Reference>
              <wd:ID wd:type="Employee_ID">21008</wd:ID>
            </wd:Worker_Reference>
          </wd:Request_References>
          <wd:Response_Group>
            <wd:Include_Reference>true</wd:Include_Reference>
            <wd:Include_Personal_Information>true</wd:Include_Personal_Information>
            <wd:Include_Employment_Information>true</wd:Include_Employment_Information>
            <wd:Include_Management_Chain_Data>true</wd:Include_Management_Chain_Data>
            <wd:Include_Organizations>true</wd:Include_Organizations>
            <wd:Include_Reference>true</wd:Include_Reference>
            <wd:Include_Transaction_Log_Data>true</wd:Include_Transaction_Log_Data>
            <wd:Include_Photo>true</wd:Include_Photo>
            <wd:Include_User_Account>true</wd:Include_User_Account>
            <wd:Include_Roles>true</wd:Include_Roles>
          </wd:Response_Group>
        </wd:Get_Workers_Request>
      </env:Body>
    </env:Envelope>
    ```
 
11. **[Send Request]\(要求を送信する\)** (緑色の矢印) をクリックしてコマンドを実行します。 成功した場合、その応答が **[Response]\(応答\)** ウィンドウに表示されます。 エラーではなく、入力したユーザー ID のデータが応答に含まれていることを確認します。

12. 成功した場合は、**[Response]\(応答\)** ウィンドウから XML をコピーし、XML ファイルとして保存します。

13. Workday Studio のコマンド バーで **[File]\(ファイル\) > [Open File]\(ファイルを開く\)** を選択し、先ほど保存した XML ファイルを開きます。 Workday Studio の XML エディターにファイルが表示されます。

    ![Workday Studio](./media/active-directory-saas-workday-inbound-tutorial/WDstudio3.PNG)

14. ファイル ツリーで、**[/env:Envelope] > [env:Body] > [wd:Get_Workers_Response] > [wd:Response_Data] > [wd:Worker]** の順に移動して、該当するユーザーのデータを特定します。 

15. **[wd:Worker]** から、追加する属性を探して選択します。

16. 選択した属性の XPath 式を **[Document Path]\(ドキュメント パス\)** フィールドからコピーします。

17. コピーした式から **/env:Envelope/env:Body/wd:Get_Workers_Response/wd:Response_Data/** プレフィックスを削除します。 

18. コピーした式の最後の項目がノード (例: "/wd:Birth_Date") である場合、式の末尾に **/text()** を付加します。 最後の項目が属性 (例: "/@wd:type") である場合、この手順は不要です。

19. 最終的には、`wd:Worker/wd:Worker_Data/wd:Personal_Data/wd:Birth_Date/text()` のようになっている必要があります。 これを Azure Portal にコピーすることになります。


**プロビジョニング構成に Workday のカスタム ユーザー属性を追加するには**

1. このチュートリアルで前述した説明に従い、[Azure Portal](https://portal.azure.com) を起動して、対象の Workday プロビジョニング アプリケーションの [プロビジョニング] セクションに移動します。

2. **[プロビジョニングの状態]** を **[オフ]** に設定し、**[保存]** を選択します。 こうすることで、準備が整った段階でのみ、変更を反映することができます。

3. **[マッピング]** で **[Synchronize Workers to OnPremises]\(ワーカーをオンプレミスと同期\)** または **[Synchronize Workers to Azure AD]\(ワーカーを Azure AD と同期\)** を選択します。

4. 次の画面の一番下までスクロールして **[詳細オプションを表示する]** を選択します。

5. **[Edit attribute list for Workday]\(Workday の属性リストの編集\)** を選択します。

    ![Workday Studio](./media/active-directory-saas-workday-inbound-tutorial/WDstudio_AAD1.PNG)

6. 属性リストの一番下にある入力フィールドまでスクロールします。

7. **[名前]** に、属性の表示名を入力します。

8. **[Type]\(型\)** には、対象となる属性にふさわしい型を選択します (通常は **String** が一般的)。

9. **[API Expression]\(API 式\)** には、Workday Studio からコピーした XPath 式を入力します。 例: `wd:Worker/wd:Worker_Data/wd:Personal_Data/wd:Birth_Date/text()`

10. **[属性の追加]** を選択します。

    ![Workday Studio](./media/active-directory-saas-workday-inbound-tutorial/WDstudio_AAD2.PNG)

11. 上の **[保存]** を選択し、ダイアログで **[はい]** を選択します。 [属性マッピング] 画面をまだ開いている場合は閉じてください。

12. メインの **[プロビジョニング]** タブに戻り、**[Synchronize Workers to OnPremises]\(ワーカーをオンプレミスと同期\)** または **[Synchronize Workers to Azure AD]\(ワーカーを Azure AD と同期\)** を再び選択します。

13. **[新しいマッピングの追加]** を選択します。

14. **[ソース属性]** リストに新しい属性が表示されます。

15. 必要に応じて新しい属性のマッピングを追加します。

16. 完了したら、忘れずに **[プロビジョニングの状態]** を **[オン]** に設定してから保存してください。


## <a name="known-issues"></a>既知の問題

* PowerShell コマンド **Add-ADSyncAgentAzureActiveDirectoryConfiguration** の実行中、グローバル管理者がカスタム ドメイン (例: admin@contoso.com) を使用すると、その管理者の資格情報が動作しません。これは既知の問題です。 この問題を回避するには、onmicrosoft.com ドメインで Azure AD のグローバル管理者アカウント (例: admin@contoso.onmicrosoft.com) を作成し、使用します。

* EU の Azure AD テナントに監査ログが表示されないという以前の問題は解決されています。 ただし、EU のAzure AD テナントには追加エージェント構成が必要です。 詳細については、「[パート 3: オンプレミスの同期エージェントの構成](#Part 3: Configure the on-premises synchronization agent)」を参照してください

## <a name="gdpr-compliance"></a>GDPR コンプライアンス

[一般データ保護規則 (GDPR)](http://ec.europa.eu/justice/data-protection/reform/index_en.htm)は、欧州連合 (EU) のデータ保護およびプライバシー保護法です。 GDPR は、EU 内で人々に製品やサービスを提供したり、EU 居住者に関連するデータを収集および分析したりする企業、政府機関、非営利組織やその他の組織を対象とするルールです。 

Azure AD プロビジョニング サービスは、Microsoft の他のサービスおよび機能と共に GDPR に準拠しています。 Microsoft の GDPR ストーリーについて詳しくは、[サービス利用規約](https://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&DocumentTypeId=31)に関するページをご覧ください。

ただし、Active Directory の Workday プロビジョニング ソリューションでは、ドメインに参加しているサーバーに同期エージェントをインストールする必要があるため、GDPR 準拠を維持していることの監視が必要な場合があります。
 
エージェントはログを **Windows イベント ログ**に作成し、ここには個人を特定できる情報が含まれることがあります。

GDPR 準拠を維持する方法は 2 つあります。

1. 要求を受けた際、個人のデータを抽出し、Windows イベント ログからその個人のデータを削除する。 
2. AADSyncAgent プロセスに由来する Windows イベント ログのリテンション期間を 48 時間未満に維持する

Windows イベント ログのデータ保有期間の構成方法については、「[Settings for event logs (イベント ログの設定)](https://technet.microsoft.com/library/cc952132.aspx)」をご覧ください。 Windows イベント ログの一般情報については、[この記事](https://msdn.microsoft.com/library/windows/desktop/aa385772.aspx)をご覧ください。


## <a name="next-steps"></a>次の手順

* [プロビジョニング アクティビティのログの確認方法およびレポートの取得方法](https://docs.microsoft.com/azure/active-directory/active-directory-saas-provisioning-reporting)
* [Workday と Azure Active Directory の間でシングル サインオンを構成する方法](active-directory-saas-workday-tutorial.md)
* [他の SaaS アプリケーションを Azure Active Directory と統合する方法](active-directory-saas-tutorial-list.md)

