## Audit for SQL Databases without geo-replication - work in progress

```json
{
  "mode": "All",
  "policyRule": {
    "if": {
      "allOf": [
        {
          "field": "type",
          "equals": "Microsoft.Sql/servers"
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
