## Disable Remote Debugging on Web Apps

This policy will allow for remediation when Web Apps are deployed with Remote Debugging.  This is a deployIfNotExist policy, therefore:

* Policy is evaluated and remediated if Remote Debugging is turned on at the deployment time; OR
* Policy is evaluated and Web App is marked as non-compliant.  You will need to remediate using the **Remediation** workflow in Azure Policy.

This policy uses Managed Service Identity to make the configuration changes.  MSI will require the following role:

* [Website Contributor](https://docs.microsoft.com/en-us/azure/role-based-access-control/built-in-roles#website-contributor)

## Policy

```json
{
  "mode": "All",
  "policyRule": {
    "if": {
      "allOf": [
        {
          "field": "type",
          "equals": "Microsoft.Web/sites"
        },
        {
          "field": "kind",
          "like": "app*"
        }
      ]
    },
    "then": {
      "effect": "DeployIfNotExists",
      "details": {
        "type": "Microsoft.Web/sites/config",
        "existenceCondition": {
          "field": "Microsoft.Web/sites/config/web.remoteDebuggingEnabled",
          "equals": "false"
        },
        "roleDefinitionIds": [
          "/providers/Microsoft.Authorization/roleDefinitions/de139f84-1756-47ae-9be6-808fbbe84772"
        ],
        "deployment": {
          "properties": {
            "mode": "incremental",
            "template": {
              "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
              "contentVersion": "1.0.0.0",
              "parameters": {
                "name": {
                  "type": "string"
                },
                "location": {
                  "type": "string"
                }
              },
              "resources": [
                {
                  "type": "Microsoft.Web/sites/config",
                  "apiVersion": "2016-08-01",
                  "name": "[concat(parameters('name'), '/web')]",
                  "location": "[parameters('location')]",
                  "properties": {
                    "remoteDebuggingEnabled": false
                  }
                }
              ]
            },
            "parameters": {
              "name": {
                "value": "[field('name')]"
              },
              "location": {
                "value": "[field('location')]"
              }
            }
          }
        }
      }
    }
  },
  "parameters": {}
}
```
