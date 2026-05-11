# io.net 客户端接口示例

本文档用于记录 `io.net` 相关接口的最小可用调用示例，便于联调与排障。

## 更新集群名称

- 请求地址：`https://api.io.solutions/v1/io-cloud/clusters/{cluster_id}/update-name`
- 请求方法：`PUT`
- 说明：将 `{cluster_id}` 替换为实际集群 ID。

### 返回示例

```json
{
  "status": "succeeded",
  "message": "Cluster name updated successfully"
}
```

