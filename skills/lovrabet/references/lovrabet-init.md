# app init — 初始化配置

创建 `.lovrabet.json` 用户意图配置文件。

> **职责边界**：`app init` 只初始化本地配置，不拉取平台应用目录，也不生成 app cache。应用目录缓存位于 `~/.lovrabet/cache/...`，通常由 `lovrabet app list` 自动刷新；必要时也可手动执行 `lovrabet app pull`。

## app init — 创建配置文件

```bash
# 交互式创建
lovrabet app init

# 直接指定 appcode
lovrabet app init --appcode app-xxxxxxxx --env daily

# 写入全局配置（可在任意目录使用）
lovrabet app init --appcode app-xxxxxxxx --global

# 从升级后的 rabetbase 配置导入
lovrabet app import --file /path/to/.rabetbase.json
```

| Flag | 类型 | 必填 | 默认 | 说明 |
|------|------|------|------|------|
| `--appcode` | string | 否 | — | 应用代码。不传则进入交互模式 |
| `--env` | string | 否 | `production` | 环境：`production` / `development` / `daily` |
| `--global` | boolean | 否 | false | 写入 `~/.lovrabet.json` |
| `--yes` | boolean | 否 | false | 跳过确认 |

## 行为

1. 已存在 `.lovrabet.json` 时报错，不覆盖
2. 不传 `--appcode` 时进入交互模式，提示输入 appcode
3. `app import --file` 只接受升级后的 rabetbase-cli `.rabetbase.json`，并导入运行态顶层配置
4. `--global` 模式下写入 `~/.lovrabet.json`
5. 默认生成顶层 `appcode + env` 结构；如导入到默认应用，会额外写入 `defaultApp`

如果源文件还是旧结构，先运行：

```bash
rabetbase project upgrade
```

## 生成结果

典型输出：

```bash
Initialized .lovrabet.json
  appcode: app-xxxxxxxx
  env:     daily
```

典型文件内容：

```json
{
  "appcode": "app-xxxxxxxx",
  "env": "daily"
}
```

## 与 app cache 的关系

- `app init` 只负责初始化本地用户意图配置
- `app init` 不会把平台应用列表写入配置
- `app init` 也不会主动刷新 cache
- 需要发现当前 AK 可见应用时，执行：

```bash
lovrabet app list
```

## 参考

- [SKILL.md](../SKILL.md)
- [lovrabet-config.md](lovrabet-config.md)
- [lovrabet-app.md](lovrabet-app.md)
