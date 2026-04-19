# BFF 脚本工作流

查看和执行 BFF（Backend For Frontend）脚本。BFF 脚本在 Lovrabet 平台上创建，CLI 用于查看详情和执行。

## 前置条件

- `bff detail` 需要 Cookie 认证（`requiresCookieAuth: true`）
- `bff exec` 使用 SDK 认证，支持 **accessKey（client-ak）**

## 先判断 app 是否明确

`bff` 工作流和 `dataset/sql` 一样，不要默认先跑 `app list`，也不要完全跳过 app 决议。

### 可以直接执行的场景

以下情况直接进入 `bff detail` / `bff exec`：

- 用户已经给了 `--appcode`
- 用户已经给了 `--app <name>`
- 当前配置已有明确 `defaultApp`
- 当前任务明显沿用当前项目 / 当前默认应用上下文

### 需要先做应用决议的场景

以下情况应先 `lovrabet app list`：

- 用户只描述了业务能力，没有说是哪个应用里的 BFF
- 当前没有 `defaultApp`
- 候选 app 可能不止一个

推荐方式：

1. `lovrabet app list`
2. 用业务关键词挑候选 app
3. 到平台或 `rabetbase bff list` 确认对应脚本
4. 再执行 `lovrabet bff detail/exec`

## bff detail — 查看 BFF 详情

```bash
lovrabet bff detail --id <scriptId>

# 返回原始完整 API 响应
lovrabet bff detail --id <scriptId> --verbose
```

| Flag | 类型 | 必填 | 说明 |
|------|------|------|------|
| `--id` | number | **是** | BFF 脚本 ID |
| `--verbose` | boolean | 否 | 返回完整原始对象 |

**输出**：脚本名称、描述、类型（ENDPOINT/COMMON）、脚本内容等。

## bff exec — 执行 BFF 函数

```bash
# 无参数执行
lovrabet bff exec --name <functionName>

# 带参数执行
lovrabet bff exec --name <functionName> --params '{"key":"value"}'
```

| Flag | 类型 | 必填 | 说明 |
|------|------|------|------|
| `--name` | string | **是** | BFF 函数名（`export default function` 后的函数名） |
| `--params` | string | 否 | 执行参数，JSON 格式 |

**风险等级**：`read`（BFF 执行为只读操作）

**输出**：BFF 函数返回值，格式由 `--format` 控制。

## 典型工作流

```bash
# 0. 如果当前 app 不明确，先看应用目录
lovrabet app list

# 1. 如果不知道脚本 ID，先从平台获取
#    （lrb 没有 bff list 命令，需要从 rabetbase bff list 或平台 UI 获取）

# 2. 查看 BFF 详情，了解参数结构
lovrabet bff detail --id 42

# 3. 执行 BFF
lovrabet bff exec --name getUserInfo --params '{"userId":123}'

# 4. 指定 JSON 格式输出
lovrabet bff exec --name getUserInfo --params '{"userId":123}' --format json
```

## 注意

- `--name` 即 BFF 中 `export default function` 后的函数名（`scriptName`），精确匹配
- `--params` 必须是合法 JSON 字符串
- lrb 没有 `bff list` 命令，需要从平台 UI 或 `rabetbase bff list` 获取脚本列表和 ID
- 当业务归属不清时，先 `app list` 做应用决议，再去确认脚本，而不是直接盲猜当前 app

## 参考

- [SKILL.md](../SKILL.md)
- [data-crud-workflow.md](data-crud-workflow.md)
