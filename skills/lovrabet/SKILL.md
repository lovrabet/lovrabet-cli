---
name: lovrabet
version: 2.1.1
description: "Lovrabet 运行态 CLI — 通过 lovrabet 命令管理应用目录、数据集查询、Instant API 数据操作、SQL 执行、BFF 调用。触发词：云图、lovrabet、lovrabet-cli、app list、dataset、data filter、data getOne、create、update、delete、sql exec、bff exec、accessKey、compress、jq。"
metadata:
  requires:
    bins: ["lovrabet"]
    node: ">=20"
  cliHelp: "lovrabet --help"
---

# lovrabet-runtime-cli — Lovrabet 运行态 CLI

面向业务人员和集成开发者的运行态 CLI，提供数据集查询、Instant API 数据操作、SQL 执行、BFF 调用能力。

> **结构化输出与 jq**：需要机器可读 JSON 时优先 **`--format compress`**（单行信封）；需缩进时用 **`--format json`**。二者均可叠加全局 **`--jq '<expr>'`** 缩小输出；语义与 `rabetbase` 一致，详见 [输出格式与 --jq](references/lovrabet-output-format-jq.md)。

## 使用顺序

先看 guides，再查 references。

- 流程 / 决策：
  - [应用决议](guides/app-resolution.md)
  - [数据集与数据流程](guides/dataset-and-data-workflow.md)
  - [SQL 与 BFF 流程](guides/sql-and-bff-workflow.md)
  - [排障](guides/troubleshooting.md)
- 命令细节 / 参数参考：
  - `references/*.md`

## 安装

```bash
npm install -g @lovrabet/lovrabet-cli
```

## 认证

当前主路径是 **User Access Key（client-ak）**。优先级：**CLI flags > 环境变量 > 配置文件**。

1. **User AK**：`.lovrabet.json` 或 `LOVRABET_ACCESS_KEY` 配 **`accessKey`**（适合 CI）；有 `accessKey` 时走 **client-ak**
2. **Cookie**：历史兼容读取，已不是推荐主路径；新文档与 Agent 指导默认按 AK 流程处理

详见 [认证参考](references/lovrabet-auth.md)。

### 认证命令选择

- **只想更新 AK，尽量保留现有配置**：使用 `lovrabet auth login`
- **还没有 AK，需要先自助创建**：访问 `https://user.lovrabet.com/user/ak` 创建，再执行 `lovrabet auth login`
- **想确认当前 AK 对应的是哪个用户**：使用 `lovrabet auth info`
- **凡是后续命令需要“当前登录用户身份信息”**：统一先执行 `lovrabet auth info` 获取，不要猜当前 AK 对应的人
- **要从头重建当前作用域认证配置**：使用 `lovrabet auth init`
- **不要**把 `auth init` 当作普通登录命令；它会清空当前作用域下已有配置，再只写回新的认证结果
- `auth login --env` 仅作为实现层兼容能力存在，默认不要在指导中主推；需要“清空后重建 + 写 env”时优先用 `auth init --env ...`

## 配置作用域原则（`--global`）

- **写操作默认当前项目**（`app init`、`app import`、`config set`、`config delete` 等），除非用户**显式**传 `--global`。
- **不要**在用户未要求时给命令加 `--global`** — 默认行为已是「项目优先」；只有用户明确要改全局配置或不在项目内且意图写全局时才使用。
- **`config set` / `config delete`**：在**没有**项目配置文件（当前目录未解析到 `.lovrabet.json`）且**未**传 `--global` 时，CLI **拒绝执行**并提示使用 `--global` 或先 `lovrabet app init`，**不会**静默写入全局。
- **`app list`**：默认走**远端优先 + 本地缓存**；`--local` 只读缓存；`--no-cache` 强制打线上并刷新缓存。
- **`app pull`**：只刷新本地 app cache，**不**把远端应用列表写入 `.lovrabet.json`；它是手动刷新命令，平时优先使用 `app list`
- **`app use` / `app import`**：只操作本地用户意图配置（`defaultApp` / 顶层 `appcode`），详见 [应用管理](references/lovrabet-app.md)。

