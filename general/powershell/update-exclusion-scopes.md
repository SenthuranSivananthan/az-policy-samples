## Update exclusion scopes on a policy assignment

### Steps

1. Identify the Policy Assignment Resource ID.  It is visible on the Portal or through CLI/Powershell
2. Identify the Resource IDs that needs to be excluded.
3. Revise the PS script to include your configuration

### Notes

1. One Policy assignment can have up to 400 exclusions only.  [See Policy limits reference](https://docs.microsoft.com/en-us/azure/azure-subscription-service-limits#azure-policy-limits)

```powershell

$PolicyAssignmentId = "/providers/Microsoft.Management/managementGroups/Senthuran/providers/Microsoft.Authorization/policyAssignments/90810c5368f84bf09d37551b"

$Assignment = Get-AzPolicyAssignment -Id $PolicyAssignmentId

$NotScope = [System.Collections.ArrayList]@($Assignment.Properties.notScopes)
$NotScope.Add("/subscriptions/_____SUBSCRIPTION_ID_____/resourceGroups/_____RESOURCEGROUP_NAME_____/providers/Microsoft.Compute/virtualMachines/vm1")
$NotScope.Add("/subscriptions/_____SUBSCRIPTION_ID_____/resourceGroups/_____RESOURCEGROUP_NAME_____/providers/Microsoft.Compute/virtualMachines/vm2")

Set-AzPolicyAssignment -Id $PolicyAssignmentId -NotScope $NotScope

```
