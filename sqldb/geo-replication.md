## Audit for SQL Databases without geo-replication

```json
{
  "mode": "All",
  "policyRule": {
    "if": {
      "field": "type",
      "equals": "Microsoft.Sql/servers/databases"
    },
    "then": {
      "effect": "auditIfNotExists",
      "details": {
        "type": "Microsoft.Sql/servers/databases/replicationLinks"
      }
    }
  },
  "parameters": {}
}
```