## Agent 禁止行为

- **不要擅自加 `--global`** — 见上文「配置作用域原则」；默认写项目、读合并；仅在用户明确要求或文档说明的场景使用 `--global`。
- **禁止通过修改配置文件提升权限** — 不得为了完成任务而修改 `.lovrabet.json`、环境变量或缓存内容来抬高 `riskLevel`、切换到并非用户明确授权的 `accessKey`、伪造 `defaultApp` / `appcode` / `env`、或借此突破当前权限边界。权限不足时，应明确说明限制，并要求用户提供合法的目标应用、凭证或确认范围。
- **不要臆测当前登录用户** — 只要任务依赖“当前是谁在登录 / 当前 AK 属于谁”，先执行 `lovrabet auth info`，再继续判断应用、权限或数据可见性。
- **SQL/BFF 标识发现与执行分层** — `lovrabet` 负责运行态 `sql exec` / `bff exec`；如果缺少 `sqlcode`、脚本 ID 或函数名，可以从平台 UI、用户提供信息、前序上下文，或显式交接到研发态 `rabetbase sql list` / `rabetbase bff list` 获取。不要把运行态执行和研发态发现混成一个隐式步骤。

## Agent 决策：何时获取应用信息

在执行 `dataset` / `data` / `sql` / `bff` 前，不要一律先跑 `app list`。先判断当前需求是否真的需要做“应用决议”。

### 可以直接使用当前应用的场景

满足以下任一条件时，优先直接用当前应用执行，不先打 `app list`：

1. 用户已经显式给了 `--appcode`
2. 用户已经显式给了 `--app <name>`
3. 问题明显是在上文已经确认过的同一 app 上继续操作
4. 用户明确说“当前应用”“默认应用”，且没有新的业务域线索

这种情况下，直接进入：

1. `dataset list --name ...`
2. `dataset detail --code ...`
3. `data filter/getOne/create/update/delete`

不要先多打一轮应用发现命令制造噪音。

`defaultApp` 只是弱候选，不是强上下文。未显式指定 app 时，如果本地已有 `defaultApp`，应先在默认候选里按需求关键词做数据集验证；无命中、弱命中或语义不合理时，再扩大到 `app list` 里查找其他应用。

### 需要进一步应用决议的场景

出现以下情况时，应先做应用决议；有 `defaultApp` 时先验证默认候选，必要时再跑 `lovrabet app list`：

1. 用户只给了业务需求，没有给 app 线索  
   例如：“帮我查订单数据集”“看一下 CRM 里的客户表”
2. 当前没有显式 `--appcode` / `--app`，且需求中有业务域、对象名或数据集线索
3. 当前配置中存在多个候选 app，且需求描述不足以直接锁定一个
4. 用户明确说“先看看我有哪些应用”或“这个需求应该在哪个应用里”
5. 已经在 `defaultApp` 下按关键词验证过，但没有合理数据集命中

### `app list` 的使用方式

- 默认：`lovrabet app list`
  - 远端优先，必要时自动刷新 cache
- 只想复用本地目录时：`lovrabet app list --local`
- 明确要强制刷新时：`lovrabet app list --no-cache`

### 如何根据需求判断使用哪个应用

优先级按下面做，不要随意跳步：

1. **显式指定优先**
   - 用户给了 app 名、appcode、当前项目语境，直接采用
2. **上下文确认优先**
   - 上文刚确认过目标 app，且本次没有引入新的业务域，继续使用该 app
3. **默认候选先验证**
   - 有 `defaultApp` 时，先执行 `dataset list --name <关键词>` 验证默认候选是否匹配
   - 命中的数据集名称、字段、描述明显贴合需求时，继续用默认候选
   - 无命中、弱命中或语义不合理时，再进入 `app list`
4. **需求关键词映射**
   - 需求里出现业务域关键词时，用 `app list` 输出的 `name` 先做语义匹配  
   - 例如：`CRM`、`订单`、`电商`、`证照`、`需求管理`
