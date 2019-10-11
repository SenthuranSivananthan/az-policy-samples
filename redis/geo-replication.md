## Audit Redis Cache instances without geo-replication

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
          "field": "Microsoft.Cache/Redis/linkedServers[*].id",
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
