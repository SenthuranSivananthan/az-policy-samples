## Audit subnets without route table - exclude subnets by name

Control egress traffic from your virtual networks through Azure Firewall or 3rd party NVA such as Fortigate or Checkpoint. This policy ensures that:

* Every subnet is configured with a Route Table
* Excludes some subnets by name -There are some subnets that you may want to exclude such as the ones delegated for Azure PaaS services

### Steps

1. Identify the subnets that need to be excluded and update the **notIn** array in the policy.

### Policy

```json
{
  "mode": "All",
  "policyRule": {
    "if": {
      "allOf": [
        {
          "field": "type",
          "equals": "Microsoft.Network/virtualNetworks/subnets"
        },
        {
          "field": "name",
          "notIn": [
            "subnet-app-gw"
          ]
        },
        {
          "field": "Microsoft.Network/virtualNetworks/subnets/routeTable.id",
          "exists": "false"
        }
      ]
    },
    "then": {
      "effect": "audit"
    }
  },
  "parameters": {}
}
```


### Test

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {},
    "variables": {},
    "resources": [
        {
            "name": "expected-udr",
            "type": "Microsoft.Network/routeTables",
            "apiVersion": "2018-08-01",
            "location": "[resourceGroup().location]",
            "tags": {},
            "properties": {
                "routes": [
                    {
                        "name": "route1",
                        "properties": {
                            "addressPrefix": "0.0.0.0/0",
                            "nextHopType": "VirtualAppliance",
                            "nextHopIpAddress": "10.0.0.5"
                        }
                    }
                ],
                "disableBgpRoutePropagation": true
            }
        },
        {
            "dependsOn": [
                "Microsoft.Network/routeTables/expected-udr"
            ],
            "name": "vnet-with-expected-udr",
            "type": "Microsoft.Network/virtualNetworks",
            "apiVersion": "2018-08-01",
            "location": "[resourceGroup().location]",
            "properties": {
                "addressSpace": {
                    "addressPrefixes": [
                        "10.0.0.0/16"
                    ]
                },
                "subnets": [
                    {
                        "name": "subnet-expected-udr",
                        "properties": {
                            "addressPrefix": "10.1.0.0/24",
                            "routeTable": {
                                "id": "[resourceId('Microsoft.Network/routeTables','expected-udr')]"
                            }
                        }
                    },
                    {
                        "name": "subnet-app-gw",
                        "properties": {
                            "addressPrefix": "10.2.0.0/24"
                        }
                    },
                    {
                        "name": "subnet-database",
                        "properties": {
                            "addressPrefix": "10.3.0.0/24"
                        }
                    }
                ]
            }
        }
    ],
    "outputs": {}
}
```
