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

## React module host contract

每个 Artifact source 都必须保持以下 host contract。

1. 每次 create 或 update 只能产出一个 `react_module` Artifact。
2. source 只能 render 一个 host `Container`，不能嵌套 `Container`。
3. 不能 import 或 render antd `Card`，source 中必须没有 `<Card`。
4. 只能有一个 default export React component。
5. default component 必须接收 `props: { env: JsxModulePropsEnv }`。
6. source 必须保留精确 destructuring 语句：`const { Container, sdkClient, navigate } = env;`。
7. source 不能包含 `html`、`body`、`script` 或 `style` tags。
8. 不能调用 `ReactDOM`、`ReactDOM.render` 或 `createRoot`。
9. 只能从 `react`、`react-dom`、`lodash`、`dayjs`、`antd`、`@ant-design/icons` import。
10. Artifact 名称使用英文 PascalCase，例如 `SalesInsightCard`、`ProjectStatusCard`、`ComparisonCard`、`WeeklyOpsCard`。

```tsx
import React from "react";
import { Button, Typography } from "antd";

interface ContainerProps {
  className?: string;
  style?: React.CSSProperties;
  title?: string;
  extra?: string | React.ReactNode;
  children?: React.ReactNode;
}

interface JsxModulePropsEnv {
  Container: React.FC<ContainerProps>;
  sdkClient: any;
  navigate: (to: string) => void;
}

export default function ExampleCard({ env }: { env: JsxModulePropsEnv }) {
  const { Container, sdkClient, navigate } = env;

  return (
    <Container title="模块标题">
      <Button type="link" onClick={() => navigate("/chat")}>
        Chat
      </Button>
      <Typography.Text>这里是 module 内容</Typography.Text>
    </Container>
  );
}
```

## personal BFF backed Artifact

当 Artifact 需要自定义运行时数据编排时，先创建或复用 personal BFF，并执行一次确认返回形状，再写 Artifact：

```bash
lovrabet personal-bff exec --id <id> --params '{"status":"active"}' --yes --format compress
lovrabet artifact create --file ./orders-artifact.tsx --name "Orders" --dry-run
```

Artifact 源码应调用宿主 SDK 的 personal BFF 能力，例如 `sdkClient.personalBff.execute({ scriptId, params })`。不要在源码里硬编码运行态域名、cookie、accessKey 或 appCode。

## 保存后观测

正式 `lovrabet artifact create` 或 `lovrabet artifact update` 保存成功后调用 `runtime-observe` 观测保存后的 Artifact。`runtime-observe` 是 sandbox helper，不是 `lovrabet` CLI 命令。只在保存成功后观测；dry-run 成功时不观测。

保存成功后调用 runtime-observe。Artifact debug URL 需要使用应用域名 `appDomain` 和保存后的 `artifactId`：

```text
daily: https://{appDomain}.daily.lovrabet.com/__debug__?artifactid={artifactId}
prod:  https://{appDomain}.lovrabet.com/__debug__?artifactid={artifactId}
```

如果 create/update/detail 输出里没有 `appDomain`，先尝试 `lovrabet app list --format compress` 或已有上下文确认当前应用域名；仍无法确认时，明确说明缺少 appDomain，观测未完成。
