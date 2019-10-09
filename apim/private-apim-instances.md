## Audit for API Management instances that have direct public access

Please see the connectivity topologies on [Azure Docs](https://docs.microsoft.com/en-us/azure/api-management/api-management-using-with-vnet#-enable-vnet-connection).

```json
{
  "mode": "All",
  "policyRule": {
    "if": {
      "allOf": [
        {
          "field": "type",
          "equals": "Microsoft.ApiManagement/service"
        },
        {
          "field": "Microsoft.ApiManagement/service/virtualNetworkType",
          "notIn": [
            "Internal"
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
