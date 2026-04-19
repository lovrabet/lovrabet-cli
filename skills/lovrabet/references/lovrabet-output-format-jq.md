# 输出格式与全局 `--jq`（与 rabetbase-cli 对齐）

运行态 `lovrabet` 与研发态 `rabetbase` 在声明式管线上使用同一套约定：**结构化输出**、**compress 默认偏好**、**全局 `--jq`**。

## `--format`

| 值 | 说明 |
|----|------|
| `json` | 缩进 JSON，信封 `{ ok, command, risk, data?, ... }` |
| `compress` | 与 `json` 同一信封，**单行**紧凑，适合 Agent / 管道 |
| `pretty` | 人类可读（非 JSON 信封的展示型输出，依命令而定） |

解析顺序：**CLI `--format`** > 配置文件 `format` > 默认（产品默认多为 **compress**，以当前 CLI 版本 `--help` 为准）。

## 全局 `--jq '<expr>'`

- **作用**：在**最终要打印的整段 JSON 字符串**上执行 **jq** 表达式（与手动 `… \| jq '<expr>'` 一致）。
- **适用**：仅当本次输出走 **`--format json` 或 `compress`** 时；若当前为 `pretty` 且你传了 `--jq`，管线会将格式**升格为 json** 再跑 jq（与 rabetbase 一致）。
- **信封**：表达式作用在**完整信封**上，取业务数据多用 **`.data…`**（例如 `.data.items`、`.data.datasets[]`）。
- **二进制**：优先使用依赖 **`node-jq`** 在安装时下载到 `node_modules/node-jq/bin/` 的 jq；若无则使用 **`PATH` 上的 `jq`**。若 `npm install` 跳过了 postinstall 且系统未装 jq，需安装 jq 或设置环境变量 **`JQ_PATH`** 指向可执行文件。

## 示例

```bash
# 列表只取必要字段（compress + jq）
lovrabet dataset list --format compress --jq '.data.datasets[] | {code, name}'

# 数据集详情：投影 data 段（json）
lovrabet dataset detail --code <数据集编码> --format json --jq '.data | {fields, operations: [.operations[].name]}'
```

## 不适用 `--jq` 的命令

声明式命令里若 **`hasFormat: false`**（例如 `lovrabet doctor`），输出由命令自控，**不参与**上述 `--format` / `--jq` 管线，传 `--jq` 无意义。

## 与 rabetbase 的关系

语义与 **rabetbase-cli** 的 `framework/runner` + `formatOutput` + `apply-jq-filter` **对齐**；差别仅在于子命令集合与默认 `format` 偏好可能不同，以各自 `--help` 为准。
