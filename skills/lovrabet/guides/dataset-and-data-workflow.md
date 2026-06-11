# Dataset And Data Workflow Guide

## 目的

把“业务需求 -> 应用决议 -> 数据集定位 -> Instant API 数据操作”收成一条稳定流程，避免直接盲猜 `dataset code` 或 app。

## 推荐流程

1. 先判断 app 是否明确  
   不明确时，先看 [app-resolution.md](app-resolution.md)

2. 需要写 Artifact、personal BFF 或知识库内容时，先做发现

```bash
lovrabet api-doc list --category dataset
lovrabet api-doc detail --code dataset_data_filter
```

如果用户已经给了明确字段和值，可以直接使用用户提供的数据；否则先确认真实 API 和数据结构。

3. 定位数据集

```bash
lovrabet dataset list --name "<关键词>"
```

如需限定候选 app：

```bash
lovrabet dataset list --app <name> --name "<关键词>"
```

4. 看数据集结构

```bash
lovrabet dataset detail --code <datasetCode>
```

5. 需要 SDK 调用示例时读取数据集 SDK 文档

```bash
lovrabet dataset sdk-doc --code <datasetCode>
```

6. 拿到 `datasetCode` 后，再进入 `data` 子命令

```bash
lovrabet data filter --code <datasetCode> --params '<json>'
lovrabet data getOne --code <datasetCode> --params '<json>'
lovrabet data create --code <datasetCode> --params '<json>'
lovrabet data batchCreate --code <datasetCode> --params '[{"name":"a"},{"name":"b"}]'
lovrabet data update --code <datasetCode> --params '<json>'
lovrabet data delete --code <datasetCode> --params '<json>' --yes
```

## 什么时候只用 `dataset list`

以下情况只用 `dataset list` 就够：

- 用户只是问“有哪些数据集”
- 用户只是想按名称搜索数据集
- 只需要数据集 code 和字段名数组

## 什么时候必须看 `dataset detail`

以下情况必须补一轮 `dataset detail`：

- 要确认字段类型
- 要确认字段是否可写
- 要确认主键 / 操作列表
- 要给 `data create/batchCreate/update` 组参数

## 什么时候看 `dataset sdk-doc`

以下场景补一轮 `dataset sdk-doc`：

- 需要在 personal BFF 或 Artifact 中调用运行态 SDK
- 需要确认 SDK 参数结构，而不只是字段名
- 要把数据访问方式写进可交付源码或文档

## `data` 的使用原则

`data` 命令只依赖 `datasetCode`，不直接做 app 决议。  
所以不要在 app 未确定、dataset 未确认时直接构造 `data filter/getOne/create/batchCreate/update/delete`。

## 批量新增的边界

`lovrabet data batchCreate` 只适合“同一数据集多条新增”，用于减少请求次数。参数必须是 JSON 数组，或 `{"items":[...]}`：

```bash
lovrabet data batchCreate --code <datasetCode> --params '[{"name":"a"},{"name":"b"}]' --dry-run
lovrabet data batchCreate --code <datasetCode> --params '[{"name":"a"},{"name":"b"}]'
```

使用前仍要先做 `dataset detail`，确认字段可写、必填字段和业务唯一键。执行后按业务唯一键只读核对，避免“请求结果未知”时重复写入。

不要在执行时直接用 `data batchCreate` 绕过 BFF 业务编排。只要写入涉及多个数据集、upsert、依赖顺序、幂等、失败恢复或 handoff 结果，就应调用业务级 `bff exec`，由 BFF/CLI service 内部决定是否使用批量新增。BFF 写入类执行仍需先确认业务授权、Studio 权限和人工确认语义；CLI 将 `bff exec` 标记为 `read`，不等同于免审批写入。

## 高风险动作

删除前先预览：

```bash
lovrabet data delete --code <datasetCode> --params '<json>' --dry-run
lovrabet data delete --code <datasetCode> --params '<json>' --yes
```

## 常见错误

### 只有业务描述，没有 dataset code

不要直接问运行态接口。先：

1. 决议 app
2. `dataset list --name`
3. `dataset detail`

### app 不明确但直接跑 `data *`

这通常是在跳过应用决议。先退回去做 app + dataset 定位。
