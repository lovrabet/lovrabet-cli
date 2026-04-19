# Dataset And Data Workflow Guide

## 目的

把“业务需求 -> 应用决议 -> 数据集定位 -> 数据操作”收成一条稳定流程，避免直接盲猜 `dataset code` 或 app。

## 推荐流程

1. 先判断 app 是否明确  
   不明确时，先看 [app-resolution.md](app-resolution.md)

2. 定位数据集

```bash
lovrabet dataset list --name "<关键词>"
```

如需限定候选 app：

```bash
lovrabet dataset list --app <name> --name "<关键词>"
```

3. 看数据集结构

```bash
lovrabet dataset detail --code <datasetCode>
```

4. 拿到 `datasetCode` 后，再进入 `data` 子命令

```bash
lovrabet data filter --code <datasetCode> --params '<json>'
lovrabet data getOne --code <datasetCode> --params '<json>'
lovrabet data create --code <datasetCode> --params '<json>'
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
- 要给 `data create/update` 组参数

## `data` 的使用原则

`data` 命令只依赖 `datasetCode`，不直接做 app 决议。  
所以不要在 app 未确定、dataset 未确认时直接构造 `data filter/getOne/create/update/delete`。

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