5. **验证式收敛**
   - 对候选应用执行 `dataset list --app <name> --name <关键词>`
   - 哪个返回的数据集更匹配，就收敛到哪个 app

### 推荐决策流程

当用户说“帮我查某个业务数据”时，推荐流程是：

1. 先判断当前是否已有明确 app 上下文
2. 若有 `defaultApp`，先 `lovrabet dataset list --name <关键词>` 验证默认候选
3. 默认候选命中合理时，继续 `dataset detail` 或 `data filter`
4. 默认候选无命中或不合理时，再 `lovrabet app list`
5. 按业务关键词挑 1-2 个候选 app，并执行 `dataset list --app <name> --name <关键词>` 验证

### 不要这样做

- 不要每次都先 `app list`
- 不要只看 app 名就武断决定，不做 `dataset list` 验证
- 不要把平台应用目录再写回 `.lovrabet.json`
- 不要因为本地配置里没有某个 app 就判定它不存在，先看 cache/remote 目录

| 服务 | 命令 | 说明 | 风险等级 | 需 Cookie |
|------|------|------|----------|-----------|
| **auth** | `login` | 保存 accessKey | — | — |
| **auth** | `init` | 清空当前作用域配置并重建认证 | write | — |
| **auth** | `logout` | 清除本地 accessKey | — | — |
| **auth** | `info` | 查询当前 AK 对应的登录用户信息 | read | — |
| **app** | `init` | 创建 `.lovrabet.json` 配置 | write | — |
| **app** | `list` | 列出当前 AK 可见应用（远端优先，带缓存） | read | — |
| **app** | `pull` | 刷新本地 app cache | write | — |
| **app** | `use` | 设置默认候选应用 | write | — |
| **app** | `import` | 从升级后的 .rabetbase.json 导入顶层配置 | write | — |
| **config** | `list` | 查看完整配置 | read | — |
| **config** | `get` | 读取配置项 | read | — |
| **config** | `set` | 写入配置项 | write | — |
| **config** | `delete` | 删除配置项 | write | — |
| **skill** | `install` | 全局安装官方运行态 Skill | write | — |
| **update** | `run` | 从 npm 更新 CLI 并刷新官方 Skill | write | — |
| **dataset** | `list` | 列出数据集（含字段列表） | read | **是** |
| **dataset** | `detail` | 查看数据集结构 | read | **是** |
| **data** | `filter` | 按条件查询数据记录 | read | — |
| **data** | `getOne` | 按 ID 获取单条记录 | read | — |
| **data** | `aggregate` | 聚合统计 | read | — |
| **data** | `create` | 新建记录 | write | — |
| **data** | `update` | 更新记录 | write | — |
| **data** | `delete` | 删除记录（需 `--yes`） | high-risk-write | — |
| **sql** | `detail` | 查看 SQL 查询详情 | read | **是** |
| **sql** | `exec` | 执行 SQL 查询 | read | — |
| **bff** | `detail` | 查看 BFF 脚本详情 | read | **是** |
| **bff** | `exec` | 执行 BFF 函数 | read | — |
| **logs** | `show` | 查看命令执行历史 | read | — |
| **logs** | `clear` | 清除命令历史 | read | — |

## 意图 → 命令索引

### 初始化与配置

| 意图 | 命令 |
|------|------|
| 安装 / 刷新 Skill | `lovrabet skill install` |
| 初始化配置 | `lovrabet app init --appcode <code> [--env daily] [--global]` |
| 从升级后的 rabetbase 配置导入 | `lovrabet app import --file /path/to/.rabetbase.json` |
| 登录 | `lovrabet auth login` |
| 查看当前 AK 对应的登录用户 | `lovrabet auth info` |
| 任何依赖“当前登录用户身份”的场景先取身份 | `lovrabet auth info` |
| 重置并重建认证配置 | `lovrabet auth init --access-key ak_xxx [--env daily]` |
| 登出 | `lovrabet auth logout` |
| 查看当前认证状态 | `lovrabet auth status` |
| 查看配置 | `lovrabet config list` |
| 读取配置项 | `lovrabet config get <key>` |
| 设置配置项 | `lovrabet config set <key> <value> [--global]` |
| 删除配置项 | `lovrabet config delete <key> [--global]` |
| 查看应用列表 | `lovrabet app list` |
| 只看本地缓存的应用列表 | `lovrabet app list --local` |
| 强制刷新线上应用列表 | `lovrabet app list --no-cache` |
| 手动刷新应用缓存 | `lovrabet app pull` |
| 设置默认候选应用 | `lovrabet app use <name>` |
| 更新 CLI 到 npm latest | `lovrabet update --latest` |
| 更新 CLI 到 npm beta | `lovrabet update --beta` |
| 更新 CLI 到指定版本 | `lovrabet update --version <version>` |

