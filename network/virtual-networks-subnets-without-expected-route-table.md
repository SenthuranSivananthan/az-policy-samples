## Audit virtual networks that has subnets without expected route table

Control egress traffic from your virtual networks through Azure Firewall or 3rd party NVA such as Fortigate or Checkpoint.  This policy ensures that:

1. Every subnet is configured with a Route Table
2. Route tables are from the whitelisted (typically controlled by another organization such as Network Team or InfoSec)
3. Policy #2 only - there are some virtual networks that you may want to exclude such as the ones created for Azure Site Recovery tests

### Steps

1. Identify the Route Table IDs that will be in the whitelist and replace the **notIn** array in the policy
2. Policy #2 only - identify the names of virtual networks to exclude and you can use patterns to filter

### Notes

* These policies will mark the entire virtual network as non-compliant.  For example, if the virtual network contained 5 subnets, all it would take for the virtual network to be marked non-compliant is 1 subnet that doesn't have the compliance configuration.  For per-subnet view, consider [this example](subnets-without-route-table-or-excluded-by-name.md).

### Policies
#### Policy #1

```json
{
  "mode": "All",
  "policyRule": {
    "if": {
      "allOf": [
        {
          "field": "type",
          "equals": "Microsoft.Network/virtualNetworks"
        },
        {
          "field": "Microsoft.Network/virtualNetworks/subnets[*].routeTable.id",
          "notIn": [
            "/subscriptions/___SUBSCRIPTION_ID___/resourceGroups/___RESOURCEGROUP_NAME___/providers/Microsoft.Network/routeTables/___UDR_NAME___"
          ]
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


#### Policy #2

```json
{
  "mode": "All",
  "policyRule": {
    "if": {
      "allOf": [
        {
          "field": "type",
          "equals": "Microsoft.Network/virtualNetworks"
        },
        {
          "field": "name",
          "notLike": "*-asr"
        },
        {
          "field": "Microsoft.Network/virtualNetworks/subnets[*].routeTable.id",
          "notIn": [
            "/subscriptions/___SUBSCRIPTION_ID___/resourceGroups/___RESOURCEGROUP_NAME___/providers/Microsoft.Network/routeTables/___UDR_NAME___"
          ]
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

This ARM template contains 4 uses cases that can be utilized to test the behaviour of the 2 policies.

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
            "name": "another-udr",
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
                        "name": "default",
                        "properties": {
                            "addressPrefix": "10.0.0.0/24",
                            "routeTable": {
                                "id": "[resourceId('Microsoft.Network/routeTables','expected-udr')]"
                            }
                        }
                    }
                ]
            }
        },
        {
            "dependsOn": [
                "Microsoft.Network/routeTables/another-udr"
            ],
            "name": "vnet-with-another-udr",
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
                        "name": "default",
                        "properties": {
                            "addressPrefix": "10.0.0.0/24",
                            "routeTable": {
                                "id": "[resourceId('Microsoft.Network/routeTables','another-udr')]"
                            }
                        }
                    }
                ]
            }
        },
        {
            "name": "vnet-with-no-udr",
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
                        "name": "default",
                        "properties": {
                            "addressPrefix": "10.0.0.0/24"
                        }
                    }
                ]
            }
        },
        {
            "name": "vnet-autocreated-asr",
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
                        "name": "default",
                        "properties": {
                            "addressPrefix": "10.0.0.0/24"
                        }
                    }
                ]
            }
        }
    ],
    "outputs": {}
}
```
