<properties 
	pageTitle="Windows ユニバーサル SDK の概要" 
	description="Azure Mobile Engagement 向け Windows ユニバーサル SDK の概要" 									
	services="mobile-engagement" 
	documentationCenter="mobile" 
	authors="piyushjo" 
	manager="dwrede" 
	editor="" />

<tags 
	ms.service="mobile-engagement" 
	ms.workload="mobile" 
	ms.tgt_pltfrm="mobile-windows-store" 
	ms.devlang="dotnet" 
	ms.topic="article" 
	ms.date="05/03/2016" 
	ms.author="piyushjo" />

#Azure Mobile Engagement 向け Windows ユニバーサル SDK の概要

ここから Azure Mobile Engagement を Windows ユニバーサル アプリに統合する方法についての説明を開始します。まず試してみる場合は、[15 分間チュートリアル](mobile-engagement-windows-store-dotnet-get-started.md)を完了してください。

[SDK コンテンツ](mobile-engagement-windows-store-sdk-content.md)について表示するにはここをクリックします。

##統合手順

1. ここから開始: [Windows ユニバーサル アプリに Mobile Engagement を統合する方法](mobile-engagement-windows-store-integrate-engagement.md)

2. 通知: [リーチ (通知) を Windows ユニバーサル アプリに統合する方法](mobile-engagement-windows-store-integrate-engagement-reach.md)

3. タグ付けプランの実装: [Windows ユニバーサル アプリで高度な Mobile Engagement タグ付け API を使用する方法](mobile-engagement-windows-store-use-engagement-api.md)

##リリース ノート

###3\.4.0 (04/19/2016)

-   Reach オーバーレイの機能強化。
-   SDK によって出力されるコンソール ログを有効化/無効化/フィルター処理するために "TestLogLevel" API を追加しました。
-   最初のアクティビティを対象としたアクティビティ内通知がアプリ起動時に表示されない問題を修正しました。

以前のバージョンについては、「[完全リリース ノート](mobile-engagement-windows-store-release-notes.md)」をご覧ください。

##アップグレードの手順

既にアプリケーションに以前のバージョンのモバイル エンゲージメントを統合してある場合は、SDK をアップグレードするときに、次の点を考慮する必要があります。

SDK のいくつかのバージョンがない場合は、次の手順に従う必要があります。完全な[アップグレードの手順](mobile-engagement-windows-store-upgrade-procedure.md)をご覧ください。たとえば、0.10.1 から 0.11.0 に移行する場合、まず「0.9.0から 0.10.1」への手順を実行してから「0.10.1 から 0.11.0」への手順を実行する必要があります。

###3\.3.0 から 3.4.0 に移行

####テスト ログ

SDK によって生成されるコンソール ログを有効化/無効化/フィルター処理できるようになりました。これをカスタマイズするには、次の例のように `EngagementAgent.Instance.TestLogEnabled` プロパティを `EngagementTestLogLevel` 列挙型の使用可能な値の 1 つに更新します。

			EngagementAgent.Instance.TestLogLevel = EngagementTestLogLevel.Verbose;
			EngagementAgent.Instance.Init();

####Resources

Reach オーバーレイの機能を強化しました。これは SDK NuGet パッケージのリソースの一部です。

新しいバージョンの SDK にアップグレードする際、リソースのオーバーレイ フォルダーにある既存ファイルを保持するかどうかを選択できます。

* 以前のオーバーレイが機能している、または `WebView` 要素を手動で統合している場合、既存ファイルを保持することで、引き続き使用することができます。 
* 新しいオーバーレイに更新する場合、リソースの `overlay` フォルダー全体を SDK パッケージの新しいフォルダーに置き換えます (UWP アプリ: アップグレード後に %USERPROFILE%\\.nuget\\packages\\MicrosoftAzure.MobileEngagement\\3.4.0\\content\\win81\\Resources から新しいオーバーレイ フォルダーを取得できます)。

> [AZURE.WARNING] 新しいオーバーレイを使用すると、以前のバージョンに対して行ったすべてのカスタマイズが上書きされます。

### 古いバージョンからのアップグレード

[アップグレード手順](mobile-engagement-windows-store-upgrade-procedure.md)をご覧ください

<!---HONumber=AcomDC_0504_2016-->