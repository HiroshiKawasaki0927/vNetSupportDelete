# 仮想ネットワークサポートの削除方法

Power Platform 連携で利用する仮想ネットワークサポートですが、Azure portalから直接削除しようとすると以下のエラーが発生します。

```error
仮想ネットワーク '<vnetName>' を削除できませんでした。エラー: サブネット default は、/subscriptions/<subscriptionId>/resourceGroups/<resourceGroups/Name>/providers/Microsoft.Network/virtualNetworks/<vnetName>/subnets/default/serviceAssociationLinks/PowerPlatformServiceLink で使用中のため、削除できません。サブネットを削除するには、サブネット内のすべてのリソースを削除します。aka.ms/deletesubnet を参照してください。
```

## 前提条件

- Azure ポータルへのアクセス権限（ネットワーク共同作成者ロールまたは同等の権限）
- Power Platform 管理センターへのアクセス権限（Power Platform 管理者ロール）
- Microsoft Entra 管理センターへのアクセス権限

## 削除手順

> **重要**: リソースは以下の順序で削除する必要があります。Enterprise Policy を削除する前に仮想ネットワークを削除しようとすると、`InUseSubnetCannotBeDeleted` エラーが発生します。

### 手順 1: Power Platform 環境からサブネットインジェクションを無効化する

Power Platform 環境から Enterprise Policy のリンクを解除します。

1. [Power Platform 管理センター](https://admin.powerplatform.microsoft.com/) にサインインする
2. 左メニューから **環境** を選択する
3. 対象の環境を選択する
4. **設定** > **製品** > **プライバシー + セキュリティ** を開く
5. **仮想ネットワーク** セクションで、リンクされている Enterprise Policy を確認する
6. **無効化** または **リンク解除** を選択する
7. 確認ダイアログで **確認** をクリックする

> **注意**: 無効化の反映には数分かかる場合があります。

### 手順 2: Enterprise Policy を削除する

Azure から Enterprise Policy リソースを削除します。

1. [Azure ポータル](https://portal.azure.com/) にサインインする
2. 上部の検索バーに **Enterprise Policies** と入力し、検索する
   - または、**リソースグループ** > 対象のリソースグループ > Enterprise Policy リソースを選択する
3. 削除する Enterprise Policy リソースをクリックする
4. 上部メニューの **削除** をクリックする
5. 確認ダイアログでリソース名を入力し、**削除** をクリックする

### 手順 3: サブネット委任を解除する

サブネットから `Microsoft.PowerPlatform/enterprisePolicies` の委任を解除します。

1. [Azure ポータル](https://portal.azure.com/) にサインインする
2. 上部の検索バーに **仮想ネットワーク** と入力し、**仮想ネットワーク** を選択する
3. 対象の仮想ネットワークをクリックする
4. 左メニューから **サブネット** を選択する
5. 委任されているサブネットをクリックする
6. **サブネットの委任** ドロップダウンから **None** を選択する
7. **保存** をクリックする

### 手順 4: サブネットを削除する

1. 手順 3 と同じ仮想ネットワークの **サブネット** 画面を開く
2. 削除するサブネットの行を右クリック、または **...** メニューをクリックする
3. **削除** を選択する
4. 確認ダイアログで **はい** をクリックする

### 手順 5: 仮想ネットワークを削除する

1. [Azure ポータル](https://portal.azure.com/) にサインインする
2. 上部の検索バーに **仮想ネットワーク** と入力し、**仮想ネットワーク** を選択する
3. 削除する仮想ネットワークの行にチェックを入れる
4. 上部メニューの **削除** をクリックする
5. 確認ダイアログで **削除** をクリックする

> **注意**: 仮想ネットワークに他のサブネットやリソースが残っている場合は、先にそれらを削除してください。

## パラメータ（確認が必要な情報）

| 項目 | 説明 | 確認方法 |
|---|---|---|
| 環境 ID | Power Platform 環境 ID | Power Platform 管理センター > 環境 > 対象環境の詳細 |
| Enterprise Policy 名 | Enterprise Policy の Azure リソース名 | Azure ポータル > リソースグループ > 該当リソース |
| サブスクリプション ID | Azure サブスクリプション ID | Azure ポータル > サブスクリプション |
| リソースグループ名 | Azure リソースグループ名 | Azure ポータル > リソースグループ |
| 仮想ネットワーク名 | 仮想ネットワーク名 | Azure ポータル > 仮想ネットワーク |
| サブネット名 | Power Platform に委任されたサブネット名 | Azure ポータル > 仮想ネットワーク > サブネット |

## 参考リンク

- [仮想ネットワークサポートの概要](https://learn.microsoft.com/ja-jp/power-platform/admin/vnet-support-overview)
- [仮想ネットワークサポートのセットアップ](https://learn.microsoft.com/ja-jp/power-platform/admin/vnet-support-setup-configure)
- [PowerPlatform-EnterprisePolicies (GitHub)](https://github.com/microsoft/PowerPlatform-EnterprisePolicies)
- [仮想ネットワークの問題のトラブルシューティング](https://learn.microsoft.com/ja-jp/troubleshoot/power-platform/administration/virtual-network)
