# auth — 认证管理

当前推荐的认证路径是 **User Access Key（client-ak）**。

## auth login — 保存 accessKey

```bash
lovrabet auth login
lovrabet auth login --access-key ak_xxx
lovrabet auth login --global
lovrabet auth login --project
```

**行为**：
- 默认写入全局配置 `~/.lovrabet.json`
- 显式 `--project` 才写项目配置
- 交互模式下，不传 `--access-key` 会提示输入 AK

**适用场景**：
- 想替换当前 AK，但尽量保留已有 `defaultApp`、`format`、`pageSize`、域名覆盖等配置
- 不希望破坏当前作用域里的其他用户意图配置

## auth init — 清空并重建当前作用域认证配置

```bash
lovrabet auth init --access-key ak_xxx
lovrabet auth init --access-key ak_xxx --env daily
lovrabet auth init --project --access-key ak_xxx
```

**行为**：
- 会清空当前作用域下已有的整份 `.lovrabet.json` 配置内容，再仅写回新的认证初始化结果
- 默认写入全局配置 `~/.lovrabet.json`
- 显式 `--project` 才写项目配置
- 支持 `--env`，会和新的 `accessKey` 一起写入

**风险提醒**：
- 这是破坏性操作
- 当前作用域下已有的 `defaultApp`、`format`、`pageSize`、域名覆盖等字段都会被清掉
- 如果只是想换 AK，不要用 `auth init`，优先用 `auth login`

## 什么时候用 login，什么时候用 init

- **保留现有配置，只更新认证**：`lovrabet auth login`
- **当前作用域配置已经混乱，想彻底重来**：`lovrabet auth init`
- **需要同时清空旧配置并重建 env**：`lovrabet auth init --access-key ak_xxx --env daily`

## auth logout — 清除本地 accessKey

```bash
lovrabet auth logout
lovrabet auth logout --project
```

## auth status — 查看当前认证状态

```bash
lovrabet auth status
```

会显示：
- 当前是否已配置 accessKey
- accessKey 来源（global / project / env）
- 当前 env

## 认证优先级

```
CLI flag (--access-key)
  ↓
环境变量 LOVRABET_ACCESS_KEY
  ↓
项目级 .lovrabet.json
  ↓
全局级 ~/.lovrabet.json
```

## 与 app cache 的关系

一旦配置了 AK：
- `lovrabet app list` 可以直接查询当前 AK 在平台上的应用目录
- 结果会缓存在 `~/.lovrabet/cache/<env>/<ak-fingerprint>/my-apps.json`

`auth login` / `auth init` 本身都不会主动把应用目录写进 `.lovrabet.json`。

## Cookie

Cookie 仍保留历史兼容读取，但不再是推荐主路径。新的 Agent 指导和用户文档应默认使用 AK 流程。

## 参考

- [SKILL.md](../SKILL.md)
- [lovrabet-config.md](lovrabet-config.md)
