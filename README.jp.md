# vNetSupportDelete

Power Platform 連携用に作成した仮想ネットワークサポートを削除する手順です。

## 前提条件

- `Microsoft.PowerPlatform.EnterprisePolicies` モジュールがインストールされた PowerShell
- ネットワーク共同作成者ロール（または同等の権限）を持つ Azure サブスクリプション
- Microsoft Entra 管理センターでの Power Platform 管理者ロール

```powershell
Install-Module -Name Microsoft.PowerPlatform.EnterprisePolicies
Import-Module Microsoft.PowerPlatform.EnterprisePolicies
```

## 削除手順

> **重要**: リソースは以下の順序で削除する必要があります。Enterprise Policy を削除する前に仮想ネットワークを削除しようとすると、`InUseSubnetCannotBeDeleted` エラーが発生します。

### 手順 1: Power Platform 環境からサブネットインジェクションを無効化する

Power Platform 環境から Enterprise Policy のリンクを解除します。

```powershell
Disable-SubnetInjection `
  -EnvironmentId "00000000-0000-0000-0000-000000000000" `
  -PolicyArmId "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/Microsoft.PowerPlatform/enterprisePolicies/<policyName>"
```

### 手順 2: Enterprise Policy を削除する

Azure からサブネットインジェクション Enterprise Policy を削除します。

```powershell
Remove-SubnetInjectionEnterprisePolicy `
  -PolicyArmId "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/Microsoft.PowerPlatform/enterprisePolicies/<policyName>"
```

### 手順 3: サブネット委任を解除する

サブネットから `Microsoft.PowerPlatform/enterprisePolicies` の委任を解除します。

```powershell
# 現在のサブネット構成を取得
$vnet = Get-AzVirtualNetwork -Name "<vnetName>" -ResourceGroupName "<resourceGroup>"
$subnet = Get-AzVirtualNetworkSubnetConfig -Name "<subnetName>" -VirtualNetwork $vnet

# 委任を解除
$subnet.Delegations = @()
Set-AzVirtualNetwork -VirtualNetwork $vnet
```

### 手順 4: サブネットを削除する

```powershell
$vnet = Get-AzVirtualNetwork -Name "<vnetName>" -ResourceGroupName "<resourceGroup>"
Remove-AzVirtualNetworkSubnetConfig -Name "<subnetName>" -VirtualNetwork $vnet
Set-AzVirtualNetwork -VirtualNetwork $vnet
```

### 手順 5: 仮想ネットワークを削除する

```powershell
Remove-AzVirtualNetwork -Name "<vnetName>" -ResourceGroupName "<resourceGroup>" -Force
```

## パラメータ

| パラメータ | 説明 |
|---|---|
| `EnvironmentId` | Power Platform 環境 ID |
| `PolicyArmId` | Enterprise Policy の ARM リソース ID |
| `subscriptionId` | Azure サブスクリプション ID |
| `resourceGroup` | Azure リソースグループ名 |
| `vnetName` | 仮想ネットワーク名 |
| `subnetName` | Power Platform に委任されたサブネット名 |

## 参考リンク

- [仮想ネットワークサポートの概要](https://learn.microsoft.com/ja-jp/power-platform/admin/vnet-support-overview)
- [仮想ネットワークサポートのセットアップ](https://learn.microsoft.com/ja-jp/power-platform/admin/vnet-support-setup-configure)
- [PowerPlatform-EnterprisePolicies (GitHub)](https://github.com/microsoft/PowerPlatform-EnterprisePolicies)
- [仮想ネットワークの問題のトラブルシューティング](https://learn.microsoft.com/ja-jp/troubleshoot/power-platform/administration/virtual-network)
