# app — 应用管理

`app` 子命令现在分成两类职责：

- **应用目录发现**：通过远端接口和本地 cache 获取当前 AK 可见的应用列表
- **本地用户意图**：通过 `.lovrabet.json` 保存 `defaultApp` 和顶层 `appcode`

> **核心边界**：平台应用列表不再写入 `.lovrabet.json`。`.lovrabet.json` 只保存用户配置；应用目录缓存位于 `~/.lovrabet/cache/...`。

## app list — 列出当前 AK 可见应用

```bash
lovrabet app list
lovrabet app list --local
lovrabet app list --no-cache
lovrabet app list --env daily
```

| Flag | 说明 |
|------|------|
| `--env <env>` | 指定环境，默认当前环境 |
| `--local` | 只读本地 cache，不打远端 |
| `--no-cache` | 强制打远端并刷新 cache |

**行为**：
- 默认：远端优先，成功后刷新 cache
- 远端失败且本地有 cache：回退到 cache
- `--local`：只读 cache
- 输出中的 `meta.source` 为 `remote` / `cache` / `mock`

**缓存路径**：

```text
~/.lovrabet/cache/<env>/<ak-fingerprint>/my-apps.json
```

**输出**：
- `data.items`：应用列表
- `data.meta.env`：环境
- `data.meta.source`：来源
- `data.meta.fetchedAt`：最近同步时间
- `data.meta.cachePath`：cache 文件路径
- `data.meta.defaultApp` / `defaultAppSource`：当前默认应用决议结果

## 何时应该先跑 app list

应该先跑 `app list` 的场景：

- 用户只说了业务域，没有给 app 线索
- 当前没有 `defaultApp`
- 你怀疑需求可能落在多个 app 中
- 你需要先确认当前 AK 在该环境下能看到哪些应用

不需要先跑 `app list` 的场景：

- 用户已经给了 `--appcode`
- 用户已经给了 `--app <name>`
- 当前默认应用已经明确，且问题明显沿用当前上下文

## 如何选择使用哪个应用

推荐方法不是只看 app 名，而是：

1. 先用 `app list` 拿到候选 app
2. 按需求关键词挑 1-2 个候选
3. 对每个候选执行 `dataset list --app <name> --name <关键词>`
4. 用数据集命中情况反向确认正确 app

这样比纯靠 app 名猜测更稳。

## app pull — 手动刷新本地 app cache

```bash
lovrabet app pull
lovrabet app pull --env daily
lovrabet app pull --local
lovrabet app pull --no-cache
```

| Flag | 说明 |
|------|------|
| `--env <env>` | 指定环境，默认当前环境 |
| `--local` | 只读 cache，不请求远端 |
| `--no-cache` | 强制打远端并刷新 cache |

**注意**：
- 平时优先用 `lovrabet app list`；只有你明确想单独刷新缓存时才用 `app pull`
- `app pull` **不再**把远端应用列表写入 `.lovrabet.json`
- 它的职责只是“刷新本地 cache”

## app use — 切换默认应用

持久写入 `.lovrabet.json` 的 `defaultApp` 字段。

```bash
lovrabet app use <name>
lovrabet app use <name> --global
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `<name>` | string | 必填。应用名 |

| Flag | 类型 | 默认 | 说明 |
|------|------|------|------|
| `--global` | boolean | false | 操作全局配置 `~/.lovrabet.json` |

**行为**：
- 应用名来自当前 cache 中的远端 app 名称
- `app use` 只修改 `defaultApp`
- 不会同步或写入整份远端应用目录

## app import — 从 rabetbase 配置导入

从升级后的 `.rabetbase.json` 导入运行态顶层配置。

```bash
lovrabet app import --file /path/to/.rabetbase.json
```

如果源文件还是 legacy / multi-app 结构，先执行：

```bash
rabetbase project upgrade
```

## 参考

- [SKILL.md](../SKILL.md)
- [lovrabet-config.md](lovrabet-config.md)