### CLI 更新

`lovrabet update` 只查询 npm package 的 dist-tags，不依赖 CDN 文件。默认等价于 `--latest`；`--beta` 使用 npm `beta` dist-tag；`--version <version>` 安装指定 semver。更新检查完成后会自动刷新官方 Skill，除非显式传 `--no-skills`。

开源二开配置集中在项目代码的 `src/constant/product.ts`，`src/constant/distribution.ts` 只做兼容导出：

- `PRODUCT_CONFIG.cliBinName`：CLI 可执行命令名
- `PRODUCT_CONFIG.npmPackageName`：npm 包名
- `PRODUCT_CONFIG.skillSource`：`npx skills add` 的 skill source
- `PRODUCT_CONFIG.envPrefix`：环境变量前缀
- `PRODUCT_CONFIG.domains`：默认服务域名
- `PRODUCT_CONFIG.userCenterDisplayName` / `accessKeyCreatePath`：AK 创建提示

### 数据集与 Instant API

| 意图 | 命令 |
|------|------|
| 找数据集 | `lovrabet dataset list [--name 关键词] [--code 精确码]` |
| 看数据集字段 | `lovrabet dataset detail --code <code> [--verbose]` |
| 查询数据 | `lovrabet data filter --code <code> --params '{"where":{"status":{"$eq":"active"}},"currentPage":1,"pageSize":20}'` |
| 获取单条 | `lovrabet data getOne --code <code> --params '{"id":123}'` |
| 新建记录 | `lovrabet data create --code <code> --params '{"name":"test","amount":100}'` |
| 更新记录 | `lovrabet data update --code <code> --params '{"id":123,"status":"done"}'` |
| 删除记录 | `lovrabet data delete --code <code> --params '{"id":123}' --yes` |
| 聚合统计 | `lovrabet data aggregate --code <code> --params '{"aggregate":[{"field":"amount","type":"SUM","alias":"total"}],"groupBy":["status"]}'` |

### SQL 与 BFF

| 意图 | 命令 |
|------|------|
| 查看 SQL | `lovrabet sql detail --sqlcode <code>` |
| 执行 SQL | `lovrabet sql exec --sqlcode <code> --params '{"key":"value"}'` |
| 查看 BFF | `lovrabet bff detail --id <id>` |
| 执行 BFF | `lovrabet bff exec --name <functionName> --params '{"key":"value"}'` |

### 日志

| 意图 | 命令 |
|------|------|
| 查看日志 | `lovrabet logs` |
| 清除日志 | `lovrabet logs clear` |

## 统一的 --params 设计

`data`、`sql exec`、`bff exec` 均通过 `--params` 传入 JSON 请求体，直接透传给运行态 API：

```bash
lovrabet <service> <command> --code <code> --params '<json>'
```

## Instant API 工作流

