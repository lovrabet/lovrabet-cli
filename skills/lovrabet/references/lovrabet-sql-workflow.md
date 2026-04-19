# SQL 查询工作流

查看和执行自定义 SQL 查询。SQL 查询在 Lovrabet 平台上创建，CLI 用于查看详情和执行。

## 前置条件

- `sql detail` 需要 Cookie 认证（`requiresCookieAuth: true`）
- `sql exec` 使用 SDK 认证，支持 **accessKey（client-ak）**

## 先判断 app 是否明确

`sql` 工作流也不要默认先查 app。

### 可以直接执行的场景

以下情况直接进入 `sql detail` / `sql exec`：

- 用户已经给了 `--appcode`
- 用户已经给了 `--app <name>`
- 当前配置已有明确 `defaultApp`
- 当前问题明显沿用当前项目 / 当前默认应用上下文

### 需要先做应用决议的场景

以下情况应先 `lovrabet app list`：

- 用户只给了业务需求，没有明确说明是哪个应用里的 SQL
- 当前没有 `defaultApp`
- 同一类业务可能落在多个 app 中

推荐方式：

1. `lovrabet app list`
2. 用业务关键词选 1-2 个候选 app
3. 到平台或 `rabetbase sql list` 中确认这些 app 下的 SQL
4. 再回到 `lovrabet sql detail/exec`

## sql detail — 查看 SQL 详情

```bash
lovrabet sql detail --sqlcode <code>

# 返回原始完整 API 响应
lovrabet sql detail --sqlcode <code> --verbose
```

| Flag | 类型 | 必填 | 说明 |
|------|------|------|------|
| `--sqlcode` | string | **是** | SQL Code，格式：`xxxxxxxx-xxxxxxxx`（8 位 hex + 8 位 hex） |
| `--verbose` | boolean | 否 | 返回完整原始对象 |

**输出**：SQL 查询名称、描述、SQL 内容、参数定义、关联的数据库等。

## sql exec — 执行 SQL 查询

```bash
# 无参数执行
lovrabet sql exec --sqlcode <code>

# 带参数执行
lovrabet sql exec --sqlcode <code> --params '{"status":"active"}'
```

| Flag | 类型 | 必填 | 说明 |
|------|------|------|------|
| `--sqlcode` | string | **是** | SQL Code，格式：`xxxxxxxx-xxxxxxxx` |
| `--params` | string | 否 | 查询参数，JSON 格式 |

**风险等级**：`read`（SQL 执行为只读操作）

**输出**：查询结果数据，格式由 `--format` 控制。

## 典型工作流

```bash
# 0. 如果当前 app 不明确，先看应用目录
lovrabet app list

# 1. 如果不知道 sqlcode，先从平台获取
#    （lrb 没有 sql list 命令，需要从 rabetbase sql list 或平台 UI 获取）

# 2. 查看 SQL 详情，了解参数结构
lovrabet sql detail --sqlcode 2305f915-dd48cd4c

# 3. 执行 SQL
lovrabet sql exec --sqlcode 2305f915-dd48cd4c --params '{"status":"active"}'

# 4. 指定 JSON 格式输出，便于程序解析
lovrabet sql exec --sqlcode 2305f915-dd48cd4c --params '{"status":"active"}' --format json
```

## 注意

- SQL Code 格式为 `{8位hex}-{8位hex}`，如 `2305f915-dd48cd4c`
- `--params` 必须是合法 JSON 字符串
- lrb 没有 `sql list` 命令，需要从平台 UI 或 `rabetbase sql list` 获取 sqlcode 列表
- 当业务归属不清时，先 `app list` 做应用决议，再去确认 sqlcode，而不是直接盲猜当前 app

## 参考

- [SKILL.md](../SKILL.md)
- [data-crud-workflow.md](data-crud-workflow.md)
