---
title: "最小限のダウンタイムでの Azure Database for MySQL への移行"
description: "この記事では、最小限のダウンタイムで MySQL データベースから Azure Database for MySQL への移行を実行する方法と、Attunity Replicate for Microsoft Migrations を使用してソース データベースからターゲット データベースへの初期読み込みと継続的なデータ同期を設定する方法について説明します。"
services: mysql
author: HJToland3
ms.author: jtoland
manager: kfile
editor: jasonwhowell
ms.service: mysql-database
ms.topic: article
ms.date: 02/28/2018
ms.openlocfilehash: e1be72d97570643cc8a7c6eb05d3d363e96357b6
ms.sourcegitcommit: c765cbd9c379ed00f1e2394374efa8e1915321b9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/28/2018
---
# <a name="minimal-downtime-migration-to-azure-database-for-mysql"></a>最小限のダウンタイムでの Azure Database for MySQL への移行
Attunity Replicate for Microsoft Migrations を使用して、既存の MySQL データベースを Azure Database for MySQL に移行できます。 Attunity Replicate は、Attunity と Microsoft の共同製品です。 これは、Azure Database Migration Service と共に、Microsoft のお客様には追加コストなしで含まれています。 

Attunity Replicate は、データベース移行中のダウンタイムを最小限に抑えるのに役立ち、プロセス全体を通してソース データベースを動作状態に維持します。

Attunity Replicate は、さまざまなソースとターゲット間のデータ同期を可能にするデータ レプリケーション ツールです。 スキーマ作成スクリプトと各データベース テーブルに関連するデータを伝達します。 Attunity Replicate は、他のアーティファクト (SP、トリガー、関数など) を伝達したり、このようなアーティファクトでホストされている PL/SQL コードなどを T-SQL に変換したりすることはありません。

> [!NOTE]
> Attunity Replicate は幅広い一連の移行シナリオをサポートしていますが、ソースとターゲットのペアの特定のサブセットのサポートに重点を置いています。

最小限のダウンタイムでの移行の実行プロセスの概要を次に示します。

* [MySQL Workbench](https://www.mysql.com/products/workbench/) を使用して、**MySQL ソース スキーマを、管理されたデータベース サービスである Azure Database for MySQL に移行**します。

* Attunity Replicate for Microsoft Migrations を使用して、**ソース データベースからターゲット データベースへの初期読み込みと継続的なデータ同期を設定**します。 これにより、アプリケーションを Azure 上のターゲット MySQL データベースに切り替える準備をするときに、ソース データベースを読み取り専用として設定する必要のある時間が最小限に抑えられます。

Attunity Replicate for Microsoft Migrations 製品の詳細については、次のリソースを参照してください。
 - [Attunity Replicate for Microsoft Migrations](https://aka.ms/attunity-replicate) の Web ページに移動します。
 - [Attunity Replicate for Microsoft Migrations](http://discover.attunity.com/download-replicate-microsoft-lp6657.html) をダウンロードします。
 - クイック スタート ガイド、チュートリアル、サポートについては、[Attunity Replicate コミュニティ](https://aka.ms/attunity-community)をご覧ください。
 - Attunity Replicate を使用して MySQL データベースを Azure Database for MySQL に移行する詳細な手順については、「[Database Migration Guide (データベース移行ガイド)](https://datamigration.microsoft.com/scenario/mysql-to-azuremysql)」をご覧ください。