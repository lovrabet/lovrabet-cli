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
- `data.meta.defaultApp` / `defaultAppSource`：当前默认候选应用决议结果

## 何时应该先跑 app list

应该先跑 `app list` 的场景：

- 用户只说了业务域，没有给 app 线索
- 当前没有显式 `--appcode` / `--app`，且需求包含业务域、对象名或数据集线索
- 你怀疑需求可能落在多个 app 中
- 你需要先确认当前 AK 在该环境下能看到哪些应用
- 已经在 `defaultApp` 下按关键词验证过，但没有合理数据集命中

不需要先跑 `app list` 的场景：

- 用户已经给了 `--appcode`
- 用户已经给了 `--app <name>`
- 问题明显沿用上文已确认的同一 app 上下文
- 用户明确说“当前应用”“默认应用”，且没有新的业务域线索

`defaultApp` 只是默认候选，不是强上下文。未显式指定 app 且存在 `defaultApp` 时，应先用 `dataset list --name <关键词>` 验证默认候选；命中不合理时，再通过 `app list` 扩大搜索。

## 如何选择使用哪个应用

推荐方法不是只看 app 名，而是：

1. 有 `defaultApp` 时，先在默认候选下执行 `dataset list --name <关键词>`
2. 默认候选命中合理时直接继续
3. 默认候选无命中、弱命中或语义不合理时，再用 `app list` 拿到候选 app
4. 按需求关键词挑 1-2 个候选，对每个候选执行 `dataset list --app <name> --name <关键词>`
5. 用数据集命中情况反向确认正确 app

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

## app use — 设置默认候选应用

持久写入 `.lovrabet.json` 的 `defaultApp` 字段。它表示没有更明确 app 线索时的默认候选，不应压过用户本次需求里的业务域线索。

```bash
lovrabet app use <name>
lovrabet app use <name> --env daily
lovrabet app use <name> --global
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `<name>` | string | 必填。应用名 |

| Flag | 类型 | 默认 | 说明 |
|------|------|------|------|
| `--env <env>` | string | 当前环境 | 指定应用名校验时使用的缓存环境：production / development / daily |
| `--global` | boolean | false | 操作全局配置 `~/.lovrabet.json` |

**行为**：
- 应用名来自当前环境 cache 中的远端 app 名称；如果前一步用 `app list --env daily` 查到应用，后续 `app use` 也要带 `--env daily`
- `app use` 只修改 `defaultApp`
- 不会同步或写入整份远端应用目录
- 如果提示 `App "<name>" not found.`，先确认 `--env` 是否和前一步 `app list` / `app pull` 使用的环境一致，再考虑 `app list --no-cache --env <env>` 刷新缓存

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
