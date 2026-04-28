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
- 当前问题明显沿用上文已确认的同一 app 上下文
- 用户明确说“当前应用”“默认应用”，且没有新的业务域线索

`defaultApp` 只是默认候选。SQL 可能按业务域分布在不同 app 中；用户提到新的业务域且未指定 app 时，先把 `defaultApp` 当第一个候选验证，验证不成立再扩大到应用列表。

### 需要先做应用决议的场景

以下情况才扩大到 `lovrabet app list`：

- 用户只给了业务需求，没有明确说明是哪个应用里的 SQL
- 当前没有显式 `--appcode` / `--app`，且需求包含业务域线索
- 同一类业务可能落在多个 app 中
- 已经把 `defaultApp` 作为候选检查过，但无法确认它承载本次 SQL

推荐方式：

1. 有 `defaultApp` 时，先把它作为第一个候选
2. 无法确认默认候选承载本次 SQL 时，再 `lovrabet app list`
3. 用业务关键词选 1-2 个候选 app
4. 从用户提供信息、平台 UI、前序上下文确认这些 app 下的 SQL；需要研发态发现时，显式交接到 `rabetbase sql list`
5. 再回到 `lovrabet sql detail/exec`

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
# 0. 如果默认候选验证不成立，再看应用目录
lovrabet app list

# 1. 如果不知道 sqlcode，先从用户提供信息、平台 UI 或前序上下文获取
#    需要研发态发现时，显式交接到 rabetbase sql list

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
- `lovrabet` 没有 `sql list` 命令；sqlcode 列表来自用户提供信息、平台 UI、前序上下文，或显式研发态发现 `rabetbase sql list`
- 当业务归属不清时，先验证 `defaultApp`；验证不成立再 `app list` 做应用决议，再去确认 sqlcode，而不是直接盲猜当前 app

## 参考

- [SKILL.md](../SKILL.md)
- [instant-api-workflow.md](instant-api-workflow.md)
