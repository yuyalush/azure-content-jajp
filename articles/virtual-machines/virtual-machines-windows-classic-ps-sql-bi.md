<properties
	pageTitle="SQL Server Business Intelligence | Microsoft Azure"
	description="このトピックでは、クラシック デプロイ モデルを使用して作成されたリソースを使用し、Azure 仮想マシン (VM) 上で実行されている SQL Server で使用できる Business Intelligence (BI) 機能について説明します。"
	services="virtual-machines-windows"
	documentationCenter="na"
	authors="guyinacube"
	manager="mblythe"
	editor="monicar"
	tags="azure-service-management"/>
<tags
	ms.service="virtual-machines-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.tgt_pltfrm="vm-windows-sql-server"
	ms.workload="infrastructure-services"
	ms.date="05/13/2016"
	ms.author="asaxton" />

# Azure Virtual Machines での SQL Server Business Intelligence

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-classic-include.md)]リソース マネージャー モデル。


Microsoft Azure 仮想マシン ギャラリーには、SQL Server インストールを含むイメージが用意されています。ギャラリー イメージでサポートされている SQL Server のエディションは、オンプレミスのコンピューターにも仮想マシンにもインストールできるインストール ファイルです。このトピックでは、イメージにインストールされている SQL Server Business Intelligence (BI) 機能の概要と、仮想マシンのプロビジョニング後に必要な構成手順について説明します。また、BI 機能用にサポートされているデプロイ トポロジとベスト プラクティスについても説明します。

## ライセンスに関する考慮事項

Microsoft Azure Virtual Machines の SQL Server のライセンスを取得するには、次の 2 つの方法があります。

