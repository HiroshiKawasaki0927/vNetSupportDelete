# How to Delete Virtual Network Support

When attempting to delete a Virtual Network used for Power Platform integration directly from the Azure portal, you may encounter the following error:

```error
Failed to delete virtual network '<vnetName>'. Error: Subnet default is in use by /subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Network/virtualNetworks/<vnetName>/subnets/default/serviceAssociationLinks/PowerPlatformServiceLink and cannot be deleted. To delete the subnet, delete all resources within the subnet. See aka.ms/deletesubnet.
```

## Prerequisites

- Access to the Azure portal with the Network Contributor role (or equivalent)
- Access to the Power Platform admin center with the Power Platform Administrator role
- Access to the Microsoft Entra admin center

## Deletion Procedure

> **Important**: Resources must be deleted in the following order. Attempting to delete the Virtual Network before removing the Enterprise Policy will result in an `InUseSubnetCannotBeDeleted` error.

### Step 1: Disable Subnet Injection from the Power Platform Environment

Unlink the Enterprise Policy from the Power Platform environment.

> **Note**: This step cannot be performed through the GUI. According to the official documentation, removing an Enterprise Policy from an environment is only possible through PowerShell.

1. Open PowerShell and install/import the module

   ```powershell
   Install-Module -Name Microsoft.PowerPlatform.EnterprisePolicies
   Import-Module Microsoft.PowerPlatform.EnterprisePolicies
   ```

2. Run the following command to disable subnet injection

   ```powershell
   Disable-SubnetInjection -EnvironmentId "<EnvironmentId>"
   ```

3. You can find the Environment ID in the [Power Platform admin center](https://admin.powerplatform.microsoft.com/) > **Environments** > select the target environment > details page

> **Note**: It may take a few minutes for the change to take effect. After completion, verify the status shows "Succeeded" in the Power Platform admin center > target environment > **History**.

### Step 2: Verify the Enterprise Policy

Identify the target Enterprise Policy using Azure Resource Graph Explorer.

1. Sign in to the [Azure portal](https://portal.azure.com/)
2. Type **Resource Graph Explorer** in the search bar at the top and select it
3. Enter the following KQL query in the query editor

   ```kql
   resources
   | where type == "microsoft.powerplatform/enterprisepolicies"
   | project name, kind, resourceGroup, location, subscriptionId, id, properties
   ```

4. Click **Run query**
5. Identify the target Enterprise Policy from the results
   - The `id` column is the `PolicyResourceId` (used in Step 3 for deletion)
   - Entries with `kind` of `NetworkInjection` are subnet injection Enterprise Policies

> **Tip**: To filter by a specific subscription or resource group, add `where` clauses:
>
> ```kql
> resources
> | where type == "microsoft.powerplatform/enterprisepolicies"
> | where subscriptionId == "<SubscriptionId>"
> | where resourceGroup == "<ResourceGroupName>"
> | project name, kind, resourceGroup, location, id, properties
> ```

### Step 3: Delete the Enterprise Policy

Delete the Enterprise Policy resource from Azure by using PowerShell.

1. List the policies in the target resource group

   ```powershell
   Get-SubnetInjectionEnterprisePolicy -SubscriptionId "<SubscriptionId>" -ResourceGroupName "<ResourceGroupName>"
   ```

2. Delete the target policy by specifying its `PolicyResourceId`

   ```powershell
   Remove-SubnetInjectionEnterprisePolicy -PolicyResourceId "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.PowerPlatform/enterprisePolicies/<policyName>"
   ```

3. Run `Get-SubnetInjectionEnterprisePolicy` again and confirm the target policy is no longer listed

### Step 4: Remove the Subnet Delegation

Remove the `Microsoft.PowerPlatform/enterprisePolicies` delegation from the subnet.

1. Sign in to the [Azure portal](https://portal.azure.com/)
2. Type **Virtual networks** in the search bar at the top and select **Virtual networks**
3. Click the target virtual network
4. Select **Subnets** from the left menu
5. Click the delegated subnet
6. Select **None** from the **Subnet delegation** dropdown
7. Click **Save**

### Step 5: Delete the Subnet

1. Open the **Subnets** page of the same virtual network from Step 4
2. Right-click the subnet row to delete, or click the **...** menu
3. Select **Delete**
4. Click **Yes** in the confirmation dialog

### Step 6: Delete the Virtual Network

1. Sign in to the [Azure portal](https://portal.azure.com/)
2. Type **Virtual networks** in the search bar at the top and select **Virtual networks**
3. Check the box next to the virtual network to delete
4. Click **Delete** in the top menu
5. Click **Delete** in the confirmation dialog

> **Note**: If other subnets or resources remain in the virtual network, delete them first.

## Required Information

| Item | Description | Where to Find |
|---|---|---|
| Environment ID | Power Platform environment ID | Power Platform admin center > Environments > Target environment details |
| PolicyResourceId | ARM resource ID of the Enterprise Policy | Output of `Get-SubnetInjectionEnterprisePolicy`, `id` column in Resource Graph Explorer, or Azure portal > resource JSON view |
| Subscription ID | Azure subscription ID | Azure portal > Subscriptions |
| Resource Group Name | Azure resource group name | Azure portal > Resource groups |
| Virtual Network Name | Virtual network name | Azure portal > Virtual networks |
| Subnet Name | Subnet name delegated to Power Platform | Azure portal > Virtual networks > Subnets |

## References

- [Virtual Network support overview](https://learn.microsoft.com/en-us/power-platform/admin/vnet-support-overview)
- [Set up Virtual Network support](https://learn.microsoft.com/en-us/power-platform/admin/vnet-support-setup-configure)
- [PowerPlatform-EnterprisePolicies (GitHub)](https://github.com/microsoft/PowerPlatform-EnterprisePolicies)
- [Troubleshoot Virtual Network issues](https://learn.microsoft.com/en-us/troubleshoot/power-platform/administration/virtual-network)
