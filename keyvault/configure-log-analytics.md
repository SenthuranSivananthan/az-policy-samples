## Configure Log Analytics workspace for metrics and logs

This policy will deploy diagnostic settings as a post-deployment step.  Since this policy will be modifying resources, it needs a Managed Service Identity to complete the changes.  The roles that are required:

* Key Vault Contributor
* Log Analytics Contributor

**Note:** If Log Analytics workspace is kept in a separate Subscription or Resource Group, make sure to assign the policy as:

* At a management group where both Key Vault and Log Analytics can inherit the Managed Service Identity; or
* Manually add the Managed Service Identity (MSI) created to the Log Analytics workspace with the permision *Log Analytics Contributor*.  MSI is listed on the Assignment screen once it's saved.

```json
{
  "mode": "All",
  "policyRule": {
    "if": {
      "field": "type",
      "equals": "Microsoft.KeyVault/vaults"
    },
    "then": {
      "effect": "DeployIfNotExists",
      "details": {
        "type": "Microsoft.Insights/diagnosticSettings",
        "name": "setbypolicy",
        "deployment": {
          "properties": {
            "mode": "incremental",
            "template": {
              "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
              "contentVersion": "1.0.0.0",
              "parameters": {
                "workspaceId": {
                  "type": "string"
                },
                "name": {
                  "type": "string"
                },
                "location": {
                  "type": "string"
                }
              },
              "resources": [
                {
                  "type": "Microsoft.KeyVault/vaults/providers/diagnosticsettings",
                  "name": "[concat(parameters('name'),'/Microsoft.Insights/setbypolicy')]",
                  "apiVersion": "2017-05-01-preview",
                  "location": "[parameters('location')]",
                  "properties": {
                    "workspaceId": "[parameters('workspaceId')]",
                    "logs": [
                      {
                        "category": "AuditEvent",
                        "enabled": true
                      }
                    ],
                    "metrics": [
                      {
                        "category": "AllMetrics",
                        "enabled": true
                      }
                    ]
                  }
                }
              ]
            },
            "parameters": {
              "workspaceId": {
                "value": "[parameters('workspaceId')]"
              },
              "name": {
                "value": "[field('name')]"
              },
              "location": {
                "value": "[field('location')]"
              }
            }
          }
        },
        "roleDefinitionIds": [
          "/providers/Microsoft.Authorization/roleDefinitions/f25e0fa2-a7c8-4377-a976-54943a77a395",
          "/providers/Microsoft.Authorization/roleDefinitions/92aaf0da-9dab-42b6-94a3-d43ce8d16293"
        ]
      }
    }
  },
  "parameters": {
    "workspaceId": {
      "type": "String",
      "metadata": {
        "displayName": "Log Analytics Workspace Id",
        "description": null,
        "strongType": "omsWorkspace"
      }
    }
  }
}
```
