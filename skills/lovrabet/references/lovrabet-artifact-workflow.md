# Artifact 工作流

`artifact` 命令用于管理当前应用下由 Agent 生成的 `react_module` Artifact。第一阶段只支持 `list`、`detail`、`create`、`update`；Artifact 删除由产品界面管理。

## 发现优先

如果 Artifact 要展示真实业务数据，且用户没有直接提供字段和值，先做只读发现：

```bash
lovrabet api-doc list --category dataset
lovrabet api-doc detail --code dataset_data_filter
lovrabet dataset list --name "<业务关键词>"
lovrabet dataset detail --code <datasetCode>
lovrabet dataset sdk-doc --code <datasetCode>
lovrabet data filter --code <datasetCode> --params '{"currentPage":1,"pageSize":5}' --format compress
```

已经有明确数据时，可以直接把用户给定的数据写入 Artifact 源码，不必重复发现。

## 查看

```bash
lovrabet artifact list --source AGENT --format compress
lovrabet artifact detail --id <id> --format compress
```

更新前必须先 `artifact detail`，确认现有源码、metadata、owner 语境和最近更新时间。

## 创建

Artifact 源码来自本地 UTF-8 文件，文件应导出可嵌入的 React 模块：

```bash
lovrabet artifact create --file ./orders-artifact.tsx --name "Orders" --dry-run
lovrabet artifact create --file ./orders-artifact.tsx --name "Orders"
```

CLI 固定写入：

- `artifactType = react_module`
- `source = AGENT`

不要传 `compiledContent`。`--metadata` 只接受 JSON 对象，其中的 `compiledContent` 会被剔除。

## 更新

```bash
lovrabet artifact detail --id <id> --format compress
lovrabet artifact update --id <id> --file ./orders-artifact.tsx --dry-run
lovrabet artifact update --id <id> --file ./orders-artifact.tsx
```

## 源码边界

CLI 会拒绝以下内容：

- 空文件
- 完整 HTML 页面
- `ReactDOM.render`、`ReactDOM.createRoot` 或 `createRoot(...)` 挂载入口
- 不在宿主允许列表内的 import

当前允许的包：`react`、`react-dom`、`lodash`、`dayjs`、`antd`、`@ant-design/icons`。

## personal BFF backed Artifact

当 Artifact 需要自定义运行时数据编排时，先创建或复用 personal BFF，并执行一次确认返回形状，再写 Artifact：

```bash
lovrabet personal-bff exec --id <id> --params '{"status":"active"}' --yes --format compress
lovrabet artifact create --file ./orders-artifact.tsx --name "Orders" --dry-run
```

Artifact 源码应调用宿主 SDK 的 personal BFF 能力，例如 `sdkClient.personalBff.execute({ scriptId, params })`。不要在源码里硬编码运行态域名、cookie、accessKey 或 appCode。
