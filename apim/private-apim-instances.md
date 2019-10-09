## Audit for API Management instances that have direct public access


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
