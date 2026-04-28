# Instant API Workflow

## 概述

通过运行态 CLI 对数据集记录进行增删改查和聚合统计。`Instant API` 分组下的所有 `data` 子命令共享统一接口：

```bash
lovrabet data <command> --code <datasetCode> --params '<json>'
```

- `--code`：数据集 code（路由标识，必填）
- `--params`：JSON 请求体，直接透传给运行态 API

## data 之前如何确定 app

`data` 子命令本身只依赖 `dataset code`，但这个 `dataset code` 总是来自某个具体 app。

因此决策顺序应该是：

1. 先判断当前 app 是否已明确
2. 当前 app 明确时，直接 `dataset list --name ...`
3. 当前 app 不明确但有 `defaultApp` 时，先 `dataset list --name <关键词>` 验证默认候选
4. 默认候选无命中、弱命中或语义不合理时，再 `app list`
5. 再用 `dataset list --app <name> --name <关键词>` 收敛到正确 app
6. 拿到 `dataset code` 后，再执行 `data filter/getOne/create/update/delete`

不要直接在 app 未决议的情况下盲目构造 `data` 命令。

## 架构说明

6 个 data 子命令对应 6 个运行态 API 端点：

| 子命令 | 说明 | 风险等级 |
|--------|------|----------|
| `filter` | 条件查询（分页） | read |
| `getOne` | 按 ID 获取单条 | read |
| `aggregate` | 聚合统计（分组、求和、计数等） | read |
| `create` | 新建记录 | write |
| `update` | 更新记录 | write |
| `delete` | 删除记录（需 `--yes`） | high-risk-write |

`--params` JSON 就是 API 请求体，无额外封装。

## 各命令详解

### data filter — 条件查询

```bash
# 查全部（默认第一页）
lovrabet data filter --code <code> --params '{"currentPage":1,"pageSize":20}'

# 带条件
lovrabet data filter --code <code> --params '{"where":{"status":{"$eq":"active"}},"currentPage":1,"pageSize":20}'

# 翻页
lovrabet data filter --code <code> --params '{"where":{"status":{"$eq":"active"}},"currentPage":2,"pageSize":50}'

# 完整示例：条件 + 分页 + 排序 + 选字段
lovrabet data filter --code <code> --params '{
  "where": {"$and":[{"status":{"$eq":"active"}},{"amount":{"$gte":100}}]},
  "select": ["id","name","status"],
  "orderBy": [{"id":"desc"}],
  "currentPage": 1,
  "pageSize": 20
}'
```

**--params 结构：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `where` | object | 查询条件 |
| `select` | string[] | 返回字段列表 |
| `orderBy` | object[] | 排序，如 `[{"id":"desc"}]` |
| `currentPage` | number | 页码 |
| `pageSize` | number | 每页条数 |

### data getOne — 单条查询

```bash
lovrabet data getOne --code <code> --params '{"id":123}'
```

### data aggregate — 聚合统计

对数据集进行分组、求和、计数、平均值等聚合操作。

```bash
# 按状态分组求总金额
lovrabet data aggregate --code <code> --params '{
  "aggregate": [{"field":"amount","type":"SUM","alias":"total"}],
  "groupBy": ["status"]
}'

# 按地区统计活跃订单数
lovrabet data aggregate --code <code> --params '{
  "aggregate": [{"field":"id","type":"COUNT","alias":"cnt"}],
  "groupBy": ["region"],
  "where": {"status":{"$eq":"active"}}
}'

# 多维聚合：按状态和地区分组，求总金额并过滤
lovrabet data aggregate --code <code> --params '{
  "aggregate": [{"field":"amount","type":"SUM","alias":"total"}],
  "groupBy": ["status","region"],
  "having": [{"columnName":"total","condition":{"$gte":1000}}],
  "orderBy": [{"total":"desc"}]
}'
```

