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

1. app 不明确时，先：

```bash
lovrabet app list
```

2. 如果没有 `sqlcode`，从平台或 `rabetbase sql list` 获取
3. 先看详情：

```bash
lovrabet sql detail --sqlcode <code>
```

4. 再执行：

```bash
lovrabet sql exec --sqlcode <code> --params '<json>'
```

## BFF 工作流

1. app 不明确时，先：

```bash
lovrabet app list
```

2. 如果没有脚本信息，从平台或 `rabetbase bff list` 获取
3. 先看详情：

```bash
lovrabet bff detail --id <id>
```

4. 再执行：

```bash
lovrabet bff exec --name <functionName> --params '<json>'
```

## 什么时候可以跳过 app 决议

以下情况可以直接进入 `sql detail/exec` 或 `bff detail/exec`：

- 用户已经给了 `--appcode`
- 用户已经给了 `--app <name>`
- 当前配置已有明确 `defaultApp`
- 当前对话已经明确在某个 app 上下文中继续

## 不要这样做

- 不要在不知道 `sqlcode` 的情况下直接跑 `sql exec`
- 不要在不知道函数名的情况下直接跑 `bff exec`
- 不要在 app 未明确时直接默认当前 app 一定对

## 推荐配合命令

- app 决议：`lovrabet app list`
- SQL 列表来源：`rabetbase sql list` 或平台
- BFF 列表来源：`rabetbase bff list` 或平台
