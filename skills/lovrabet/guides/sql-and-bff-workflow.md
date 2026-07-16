# SQL And BFF Workflow Guide

## 目的

统一 `sql` / `bff` 的使用方式，避免在 app 未明确、脚本/SQL 标识未确认时直接执行。

## 共同原则

`sql` 和 `bff` 都不是“先猜再试”的命令。

标准顺序：

1. 先判断 app 是否明确
2. 不明确时，先做应用决议
3. 再确认 `sqlcode` 或 `script id / function name`
4. 最后执行 `detail` / `exec`

## SQL 工作流

1. app 不明确时，按 [app-resolution.md](app-resolution.md) 决议：有 `defaultApp` 先验证默认候选，验证不成立再扩大到应用列表：

```bash
# 默认候选验证不成立时
lovrabet app list
```

2. 如果没有 `sqlcode`，从用户提供信息、平台 UI、前序上下文获取；需要研发态发现时，显式交接到 `rabetbase sql list`
3. 先看详情：

```bash
lovrabet sql detail --sqlcode <code>
```

4. 再执行：

```bash
lovrabet sql exec --sqlcode <code> --params '<json>'
```

## BFF 工作流

1. app 不明确时，按 [app-resolution.md](app-resolution.md) 决议：有 `defaultApp` 先验证默认候选，验证不成立再扩大到应用列表：

```bash
# 默认候选验证不成立时
lovrabet app list
```

2. 如果没有脚本信息，从用户提供信息、平台 UI、前序上下文获取；需要研发态发现时，显式交接到 `rabetbase bff list`
3. 先看详情：

```bash
lovrabet bff detail --name <functionName>
```

4. 再执行：

```bash
lovrabet bff exec --name <functionName> --params '<json>'
```

## app-config value 查询

`lovrabet app-config get <key>` 按当前 appCode 和 key 查询运行态 app-config value：

```bash
lovrabet app-config get example_api_key --format compress
```

命令的 `data` 直接返回 value；该值可能包含敏感信息，不要写入本地配置、缓存、日志、`--params` 或业务 Skill 入参。

示例 key 仅用于说明；实际执行必须使用用户或业务 Skill 明确给出的 key，不能猜测或默认使用示例 key。Agent 执行 Skill 时同样调用该 CLI 获取 value，不创建取配置 BFF；value 只在当前任务内消费，除非用户明确要求，否则最终答复不重复展示。

## personal BFF 工作流

personal BFF 是当前用户在当前应用下维护的个人脚本，适合给 Artifact 做轻量数据编排，或先验证一个临时业务接口的返回形状。

标准顺序：

1. `lovrabet personal-bff list` 查当前用户已有脚本
2. `lovrabet personal-bff detail --id <id>` 查看现有脚本后再更新
3. 从本地脚本文件 `create` 或 `update`
4. `lovrabet personal-bff exec --id <id> --params '<json>' --yes --format compress` 确认返回形状
5. 再把结果形状用于 Artifact 源码或交付说明

```bash
lovrabet personal-bff create --name loadOrders --file ./load-orders.js --dry-run
lovrabet personal-bff exec --id <id> --params '{"status":"active"}' --yes --format compress
```

`personal-bff exec` 是 `high-risk-write`，因为脚本行为可能有副作用；非交互场景必须带 `--yes`。

## 复杂业务写入与频率保护

多数据集写入、跨步骤依赖、upsert、执行时需要 handoff 结果或需要幂等恢复的业务动作，优先封装在 BFF 中，再通过 `lovrabet bff exec` 调用。Agent 不应在执行时直接拼多次 `data create` 或 `data batchCreate` 绕过业务入口。BFF 写入类执行仍需先确认业务授权、Studio 权限和人工确认语义；CLI 将 `bff exec` 标记为 `read`，不等同于免审批写入。

推荐 BFF 入参包含业务级 `requestId` 或 `idempotencyKey`。BFF 内部按业务唯一键先只读核对，确认不存在再写入；遇到频率保护、超时或客户端没有拿到成功结果时，等待后再次只读核对，确认没有落地再重试。只有同一数据集多条新增时，BFF/CLI service 内部才适合使用 `batchCreate` 减少请求次数。

BFF 返回值应包含 handoff 所需结果：已创建记录、已复用记录、失败步骤、是否可重试和 traceId。这样 Agent 拿到的是业务结果，而不是一串低层写接口的临场拼装结果。

## 什么时候可以跳过 app 决议

以下情况可以直接进入 `sql detail/exec` 或 `bff detail/exec`：

- 用户已经给了 `--appcode`
- 用户已经给了 `--app <name>`
- 当前对话已经明确在某个 app 上下文中继续
- 用户明确说“当前应用”“默认应用”，且没有新的业务域线索

`defaultApp` 只是默认候选，不是强上下文。SQL / BFF 经常按业务域分布在不同 app 中；用户提到新的业务域或业务对象且未指定 app 时，先把 `defaultApp` 当第一个候选验证，验证不成立再扩大到应用列表。

## 不要这样做

- 不要在不知道 `sqlcode` 的情况下直接跑 `sql exec`
- 不要在不知道函数名的情况下直接跑 `bff exec`
- 不要在 app 未明确时直接默认当前 app 一定对

## 推荐配合命令

- app 决议：`lovrabet app list`
- SQL 标识来源：用户提供、平台 UI、前序上下文，或显式研发态发现 `rabetbase sql list`
- BFF 标识来源：用户提供、平台 UI、前序上下文，或显式研发态发现 `rabetbase bff list`
- personal BFF：`lovrabet personal-bff list/detail/create/update/exec`
