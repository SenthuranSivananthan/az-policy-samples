## Audit Redis Cache instances without virtual network isolation

```json
{
  "mode": "All",
  "policyRule": {
    "if": {
      "allOf": [
        {
          "field": "type",
          "equals": "Microsoft.Cache/redis"
        },
        {
          "field": "Microsoft.Cache/Redis/subnetId",
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
