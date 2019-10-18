## Configure Log Analytics workspace for metrics and logs

This policy will deploy diagnostic settings as a post-deployment step.  Since this policy will be modifying resources, it needs a Managed Service Identity to complete the changes.  The built-in roles that are required:

* [Data Factory Contributor](https://docs.microsoft.com/en-us/azure/role-based-access-control/built-in-roles#data-factory-contributor)
* [Log Analytics Contributor](https://docs.microsoft.com/en-us/azure/role-based-access-control/built-in-roles#log-analytics-contributor)

## Design Choices

**#1:** If Log Analytics workspace is kept in a separate Subscription or Resource Group, make sure to assign the policy as:

* At a management group where both Data Factory and Log Analytics can inherit the Managed Service Identity; or
* Manually add the Managed Service Identity (MSI) created to the Log Analytics workspace with the permision *Log Analytics Contributor*.  MSI is listed on the Assignment screen.  The name is the guid that's part of the assignment id.  For example:
  /subscriptions/xxxxxxxxxxxxxxxxxx/resourceGroups/xxxxxxx/providers/Microsoft.Authorization/policyAssignments/**2a2b62bc7d29476e8afe9ec9**

**#2:** The MSI created for the assignment will be automatically deleted when the assignment is removed.

**#3:** The policy sets diagnostic settings with the value provided in the parameter *diagnosticSettingsName*.  This parameter has the default value **setbypolicy** but can be customized to apply to your environment.

**#4:** Only attempts to create diagnostic settings if there isn't one setup with Log Analytics.  Policy will enable Metrics and Logs.  This can be customized in the policy by removing either Metrics or Logs sections per your environment.

**#5:** Logs will be stored in **Resource specific** tables in Log Analytics.  This is configured with *logAnalyticsDestinationType* property.

## Policy

```json
{
  "mode": "All",
  "policyRule": {
    "if": {
      "field": "type",
      "equals": "Microsoft.DataFactory/factories"
    },
    "then": {
      "effect": "DeployIfNotExists",
      "details": {
        "type": "Microsoft.Insights/diagnosticSettings",
        "existenceCondition": {
          "allOf": [
            {
              "field": "Microsoft.Insights/diagnosticSettings/workspaceId",
              "exists": "true"
            }
          ]
        },
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
                },
                "diagnosticSettingsName": {
                  "type": "string"
                }
              },
              "resources": [
                {
                  "type": "Microsoft.DataFactory/factories/providers/diagnosticsettings",
                  "name": "[concat(parameters('name'),'/Microsoft.Insights/', parameters('diagnosticSettingsName'))]",
                  "apiVersion": "2017-05-01-preview",
                  "location": "[parameters('location')]",
                  "properties": {
                    "workspaceId": "[parameters('workspaceId')]",
                    "logAnalyticsDestinationType": "Dedicated",
                    "logs": [
                      {
                        "category": "ActivityRuns",
                        "enabled": true
                      },
                      {
                        "category": "PipelineRuns",
                        "enabled": true
                      },
                      {
                        "category": "TriggerRuns",
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
              },
              "diagnosticSettingsName": {
                "value": "[parameters('diagnosticSettingsName')]"
              }
            }
          }
        },
        "roleDefinitionIds": [
          "/providers/Microsoft.Authorization/roleDefinitions/92aaf0da-9dab-42b6-94a3-d43ce8d16293",
          "/providers/Microsoft.Authorization/roleDefinitions/673868aa-7521-48a0-acc6-0f60742d39f5"
        ]
      }
    }
  },
  "parameters": {
    "workspaceId": {
      "type": "String",
      "metadata": {
        "displayName": "Log Analytics Workspace Id",
        "strongType": "omsWorkspace"
      }
    },
    "diagnosticSettingsName": {
      "type": "String",
      "metadata": {
        "displayName": "Diagnostic Settings Display Name"
      },
      "defaultValue": "setbypolicy"
    }
  }
}
```
