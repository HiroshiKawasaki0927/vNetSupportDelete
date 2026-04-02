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

> **注意**: この手順は画面操作では実行できません。公式ドキュメントに「Enterprise Policy の環境からの解除は PowerShell でのみ可能」と記載されています。

1. PowerShell を開き、以下のモジュールをインストール・インポートする

   ```powershell
   Install-Module -Name Microsoft.PowerPlatform.EnterprisePolicies
   Import-Module Microsoft.PowerPlatform.EnterprisePolicies
   ```

2. 以下のコマンドを実行し、サブネットインジェクションを無効化する


   ```powershell
   Disable-SubnetInjection -EnvironmentId "<環境ID>"
   ```
実行例<img width="2851" height="1186" alt="image" src="https://github.com/user-attachments/assets/b54337e1-639d-470d-9f84-cefb67f46e48" />
> **注意**: Powershell 5系ではコマンドが失敗します。Powershell 7をインストールして実行します。

3. 環境 ID は [Power Platform 管理センター](https://admin.powerplatform.microsoft.com/) > **環境** > 対象環境の詳細画面で確認できる<img width="1289" height="536" alt="image" src="https://github.com/user-attachments/assets/7352b694-c418-4353-8f68-52b01e132188" />


> **注意**: 無効化の反映には数分かかる場合があります。操作完了後、Power Platform 管理センター > 対象環境 > **履歴** で状態が「成功」になっていることを確認してください。

### 手順 2: Enterprise Policy を削除する

Azure から Enterprise Policy リソースを PowerShell で削除します。

1. PowerShell で対象ポリシーを確認する

   ```powershell
   Get-SubnetInjectionEnterprisePolicy -SubscriptionId "<サブスクリプションID>" -ResourceGroupName "<リソースグループ名>"
   ```

2. 削除対象の `PolicyArmId` を指定して削除する

   ```powershell
   Remove-SubnetInjectionEnterprisePolicy -PolicyArmId "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.PowerPlatform/enterprisePolicies/<policyName>"
   ```

3. 削除後に同じ `Get-SubnetInjectionEnterprisePolicy` コマンドを実行し、対象ポリシーが表示されないことを確認する

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
| PolicyArmId | Enterprise Policy の ARM リソース ID | `Get-SubnetInjectionEnterprisePolicy` の出力、または Azure ポータル > リソースの JSON ビュー |
| サブスクリプション ID | Azure サブスクリプション ID | Azure ポータル > サブスクリプション |
| リソースグループ名 | Azure リソースグループ名 | Azure ポータル > リソースグループ |
| 仮想ネットワーク名 | 仮想ネットワーク名 | Azure ポータル > 仮想ネットワーク |
| サブネット名 | Power Platform に委任されたサブネット名 | Azure ポータル > 仮想ネットワーク > サブネット |

## 参考リンク

- [仮想ネットワークサポートの概要](https://learn.microsoft.com/ja-jp/power-platform/admin/vnet-support-overview)
- [仮想ネットワークサポートのセットアップ](https://learn.microsoft.com/ja-jp/power-platform/admin/vnet-support-setup-configure)
- [PowerPlatform-EnterprisePolicies (GitHub)](https://github.com/microsoft/PowerPlatform-EnterprisePolicies)
- [仮想ネットワークの問題のトラブルシューティング](https://learn.microsoft.com/ja-jp/troubleshoot/power-platform/administration/virtual-network)