```bash
# 1. 找到目标数据集
lovrabet dataset list --name "订单"

# 2. 查看字段结构（确认数据集 code + 字段名）
lovrabet dataset detail --code <datasetCode>

# 3. 查询数据
lovrabet data filter --code <datasetCode> --params '{"where":{"status":{"$eq":"active"}},"currentPage":1,"pageSize":20}'

# 翻到第 2 页
lovrabet data filter --code <datasetCode> --params '{"where":{"status":{"$eq":"active"}},"currentPage":2,"pageSize":20}'

# 4. 获取单条
lovrabet data getOne --code <datasetDataset> --params '{"id":123}'

# 5. 创建记录（先 dry-run 预览）
lovrabet data create --code <datasetCode> --params '{"name":"test","amount":100}' --dry-run
lovrabet data create --code <datasetCode> --params '{"name":"test","amount":100}'

# 6. 更新记录
lovrabet data update --code <datasetCode> --params '{"id":123,"status":"completed"}'

# 7. 删除记录（高风险，需确认）
lovrabet data delete --code <datasetCode> --params '{"id":123}' --dry-run
lovrabet data delete --code <datasetCode> --params '{"id":123}' --yes
```

### filter 的 --params 结构

`--params` JSON 直接作为 `filter` 请求体透传：

```json
{
  "where": { "status": { "$eq": "active" } },
  "select": ["id", "name", "status"],
  "orderBy": [{ "id": "desc" }],
  "currentPage": 1,
  "pageSize": 20
}
```

### where 查询语法

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
| `$and/$or` | 组合条件 | `{"$and":[...]}` |

### aggregate 的 --params 结构

`--params` JSON 直接作为 `aggregate` 请求体透传，字段与 SDK `AggregateParams` 对齐：

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

## 全局选项

| 选项 | 说明 |
|------|------|
| `--appcode <code>` | 覆盖 appcode |
| `--env <env>` | 环境: production / development / daily |
| `--format <fmt>` | 输出: **json** / **pretty** / **compress**（默认偏好见配置与 `--help`） |
| `--jq '<expr>'` | 对 **json / compress** 最终打印的 JSON 做 jq 过滤；与 `rabetbase` 行为一致，详见 [lovrabet-output-format-jq.md](references/lovrabet-output-format-jq.md) |
| `--app <name>` | 多应用模式下指定应用 |
| `--dry-run` | 预览不执行（write 命令） |
| `--yes` | 跳过确认（high-risk-write） |
| `--non-interactive` | 非交互模式（CI） |

## 配置文件

项目根目录 `.lovrabet.json`（完整字段、优先级、环境变量见 [配置参考](references/lovrabet-config.md)）：

```json
{
  "appcode": "app-xxxxxxxx",
  "env": "daily",
  "accessKey": "ak_xxx"
}
```

配置文件只保存用户意图配置，不保存平台应用目录。远端应用列表缓存位于：

```text
~/.lovrabet/cache/<env>/<ak-fingerprint>/my-apps.json
```

多应用意图配置示例：

```json
{
  "accessKey": "ak_xxx",
  "env": "daily",
  "defaultApp": "crm"
}
```

## 风险控制

| 等级 | 命令 | 保护 |
|------|------|------|
| read | dataset list/detail, data filter/getOne/aggregate, sql detail/exec, bff detail/exec, app list, config list/get, logs | 无 |
| write | data create, data update, app use, config set/delete, app import | dry-run 预览 |
| high-risk-write | data delete | 需 `--yes` 或交互确认 |

## 详细参考

| 主题 | 文件 |
|------|------|
| 输出格式与 `--jq` | [lovrabet-output-format-jq.md](references/lovrabet-output-format-jq.md) |
| 认证 | [lovrabet-auth.md](references/lovrabet-auth.md) |
| 初始化 | [lovrabet-init.md](references/lovrabet-init.md) |
| 应用管理 | [lovrabet-app.md](references/lovrabet-app.md) |
| 配置管理 | [lovrabet-config-commands.md](references/lovrabet-config-commands.md) |
| 配置文件参考 | [lovrabet-config.md](references/lovrabet-config.md) |
| 数据集发现 | [dataset-discovery-workflow.md](references/dataset-discovery-workflow.md) |
| Instant API | [instant-api-workflow.md](references/instant-api-workflow.md) |
| SQL 工作流 | [lovrabet-sql-workflow.md](references/lovrabet-sql-workflow.md) |
| BFF 工作流 | [lovrabet-bff-workflow.md](references/lovrabet-bff-workflow.md) |
| 日志 | [lovrabet-logs.md](references/lovrabet-logs.md) |
