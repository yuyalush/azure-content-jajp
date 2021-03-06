<properties
	pageTitle="Azure SQL Database の一般的な管理タスクを実行する方法"
	description="一般的な管理タスクを実行する方法について説明します"
	services="sql-database"
	documentationCenter=""
	authors="v-shysun"
	manager="felixwu"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.workload="data-management"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="06/03/2016"
	ms.author="v-shysun"/>

# Azure SQL Database の一般的な管理タスクを実行する方法
Azure SQL Database へのアクセスの付与および削除を行う簡単な手順については、このトピックをご覧ください。より包括的な情報については、次を参照してください。

- [Azure SQL Database におけるデータベースとログインの管理](sql-database-manage-logins.md)
- [SQL Database の保護](sql-database-security.md)
- [SQL Server Database エンジンと Azure SQL Database のセキュリティ センター](https://msdn.microsoft.com/library/bb510589)


[AZURE.INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]


## 論理サーバーの管理パスワードを変更するには


- [Azure ポータル](https://portal.azure.com)で **[SQL Server]** をクリックし、一覧からサーバーを選択して、**[パスワードのリセット]** をクリックします。

## 権限のある IP アドレスのみがサーバーへのアクセスを許可されるようにするには
- 「[方法: ファイアウォール設定を構成する (SQL Database)](sql-database-configure-firewall-settings.md)」を参照してください。

## ユーザー データベースに包含データベース ユーザーを作成するには
- [CREATE USER](https://msdn.microsoft.com/library/ms173463.aspx) ステートメントを使用し、「[包含データベース ユーザー - データベースの可搬性を確保する](https://msdn.microsoft.com/library/ff929188.aspx)」をご覧ください。

## Azure Active Directory を使用して包含データベース ユーザーを認証するには
- 「[Azure Active Directory 認証を使用して SQL Database または SQL Data Warehouse に接続する](sql-database-aad-authentication.md)」をご覧ください。

## 仮想マスター データベースで高い特権を持つユーザーの追加ログインを作成するには
- [CREATE LOGIN](https://msdn.microsoft.com/library/ms189751.aspx) ステートメントを使用し、[Azure SQL Database でのデータベースとログインの管理](sql-database-manage-logins.md)に関するページのログイン管理セクションで詳細を確認してください。

<!---HONumber=AcomDC_0615_2016-->