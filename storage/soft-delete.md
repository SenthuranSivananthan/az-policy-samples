## Audit for storage accounts without soft-delete

When turned on, soft delete enables you to save and recover your data when blobs or blob snapshots are deleted. This protection extends to blob data that is erased as the result of an overwrite.

When data is deleted, it transitions to a soft deleted state instead of being permanently erased. When soft delete is on and you overwrite data, a soft deleted snapshot is generated to save the state of the overwritten data. Soft deleted objects are invisible unless explicitly listed. You can configure the amount of time soft deleted data is recoverable before it is permanently expired.

See [Soft delete for Azure Storage blobs](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blob-soft-delete) for more information.

```json
{
  "mode": "All",
  "policyRule": {
    "if": {
      "field": "type",
      "equals": "Microsoft.Storage/storageAccounts"
    },
    "then": {
      "effect": "auditIfNotExists",
      "details": {
        "type": "Microsoft.Storage/storageAccounts/blobServices",
        "name": "default",
        "existenceCondition": {
          "allOf": [
            {
              "field": "Microsoft.Storage/storageAccounts/blobServices/deleteRetentionPolicy.enabled",
              "exists": "true"
            },
            {
              "field": "Microsoft.Storage/storageAccounts/blobServices/deleteRetentionPolicy.enabled",
              "equals": "true"
            }
          ]
        }
      }
    }
  },
  "parameters": {}
}
```
