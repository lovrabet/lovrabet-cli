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