**--params 结构：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `aggregate` | object[] | **必填**。聚合定义列表 |
| `groupBy` | string[] | 分组字段 |
| `where` | object | 聚合前过滤（与 filter 相同的 where 语法） |
| `having` | object[] | 聚合后过滤，如 `[{"columnName":"total","condition":{"$gte":1000}}]` |
| `select` | string[] | 随聚合结果返回的原始字段 |
| `join` | object[] | 关联配置，如 `[{"type":"LEFT","table":"users","condition":{"user_id":"id"}}]` |
| `orderBy` | object[] | 排序，如 `[{"total":"desc"}]` |
| `currentPage` | number | 页码 |
| `pageSize` | number | 每页条数 |

**聚合类型：**

| type | 说明 | 示例 |
|------|------|------|
| `SUM` | 求和 | `{"field":"amount","type":"SUM","alias":"total"}` |
| `COUNT` | 计数 | `{"field":"id","type":"COUNT","alias":"cnt"}` |
| `COUNT` (distinct) | 去重计数 | `{"field":"id","type":"COUNT","alias":"cnt","distinct":true}` |
| `AVG` | 平均值 | `{"field":"score","type":"AVG","alias":"avg_score"}` |
| `AVG` (rounded) | 带精度平均值 | `{"field":"score","type":"AVG","alias":"avg","round":true,"precision":2}` |

### data create — 新建记录

```bash
# 预览
lovrabet data create --code <code> --params '{"name":"test","amount":100}' --dry-run

# 执行
lovrabet data create --code <code> --params '{"name":"test","amount":100}'
```

### data update — 更新记录

```bash
# 预览
lovrabet data update --code <code> --params '{"id":123,"status":"completed"}' --dry-run

# 执行
lovrabet data update --code <code> --params '{"id":123,"status":"completed"}'
```

### data delete — 删除记录

```bash
# 预览
lovrabet data delete --code <code> --params '{"id":123}' --dry-run

# 执行（需确认）
lovrabet data delete --code <code> --params '{"id":123}' --yes
```

## where 查询语法

| 操作符 | 含义 | 示例 |
|--------|------|------|
| `$eq` | 等于 | `{"status":{"$eq":"active"}}` |
| `$ne` | 不等于 | `{"status":{"$ne":"deleted"}}` |
| `$gte/$gteq/$lte/$lteq` | 比较 | `{"amount":{"$gte":100}}` |
| `$contain` | 包含匹配 | `{"name":{"$contain":"test"}}` |
| `$startWith` | 前缀匹配 | `{"name":{"$startWith":"pre"}}` |
| `$endWith` | 后缀匹配 | `{"name":{"$endWith":"suf"}}` |
| `$notNull` | 非空判断 | `{"app_code":{"$notNull":true}}` |
| `$in` | 包含 | `{"status":{"$in":["active","pending"]}}` |
| `$and` | AND 组合 | `{"$and":[cond1, cond2]}` |
| `$or` | OR 组合 | `{"$or":[cond1, cond2]}` |

## aggregate 参数结构（与 SDK 对齐）

```json
{
  "select": ["category_id"],
  "aggregate": [
    { "type": "SUM", "field": "amount", "alias": "total_amount", "round": true, "precision": 2 }
  ],
  "where": { "status": { "$eq": "active" } },
  "groupBy": ["category_id"],
  "having": [
    { "columnName": "total_amount", "condition": { "$gte": 1000 } }
  ],
  "join": [
    { "type": "LEFT", "table": "users", "condition": { "user_id": "id" } }
  ],
  "orderBy": [{ "total_amount": "desc" }],
  "currentPage": 1,
  "pageSize": 20
}
```

## 错误处理

- 缺少认证 → `Error: No authentication configured`
- `--params` 不是合法 JSON → `Error: Invalid JSON for --params: ...`
- 数据集不存在 → API 返回错误
- 非交互模式下 delete 未加 `--yes` → 高风险保护拦截
