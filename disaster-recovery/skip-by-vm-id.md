## Ignore Azure to Azure ASR compliance check using VM's Resource ID

* Skip Domain Controllers that are being synchronized outside of ASR
* Skip Database Servers that are using built-in replication such as Always On Availability Groups, Log Shippping, Data Guard, etc.

### Steps

1. Identify VMs by their resource id that needs to be excluded and update the **notIn** array.

### Disclaimer

This policy is shared to describe the approach for generating VM Resource IDs using `value`.  It is not a feasible solution when there are 100s of VMs to be ignored.  Recommendation is to use either:

1. Exclusion Lists built into Policy Assignments; or
2. [A tag to exclude VMs](skip-tagged-vms.md)

### Policy
```json
{
  "mode": "All",
  "policyRule": {
    "if": {
      "allOf": [
        {
          "field": "type",
          "in": [
            "Microsoft.Compute/virtualMachines"
          ]
        },
        {
          "value": "[concat(resourceGroup().id, '/providers/', field('type'), '/', field('name'))]",
          "notIn": [
            "/subscriptions/____SUBSCRIPTION_ID____/resourceGroups/____RESOURCEGROUP_NAME____/providers/Microsoft.Compute/virtualMachines/____VM_NAME____"
          ]
        }
      ]
    },
    "then": {
      "effect": "auditIfNotExists",
      "details": {
        "type": "Microsoft.Resources/links",
        "existenceCondition": {
          "field": "name",
          "like": "ASR-Protect-*"
        }
      }
    }
  },
  "parameters": {}
}
```
