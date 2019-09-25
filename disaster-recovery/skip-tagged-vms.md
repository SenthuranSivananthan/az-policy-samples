## Ignore Azure to Azure ASR compliance check when VMs are tagged

### Scenarios:

* Skip Domain Controllers that are being synchronized outside of ASR
* Skip Database Servers that are using built-in replication such as Always On Availability Groups, Log Shippping, Data Guard, etc.

### Policy

#### Setup:

1. On the VMs that should be ignored from the policy check, add a tag `policy-check-skip-asr` with any value.  We only check for existence of this tag on the resource.

#### Explanation:

1. Identify ARM and Classic VMs that doesn't have the tag `policy-check-skip-asr`.  The VMs that satisfy the **if** will be considered VMs in scope for compliance check.
2. Check to see if a sub resource of type `Microsoft.Resources/links` with name **like** `ASR-Protect-*`
3. If (2) is true, then the VM is considered compliant
4. If (2) is false, then the VM is considered non-compliant

#### Notes:

1. VMs that are not satified in (Step 1) will not be shown in Compliance View 
2. The tag name `policy-check-skip-asr` is arbitary.  You can change it to anything you like in the policy.
3. The tag name can also be parameterized.  For simplicity and ease of understanding, I've set the static value.

```json
{
  "mode": "All",
  "policyRule": {
    "if": {
      "allOf": [
        {
          "field": "tags['policy-check-skip-asr']",
          "exists": "false"
        },
        {
          "field": "type",
          "in": [
            "Microsoft.Compute/virtualMachines",
            "Microsoft.ClassicCompute/virtualMachines"
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
