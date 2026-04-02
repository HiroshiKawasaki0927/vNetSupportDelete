# vNetSupportDelete

Steps to delete Virtual Network support created for Power Platform integration.

## Prerequisites

- PowerShell with the Microsoft.PowerPlatform.EnterprisePolicies module installed
- Azure subscription with Network Contributor role (or equivalent)
- Power Platform Administrator role in Microsoft Entra admin center

```powershell
Install-Module -Name Microsoft.PowerPlatform.EnterprisePolicies
Import-Module Microsoft.PowerPlatform.EnterprisePolicies
```

## Deletion Procedure

> **Important**: Resources must be deleted in the following order. Attempting to delete the Virtual Network before removing the Enterprise Policy will result in an InUseSubnetCannotBeDeleted error.

### Step 1: Disable Subnet Injection from the Power Platform Environment

Unlink the Enterprise Policy from the Power Platform environment.

```powershell
Disable-SubnetInjection `
  -EnvironmentId "00000000-0000-0000-0000-000000000000" `
  -PolicyArmId "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/Microsoft.PowerPlatform/enterprisePolicies/<policyName>"
```

### Step 2: Delete the Enterprise Policy

Remove the Subnet Injection Enterprise Policy from Azure.

```powershell
Remove-SubnetInjectionEnterprisePolicy `
  -PolicyArmId "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/Microsoft.PowerPlatform/enterprisePolicies/<policyName>"
```

### Step 3: Remove the Subnet Delegation

Remove the Microsoft.PowerPlatform/enterprisePolicies delegation from the subnet.

```powershell
# Get the current subnet configuration
$vnet = Get-AzVirtualNetwork -Name "<vnetName>" -ResourceGroupName "<resourceGroup>"
$subnet = Get-AzVirtualNetworkSubnetConfig -Name "<subnetName>" -VirtualNetwork $vnet

# Remove the delegation
$subnet.Delegations = @()
Set-AzVirtualNetwork -VirtualNetwork $vnet
```

### Step 4: Delete the Subnet

```powershell
$vnet = Get-AzVirtualNetwork -Name "<vnetName>" -ResourceGroupName "<resourceGroup>"
Remove-AzVirtualNetworkSubnetConfig -Name "<subnetName>" -VirtualNetwork $vnet
Set-AzVirtualNetwork -VirtualNetwork $vnet
```

### Step 5: Delete the Virtual Network

```powershell
Remove-AzVirtualNetwork -Name "<vnetName>" -ResourceGroupName "<resourceGroup>" -Force
```

## Parameters

| Parameter | Description |
|---|---|
| EnvironmentId | Power Platform environment ID |
| PolicyArmId | ARM resource ID of the Enterprise Policy |
| subscriptionId | Azure subscription ID |
| esourceGroup | Azure resource group name |
| netName | Virtual Network name |
| subnetName | Subnet name delegated to Power Platform |

## References

- [Virtual Network support overview](https://learn.microsoft.com/en-us/power-platform/admin/vnet-support-overview)
- [Set up Virtual Network support](https://learn.microsoft.com/en-us/power-platform/admin/vnet-support-setup-configure)
- [PowerPlatform-EnterprisePolicies (GitHub)](https://github.com/microsoft/PowerPlatform-EnterprisePolicies)
- [Troubleshoot Virtual Network issues](https://learn.microsoft.com/en-us/troubleshoot/power-platform/administration/virtual-network)
