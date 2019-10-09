## Audit for SQL Databases without geo-replication

```json
{
  "mode": "All",
  "policyRule": {
    "if": {
      "allOf": [
        {
          "field": "type",
          "equals": "Microsoft.Sql/servers/disasterRecoveryConfiguration"
        },
        {
          "field": "Microsoft.Sql/servers/disasterRecoveryConfiguration/partnerServerId",
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
