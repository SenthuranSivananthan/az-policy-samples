

```powershell
$PolicySet = Get-AzPolicySetDefinition -Id "__INITIATIVE_DEFINITION_ID__"

foreach ($PolicyDefinition in $PolicySet.Properties.policyDefinitions)
{
    $definition = Get-AzPolicyDefinition -Id $PolicyDefinition.policyDefinitionId

    $AssignmentName = "__ASSIGNMENT__PREFIX__ - " + $definition.Properties.displayName

    New-AzPolicyAssignment -Name (New-Guid).ToString().Replace('-','').ToLower().Substring(0, 24) `
        -DisplayName $AssignmentName `
        -PolicyDefinition $definition `
        -Scope "/providers/Microsoft.Management/managementGroups/____MANAGEMENT_GROUP_NAME____"
    $PolicyDefinition.policyDefinitionId
}
```