1. ソフトウェア アシュアランスの一部であるライセンス モビリティを利用する。詳細については、「[Azure でのソフトウェア アシュアランスによるライセンス モビリティ](https://azure.microsoft.com/pricing/license-mobility/)」を参照してください。

1. SQL Server がインストールされている Azure Virtual Machines の時間単位の料金を支払う。「[Virtual Machines の価格](https://azure.microsoft.com/pricing/details/virtual-machines/#Sql)」の SQL Server に関するセクションをご覧ください。

ライセンスと現在の料金の詳細については、「[Virtual Machines Licensing FAQ (Virtual Machines のライセンスに関するFAQ)](https://azure.microsoft.com/pricing/licensing-faq/%20/)」 をご覧ください。

## Azure 仮想マシン ギャラリーで提供される SQL Server イメージ

Microsoft Azure 仮想マシン ギャラリーには、Microsoft SQL Server を含むイメージがいくつか用意されています。仮想マシン イメージにインストールされているソフトウェアは、オペレーティング システムのバージョンや SQL Server のバージョンによって異なります。Azure 仮想マシン ギャラリーで提供されるイメージのリストは頻繁に変更されます。

![Azure VM ギャラリーの SQL イメージ](./media/virtual-machines-windows-classic-ps-sql-bi/IC741367.png)

![PowerShell](./media/virtual-machines-windows-classic-ps-sql-bi/IC660119.gif) 次の PowerShell スクリプトは、ImageName に "SQL Server" が含まれた Azure イメージのリストを返します。

	# assumes you have already uploaded a management certificate to your Microsoft Azure Subscription. View the thumbprint value from the "settings" menu in Azure classic portal.

	$subscriptionID = ""    # REQUIRED: Provide your subscription ID.
	$subscriptionName = "" # REQUIRED: Provide your subscription name.
	$thumbPrint = "" # REQUIRED: Provide your certificate thumbprint.
	$certificate = Get-Item cert:\currentuser\my\$thumbPrint # REQUIRED: If your certificate is in a different store, provide it here.-Ser  store is the one specified with the -ss parameter on MakeCert

	Set-AzureSubscription -SubscriptionName $subscriptionName -Certificate $certificate -SubscriptionID $subscriptionID

	Write-Host -foregroundcolor green "List of available gallery images where imagename contains 2016"
	Write-Host -foregroundcolor green ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>"
	get-azurevmimage | where {$_.ImageName -Like "*SQL-Server-2016*"} | select imagename,category, location, label, description

	Write-Host -foregroundcolor green "List of available gallery images where imagename contains 2014"
	Write-Host -foregroundcolor green ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>"
	get-azurevmimage | where {$_.ImageName -Like "*SQL-Server-2014*"} | select imagename,category, location, label, description

SQL Server でサポートされているエディションと機能の詳細については、以下をご覧ください。

- [SQL Server のエディション](https://www.microsoft.com/server-cloud/products/sql-server-editions/#fbid=Zae0-E6r5oh)

- [SQL Server 2016 の各エディションがサポートする機能](https://msdn.microsoft.com/library/cc645993.aspx)

### SQL Server 仮想マシン ギャラリー イメージにインストールされている BI 機能

次の表は、SQL Server の一般的な Microsoft Azure 仮想マシン ギャラリー イメージにインストールされている Business Intelligence 機能を示しています。

- SQL Server 2016 RC3

- SQL Server 2014 SP1 Enterprise

- SQL Server 2014 SP1 Standard

- SQL Server 2012 SP2 Enterprise

- SQL Server 2012 SP2 Standard

|SQL Server BI 機能|ギャラリー イメージにインストールされているかどうか|メモ|
|---|---|---|
|**Reporting Services ネイティブ モード**|あり|インストールされていますが、レポート マネージャー URL などの構成が必要です。「[Reporting Services の構成](#configure-reporting-services)」をご覧ください。|
|**Reporting Services SharePoint モード**|いいえ|Microsoft Azure 仮想マシン ギャラリー イメージには、SharePoint も SharePoint インストール ファイルも含まれていません。<sup>1</sup>|
|**Analysis Services 多次元およびデータ マイニング (OLAP)**|あり|インストールされ、既定の Analysis Services インスタンスとして構成されています。|
|**Analysis Services 表形式**|いいえ|SQL Server 2012、2014、および 2016 のイメージでサポートされていますが、既定ではインストールされていません。Analysis Services の別のインスタンスをインストールします。このトピックの「SQL Server のその他のサービスと機能のインストール」をご覧ください。|
|**Analysis Services Power Pivot for SharePoint**|いいえ|Microsoft Azure 仮想マシン ギャラリー イメージには、SharePoint も SharePoint インストール ファイルも含まれていません。<sup>1</sup>|

<sup>1</sup> SharePoint と Azure 仮想マシンの追加情報については、「[SharePoint 2013 用の Microsoft Azure アーキテクチャ](https://technet.microsoft.com/library/dn635309.aspx)」および「[Microsoft Azure Virtual Machines の SharePoint デプロイ](https://www.microsoft.com/download/details.aspx?id=34598)」をご覧ください。

![PowerShell](./media/virtual-machines-windows-classic-ps-sql-bi/IC660119.gif) 次の PowerShell コマンドを実行して、サービス名に "SQL" が含まれたインストール済みサービスのリストを取得します。

	get-service | Where-Object{ $_.DisplayName -like '*SQL*' } | Select DisplayName, status, servicetype, dependentservices | format-Table -AutoSize

## 一般的な推奨事項とベスト プラクティス

- SQL Server Enterprise Edition の使用時に推奨される仮想マシンの最小サイズは **A3** です。Analysis Services と Reporting Services の SQL Server BI デプロイでは、推奨される仮想マシン サイズは **A4** です。

	最新の VM サイズについては、「[仮想マシンのサイズ](virtual-machines-linux-sizes.md)」をご覧ください。

- ディスク管理のベスト プラクティスとして、データ ファイル、ログ ファイル、バックアップ ファイルは **C**: および **D**: 以外のドライブに保存することをお勧めします。たとえば、データ ディスク **E**: と **F**: を作成します。

	- 既定の **C**: ドライブのドライブ キャッシュ ポリシーは、データ操作に最適でありません。

	- **D**: ドライブは、主にページ ファイルに使用される一時ドライブです。**D**: ドライブは永続化されず、BLOB ストレージには保存されません。仮想マシンのサイズ変更などの管理タスクを行うと、**D**: ドライブはリセットされます。tempdb も含め、データベース ファイルには** D**: ドライブを使用**しない**ことをお勧めします。

	ディスクの作成と接続の詳細については、「[データ ディスクを Windows 仮想マシンに接続する方法](virtual-machines-windows-classic-attach-disk.md)」をご覧ください。

- 使用する予定がないサービスは、停止またはアンインストールします。たとえば、仮想マシンを Reporting Services だけに使用する場合は、Analysis Services と SQL Server Integration Services を停止またはアンインストールします。次の図は、既定で開始されるサービスの例を示しています。

	![SQL Server サービス](./media/virtual-machines-windows-classic-ps-sql-bi/IC650107.gif)

	>[AZURE.NOTE] サポートされている BI シナリオでは、SQL Server データベース エンジンが必要です。1 つのサーバー VM のトポロジでは、データベース エンジンが同じ VM 上で実行されている必要があります。

	詳細については、「[Reporting Services のアンインストール](https://msdn.microsoft.com/library/hh479745.aspx)」および「[Analysis Services のインスタンスをアンインストールする方法](https://msdn.microsoft.com/library/ms143687.aspx)」をご覧ください。

- **Windows Update** で新しい "重要な更新プログラム" があるかどうかを確認します。Microsoft Azure 仮想マシン イメージは頻繁に更新されます。ただし、VM イメージの最終更新後は、重要な更新プログラムを **Windows Update** から入手できるようになります。

## デプロイ トポロジの例

Microsoft Azure Virtual Machines を使用するデプロイの例を以下に示します。これらの図のトポロジは、SQL Server BI 機能と Microsoft Azure Virtual Machines で使用できるトポロジの一部にすぎません。

### 1 つの仮想マシン

1 つの仮想マシン上の Analysis Services、Reporting Services、SQL Server データベース エンジン、データ ソース。

![1 つの仮想マシンを使用する BI IaaS シナリオ](./media/virtual-machines-windows-classic-ps-sql-bi/IC650108.gif)

### 2 つの仮想マシン

- 1 つの仮想マシン上の Analysis Services、Reporting Services、SQL Server データベース エンジン。このデプロイには、レポート サーバー データベースが含まれます。

- 2 つ目の VM 上のデータ ソース。2 つ目の VM には、データ ソースとして SQL Server データベース エンジンが含まれます。

![2 つの仮想マシンを使用する BI IaaS シナリオ](./media/virtual-machines-windows-classic-ps-sql-bi/IC650109.gif)

### Azure が混在 - Azure SQL Database 上のデータ

- 1 つの仮想マシン上の Analysis Services、Reporting Services、SQL Server データベース エンジン。このデプロイには、レポート サーバー データベースが含まれます。

- データ ソースは Azure SQL Database です。

![VM とデータ ソースとして Azure SQL を使用する BI IaaS シナリオ](./media/virtual-machines-windows-classic-ps-sql-bi/IC650110.gif)

### ハイブリッド - オンプレミスのデータ

- このデプロイ例では、1 つの仮想マシン上で Analysis Services、Reporting Services、SQL Server データベース エンジンが実行されます。仮想マシンは、レポート サーバー データベースをホストします。仮想マシンは、Azure Virtual Networking またはその他の VPN トンネリング ソリューションを使用して、オンプレミス ドメインに参加しています。

- データ ソースはオンプレミスです。

![VM とオンプレミス データ ソースを使用する BI IaaS シナリオ](./media/virtual-machines-windows-classic-ps-sql-bi/IC654384.gif)

## Reporting Services ネイティブ モードの構成

SQL Server の仮想マシン ギャラリー イメージには、Reporting Services ネイティブ モードがインストールされていますが、レポート サーバーは構成されていません。このセクションの手順では、Reporting Services レポート サーバーを構成します。Reporting Services ネイティブ モードの構成の詳細については、「[Reporting Services ネイティブ モードのレポート サーバーのインストール](https://msdn.microsoft.com/library/ms143711.aspx)」をご覧ください。

>[AZURE.NOTE] Windows PowerShell スクリプトを使用したレポート サーバーの構成に関する類似コンテンツについては、「[ネイティブ モードのレポート サーバーを実行する Azure VM を PowerShell を使用して作成する](virtual-machines-windows-classic-ps-sql-report.md)」をご覧ください。

### 仮想マシンへの接続と Reporting Services 構成マネージャーの起動

Azure 仮想マシンに接続するための 2 つの一般的なワークフローがあります。

- で接続するには、仮想マシンの名前をクリックし、**[接続]** をクリックします。リモート デスクトップ接続が開き、コンピューター名が自動的に設定されます。

	![Azure 仮想マシンに接続](./media/virtual-machines-windows-classic-ps-sql-bi/IC650112.gif)

- Windows リモート デスクトップ接続を使用して、仮想マシンに接続します。リモート デスクトップのユーザー インターフェイスで、次の手順を実行します。

	1. コンピューター名として**クラウド サービス名**を入力します。

	1. コロン (:) を入力し、TCP リモート デスクトップ エンドポイント用に構成されているパブリック ポート番号を入力します。

		Myservice.cloudapp.net:63133

		詳細については、「[What is a Cloud Service? (クラウド サービスとは)](https://azure.microsoft.com/manage/services/cloud-services/what-is-a-cloud-service/)」を参照してください。

**Reporting Services 構成マネージャーを起動します。**

1. **Windows Server 2012** では、次の手順を実行します。

1. **[スタート]** 画面で、「**Reporting Services**」と入力してアプリケーションの一覧を表示します。

1. **[Reporting Services 構成マネージャー]** を右クリックし、**[管理者として実行]** をクリックします。

1. **Windows Server 2008 R2** では、次の手順を実行します。

1. **[スタート]** ボタンをクリックし、**[すべてのプログラム]** をクリックします。

1. **[Microsoft SQL Server 2016]** をクリックします。

1. **[構成ツール]** をクリックします。

1. **[Reporting Services 構成マネージャー]** を右クリックし、**[管理者として実行]** をクリックします。

または

1. **[開始]** をクリックします。

1. **[プログラムとファイルの検索]** ダイアログで、「**reporting services**」と入力します。VM で Windows Server 2012 が実行されている場合は、Windows Server 2012 のスタート画面で「**reporting services**」と入力します。

1. **[Reporting Services 構成マネージャー]** を右クリックし、**[管理者として実行]** をクリックします。

	![SSRS 構成マネージャーの検索](./media/virtual-machines-windows-classic-ps-sql-bi/IC650113.gif)

### Reporting Services の構成

**サービス アカウントと Web サービス URL:**

1. **[サーバー名]** がローカル サーバー名であることを確認し、**[接続]** をクリックします。

1. **[レポート サーバー データベース名]** は空白になっています。データベースは構成の完了時に作成されます。

1. **[レポート サーバーの状態]** が **[開始]** になっていることを確認します。Windows サーバー マネージャーでサービスを確認する場合、サービスは **[SQL Server Reporting Services]** Windows サービスです。

1. **[サービス アカウント]** をクリックし、必要に応じてアカウントを変更します。ドメインに参加していない環境で仮想マシンを使用する場合は、組み込みの **ReportServer** アカウントで十分です。サービス アカウントの詳細については、「[サービス アカウント (SSRS ネイティブ モード)](https://msdn.microsoft.com/library/ms189964.aspx)」をご覧ください。

1. 左側のウィンドウで、**[Web サービス URL]** をクリックします。

1. **[適用]** をクリックして、既定値を構成します。

1. **[レポート サーバー Web サービスの URL]** を確認します。既定の TCP ポートが 80 であり、URL に含まれていることを確認します。後の手順で、このポートの Microsoft Azure 仮想マシン エンドポイントを作成します。

1. **[結果]** ウィンドウで、操作が正常に完了したことを確認します。

**データベース:**

1. 左側のウィンドウで、**[データベース]** をクリックします。

1. **[データベースの変更]** をクリックします。

1. **[新しいレポート サーバー データベースを作成する]** が選択されていることを確認し、[次へ] をクリックします。

1. **[サーバー名]** を確認し、**[接続テスト]** をクリックします。

1. **"接続テストに成功しました"** というメッセージが表示されたら、**[OK]** をクリックし、**[次へ]** をクリックします。

1. データベース名が **[ReportServer]**、**[レポート サーバー モード]** が **[ネイティブ]** であることを確認し、**[次へ]** をクリックします。

1. **[資格情報]** ページで **[次へ]** をクリックします。

1. **[概要]** ページで **[次へ]** をクリックします。

1. **[続行して完了する]** ページで **[次へ]** をクリックします。

**Web ポータル URL または 2012 および 2014 用のレポート マネージャ URL:**

1. 左側のウィンドウで、**[Web ポータル URL]** または2014 および 2012 用の **[レポート manager URL]** をクリックします。

1. **[Apply]** をクリックします。

1. **[結果]** ウィンドウで、操作が正常に完了したことを確認します。

1. **[終了]** をクリックします。

レポート サーバーのアクセス許可については、「[ネイティブ モードのレポート サーバーに対する権限の許可](https://msdn.microsoft.com/library/ms156014.aspx)」をご覧ください。

### ローカル レポート マネージャーの参照

構成を確認するには、VM 上でレポート マネージャーを参照します。

1. VM で、管理者特権を使用して Internet Explorer を起動します。

1. VM で http://localhost/reports を参照します。

### リモート Web ポータル、または 2014 および 2012 用のレポート マネージャーに接続するには

リモート コンピューターから仮想マシン上の Web ポータル、または 2014 および 2012 用のレポート マネージャーに接続する場合は、新しい仮想マシン TCP エンドポイントを作成します。既定では、レポート サーバーは**ポート 80** で HTTP 要求をリッスンします。別のポートを使用するようにレポート サーバーの URL を構成した場合は、次の手順でそのポート番号を指定する必要があります。

1. TCP ポート 80 の仮想マシンのエンドポイントを作成します。詳細については、このドキュメントの「[仮想マシン エンドポイントとファイアウォール ポート](#virtual-machine-endpoints-and-firewall-ports)」をご覧ください。

1. 仮想マシンのファイアウォールでポート 80 を開きます。

1. URL 内のサーバー名として Azure 仮想マシンの **DNS 名**を使用して、Web ポータルまたはレポート マネージャーを参照します。次に例を示します。

	**レポート サーバー**: http://uebi.cloudapp.net/reportserver **Web ポータル**: http://uebi.cloudapp.net/reports

	[レポート サーバー アクセスに対するファイアウォールの構成](https://msdn.microsoft.com/library/bb934283.aspx)

### レポートを作成して Azure 仮想マシンに発行するには

Microsoft Azure 仮想マシンでホストされているレポート サーバーに、オンプレミスのコンピューターから既存のレポートを発行する際に使用できるオプションの一部を次に示します。

- **レポート ビルダー**: 仮想マシンには、SQL 2014 および 2012 用の Microsoft SQL Server レポート ビルダーの ClickOnce バージョンが含まれています。仮想マシン上で SQL 2016 を使用してレポート ビルダーを初めて起動するときは、次の手順を実行します。

	1. 管理者特権でブラウザーを起動します。

	1. 仮想マシン上で Web ポータルを参照し、右上の **[ダウンロード]** アイコンを選択します。
	
	1. **[レポート ビルダー]** を選択します。

	詳細については、「[レポート ビルダーの起動](https://msdn.microsoft.com/library/ms159221.aspx)」を参照してください。

- **SQL Server Data Tools: VM**: SQL Server Data Tools は仮想マシンにインストールされます。このツールを使用して、仮想マシンで**レポート サーバー プロジェクト**とレポートを作成できます。SQL Server Data Tools では、仮想マシン上のレポート サーバーにレポートを発行できます。

- **SQL Server Data Tools: Remote**: ローカル コンピューター上の SQL Server Data Tools で、Reporting Services レポートを含む Reporting Services プロジェクトを作成します。Web サービス URL に接続するようにプロジェクトを構成します。

	![SSRS プロジェクトの SSDT プロジェクト プロパティ](./media/virtual-machines-windows-classic-ps-sql-bi/IC650114.gif)

- レポートを格納する .VHD ハード ドライブを作成し、ドライブをアップロードして接続します。

	1. ローカル コンピューター上で、レポートを格納する .VHD ハード ドライブを作成します。

	1. 管理証明書を作成し、インストールします。

	1. Add-AzureVHD コマンドレットを使用して、VHD ファイルを Azure にアップロードします (「[Windows Server VHD の作成と Azure へのアップロード](virtual-machines-windows-classic-createupload-vhd.md)」を参照)。

	1. ディスクを仮想マシンに接続します。

## SQL Server のその他のサービスと機能のインストール

表形式モードの Analysis Services など、SQL Server のその他のサービスをインストールするには、SQL Server セットアップ ウィザードを実行します。セットアップ ファイルは仮想マシンのローカル ディスクにあります。

1. **[スタート]** ボタンをクリックし、**[すべてのプログラム]** をクリックします。

1. [**Microsoft SQL Server 2016]**、**[Microsoft SQL Server 2014]**、または **[Microsoft SQL Server 2012]** をクリックし、**[構成ツール]** をクリックします。

1. **[SQL Server インストール センター]** をクリックします。

または、C:\\SQLServer\_13.0\_full\\setup.exe、C:\\SQLServer\_12.0\_full\\setup.exe、または C:\\SQLServer\_11.0\_full\\setup.exe を実行します。

>[AZURE.NOTE] SQL Server セットアップを初めて実行すると、さらに多くのセットアップ ファイルがダウンロードされ、仮想マシンの再起動と SQL Server セットアップの再起動が必要になる場合があります。
>
>Microsoft Azure 仮想マシンから選択したイメージを何度もカスタマイズする必要がある場合は、独自の SQL Server イメージを作成することを検討してください。SQL Server 2012 SP1 CU2 で、Analysis Services SysPrep 機能が有効になりました。詳細については、「[SysPrep を使用した SQL Server のインストールに関する注意点](https://msdn.microsoft.com/library/ee210754.aspx)」をご覧ください。

### Analysis Services 表形式モードをインストールするには

このセクションの手順では、Analysis Services 表形式モードのインストールの**概要**を示しています。詳細については、次のトピックを参照してください。

- [表形式モードでの Analysis Services のインストール](https://msdn.microsoft.com/library/hh231722.aspx)

- [テーブル モデリング (Adventure Works チュートリアル)](https://msdn.microsoft.com/library/140d0b43-9455-4907-9827-16564a904268)

**Analysis Services 表形式モードをインストールするには:**

1. SQL Server インストール ウィザードで、左側のウィンドウの **[インストール]** をクリックし、**[SQL Server の新規スタンドアロン インストールを実行するか、既存のインストールに機能を追加します]** をクリックします。

	- **[フォルダーの参照]** が表示されたら、c:\\SQLServer\_13.0\_full、c:\\SQLServer\_12.0\_full、または c:\\SQLServer\_11.0\_full を参照し、**[OK]** をクリックします。

1. 製品の更新プログラム ページで **[次へ]** をクリックします。

1. **[インストールの種類]** ページで、**[SQL Server の新規インストールを実行する]** を選択し、**[次へ]** をクリックします。

1. **[セットアップ ロール]** ページで、**[SQL Server 機能のインストール]** をクリックします。

1. **[機能の選択]** ページで、**[Analysis Services]** をクリックします。

1. **[インスタンスの構成]** ページで、**[名前付きインスタンス]** ボックスと **[インスタンス ID]** ボックスに、「**表形式**」などのわかりやすい名前を入力します。

1. **[Analysis Services の構成]** ページで、**[表形式モード]** を選択します。現在のユーザーを管理権限の一覧に追加します。

1. SQL Server インストール ウィザードを完了して閉じます。

## Analysis Services の構成

### Analysis Services サーバーへのリモート アクセス

Analysis Services サーバーでは、Windows 認証だけがサポートされています。SQL Server Management Studio や SQL Server Data Tools などのクライアント アプリケーションから Analysis Services にリモート アクセスするには、Azure Virtual Networking を使用して、仮想マシンをローカル ドメインに参加させる必要があります。詳細については、「[Virtual Network の概要](../virtual-network/virtual-networks-overview.md)」をご覧ください。

Analysis Services の**既定のインスタンス**は、TCP ポート **2383** でリッスンします。仮想マシンのファイアウォールでポートを開きます。Analysis Services のクラスター化された名前付きインスタンスも、ポート **2383** でリッスンします。

Analysis Services の**名前付きインスタンス**の場合、ポート アクセスを管理するために SQL Server Browser サービスが必要です。SQL Server Browser の既定の構成は、ポート **2382** です。

仮想マシンのファイアウォールで、ポート **2382** を開き、Analysis Services の名前付きインスタンスの静的ポートを作成します。

1. VM 上で既に使用されているポートと、ポートを使用しているプロセスを確認するには、管理者特権で次のコマンドを実行します。

		netstat /ao

1. SQL Server Management Studio を使用して、表形式 AS インスタンスの全般プロパティで "ポート" 値を更新し、Analysis Services の名前付きインスタンスの静的ポートを作成します。詳細については、「[Analysis Services のアクセスを許可するための Windows ファイアウォールの構成](https://msdn.microsoft.com/library/ms174937.aspx#bkmk_fixed)」の「Analysis Services の既定のインスタンスまたは名前付きインスタンスに対する固定ポートの使用」をご覧ください。

1. Analysis Services サービスの表形式インスタンスを再起動します。

詳細については、このドキュメントの「**仮想マシン エンドポイントとファイアウォール ポート**」をご覧ください。

## 仮想マシン エンドポイントとファイアウォール ポート

ここでは、作成する Microsoft Azure 仮想マシン エンドポイントと、仮想マシンのファイアウォールで開くポートの概要を説明します。次の表は、エンドポイントの作成対象の **TCP** ポートと、仮想マシンのファイアウォールで開くポートを示しています。

- VM を 1 つだけ使用し、次の 2 つの条件に該当する場合は、VM エンドポイントを作成する必要はありません。また、VM 上のファイアウォールでポートを開く必要もありません。

	- VM 上の SQL Server 機能にリモート接続しない。VM へのリモート デスクトップ接続を確立し、VM 上で SQL Server 機能にローカルでアクセスすることは、SQL Server 機能へのリモート接続とは見なされません。

	- Azure Virtual Networking または別の VPN トンネリング ソリューションを使用して、VM をオンプレミス ドメインに参加させない。

- 仮想マシンがドメインに参加していない場合に VM 上の SQL Server 機能にリモート接続するには、次の操作を行います。

	- VM 上のファイアウォールでポートを開きます。

	- アスタリスク (*) が付いているポートの仮想マシン エンドポイントを作成します。

- 仮想マシンが Azure Virtual Networking などの VPN トンネルを使用してドメインに参加している場合、エンドポイントは不要です。ただし、VM 上のファイアウォールでポートを開いてください。

	|ポート|型|説明|
|---|---|---|
|**80**|TCP|レポート サーバー リモート アクセス (*).|
|**1433**|TCP|SQL Server Management Studio (*).|
|**1434**|UDP|SQL Server Browser。これは、VM がドメインに参加しているときに必要となります。|
|**2382**|TCP|SQL Server Browser。|
|**2383**|TCP|SQL Server Analysis Services の既定のインスタンスとクラスター化された名前付きインスタンス。|
|**ユーザー定義**|TCP|選択したポート番号で Analysis Services の名前付きインスタンスの静的ポートを作成し、ファイアウォールでそのポート番号のブロックを解除します。|

エンドポイントの作成の詳細については、次のトピックをご覧ください。

- エンドポイントの作成: [仮想マシンに対してエンドポイントを設定する方法](virtual-machines-windows-classic-setup-endpoints.md)

- SQL Server: 「[Azure での SQL Server 仮想マシンのプロビジョニング](virtual-machines-windows-portal-sql-server-provision.md)」の「別のコンピューターにある SQL Server Management Studio を使用して仮想マシンに接続するための構成手順を完了する」をご覧ください。

次の図は、VM 上の機能とコンポーネントへのリモート アクセスを可能にするために、VM のファイアウォールで開くポートを示しています。

![Azure VM の BI アプリケーションに対して開くポート](./media/virtual-machines-windows-classic-ps-sql-bi/IC654385.gif)

## リソース

- Azure 仮想マシン環境で使用される、マイクロソフト サーバー ソフトウェアのサポート ポリシーを確認します。BitLocker、フェールオーバー クラスタリング、ネットワーク負荷分散などの機能のサポートについては、「[Microsoft Azure Virtual Machines のマイクロソフト サーバー ソフトウェアのサポート](http://support.microsoft.com/kb/2721672)」をご覧ください。

- [Azure Virtual Machines における SQL Server の概要](virtual-machines-windows-sql-server-iaas-overview.md)

- [Virtual Machines](https://azure.microsoft.com/documentation/services/virtual-machines/)

- [Azure での SQL Server 仮想マシンのプロビジョニング](virtual-machines-windows-portal-sql-server-provision.md)

- [データ ディスクを仮想マシンに接続する方法](virtual-machines-windows-classic-attach-disk.md)

- [Azure VM の SQL Server へのデータベースの移行](virtual-machines-windows-migrate-sql.md)

- [Analysis Services インスタンスのサーバー モードの決定](https://msdn.microsoft.com/library/gg471594.aspx)

- [多次元モデリング (Adventure Works チュートリアル)](https://technet.microsoft.com/library/ms170208.aspx)

- [Azure ドキュメント センター](https://azure.microsoft.com/documentation/)

- [Using Power BI in a Hybrid Environment (ハイブリッド環境での Power BI の使用)](https://msdn.microsoft.com/library/dn798994.aspx)

>[AZURE.NOTE] [Submit feedback and contact information through Microsoft SQL Server Connect](https://connect.microsoft.com/SQLServer/Feedback)

### コミュニティ コンテンツ

- [Azure SQL Database Management with PowerShell (PowerShell を使用した Azure SQL Database の管理)](http://blogs.msdn.com/b/windowsazure/archive/2013/02/07/windows-azure-sql-database-management-with-powershell.aspx)

<!---HONumber=AcomDC_0518_2016-->