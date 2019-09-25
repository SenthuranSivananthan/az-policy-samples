## Audit subnets that does not have one of the whitelisted route table

Control egress traffic from your virtual networks through Azure Firewall or 3rd party NVA such as Fortigate or Checkpoint.  This policy ensures that:

1. Every subnet is configured with a Route Table
2. Route tables are from the whitelisted (typically controlled by another organization such as Network Team or InfoSec)
3. Policy #2 only - there are some virtual that you may want to exclude such as the ones created for Azure Site Recovery tests

### Steps

1. Identify the Route Table IDs that will be in the whitelist and replace the **notIn** array in the policy
2. Policy #2 only - identify the names of virtual networks to exclude and you can use patterns to filter

### Policy #1

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


### Policy #2

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
