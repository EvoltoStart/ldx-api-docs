# io.net Client

This page records the minimal request example for the io.net client integration.

## Update Cluster Name

- Request URL: `https://api.io.solutions/v1/io-cloud/clusters/{cluster_id}/update-name`
- Method: `PUT`
- Replace `{cluster_id}` with the real cluster ID.

## Response Example

```json
{
  "status": "succeeded",
  "message": "Cluster name updated successfully"
}
```
