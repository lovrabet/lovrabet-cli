# personal BFF 工作流

`personal-bff` 用于当前用户在当前应用下维护个人 ENDPOINT 脚本。第一阶段支持 `list`、`detail`、`create`、`update`、`exec`；不提供删除命令。

## 什么时候用

- Artifact 需要运行时自定义数据编排
- 需要把多步只读查询包装成稳定函数
- 需要先验证一个轻量业务接口的返回形状，再交给 Artifact 展示

若已有平台正式 BFF，优先复用 `bff detail/exec`。personal BFF 更适合个人工作流、验证和 Agent 生成的轻量端点。

## 查看

```bash
lovrabet personal-bff list --format compress
lovrabet personal-bff detail --id <id> --format compress
```

更新前先 `personal-bff detail`，确认函数名、脚本内容、描述和版本。

## 创建

脚本来自本地 UTF-8 JavaScript 或 TypeScript 文件。`--name` 必须是合法 JavaScript identifier。

```bash
lovrabet personal-bff create --name loadOrders --file ./load-orders.js --dry-run
lovrabet personal-bff create --name loadOrders --file ./load-orders.js
```

可选：

```bash
lovrabet personal-bff create \
  --name loadOrders \
  --description "Load active orders for dashboard" \
  --source-session-id <sessionId> \
  --file ./load-orders.js \
  --dry-run
```

## 更新

```bash
lovrabet personal-bff detail --id <id> --format compress
lovrabet personal-bff update --id <id> --file ./load-orders-v2.js --dry-run
lovrabet personal-bff update --id <id> --name loadActiveOrders --description "v2" --file ./load-orders-v2.js
```

`update` 至少要提供 `--name`、`--description`、`--file` 或 `--source-session-id` 之一。

## 执行

`exec` 会运行用户脚本，可能有副作用，因此是 `high-risk-write`。非交互场景必须显式确认：

```bash
lovrabet personal-bff exec --id <id> --params '{"status":"active"}' --yes --format compress
```

不传 `--params` 时默认 `{}`。传入值必须是 JSON 对象。

## 与 Artifact 配合

写 BFF-backed Artifact 前先执行 personal BFF，并保存返回形状摘要：

```bash
lovrabet personal-bff exec --id <id> --params '{"status":"active"}' --yes --format compress
```

确认字段、空态和错误形状后，再把 Artifact 源码写成本地 React 模块，并通过 `artifact create/update` 持久化。
