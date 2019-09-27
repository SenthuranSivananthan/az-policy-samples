## Export results of a policy execution based on policy name

### Steps

1. Navigate to the policy definition and identify the name field (value will be a GUID)
2. Identify the Management Group name to scope the results

```powershell
$ManagementGroupName = "Work"
$PolicyName = "e56962a6-4747-49cd-b67b-bf8b01975c4c"

$NonCompliantResources = Get-AzPolicyState `
    -ManagementGroupName $ManagementGroupName `
    -Filter "PolicyDefinitionName eq '$PolicyName' and IsCompliant eq false"

$NonCompliantResources | Select SubscriptionId, ResourceGroup, ResourceId
```
