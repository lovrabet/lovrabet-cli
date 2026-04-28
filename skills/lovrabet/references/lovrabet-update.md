# update — 更新 CLI

`lovrabet update` 从 npm registry 查询当前 CLI 包的 dist-tags，不依赖 CDN 配置文件。更新检查完成后会刷新官方 Skill，保证 Agent 本地工作流与 CLI 版本同步。

## 命令

```bash
# 默认等价于 --latest
lovrabet update

# 安装 npm latest dist-tag
lovrabet update --latest

# 安装 npm beta dist-tag
lovrabet update --beta

# 安装指定版本
lovrabet update --version 2.0.6

# 只更新 CLI，不刷新 Skill
lovrabet update --latest --no-skills
```

## 行为

- `--latest`、`--beta`、`--version <version>` 互斥
- 默认目标是 npm `latest` dist-tag
- `--beta` 目标是 npm `beta` dist-tag
- `--version` 必须是完整 semver，例如 `2.0.6` 或 `2.0.7-beta.1`
- 成功检查或更新 CLI 后，默认执行官方 Skill 刷新
- Skill 刷新失败只报警，不让 CLI 更新失败

## 开源配置点

开源二开配置集中在 `src/constant/product.ts`，`src/constant/distribution.ts` 只保留给更新逻辑使用的兼容导出：

| 常量 | 说明 |
|------|------|
| `PRODUCT_CONFIG.cliBinName` | CLI 可执行命令名 |
| `PRODUCT_CONFIG.cliDisplayName` | 帮助与诊断中的 CLI 展示名 |
| `PRODUCT_CONFIG.npmPackageName` | npm 包名 |
| `PRODUCT_CONFIG.skillSource` | `npx skills add` 使用的 skill source |
| `PRODUCT_CONFIG.npmRegistryBaseUrl` | npm registry 地址 |
| `PRODUCT_CONFIG.envPrefix` | 环境变量前缀 |
| `PRODUCT_CONFIG.configFileNames` | 配置文件查找名 |
| `PRODUCT_CONFIG.domains` | 默认服务域名 |
| `PRODUCT_CONFIG.userCenterDisplayName` / `accessKeyCreatePath` | AK 创建提示 |

> 发布二开包时，`package.json` 的 `name` / `bin` 仍需与 `PRODUCT_CONFIG.npmPackageName` / `cliBinName` 保持一致。

## 跳过 Skill 刷新

```bash
lovrabet update --latest --no-skills
```

测试或离线环境也可以复用 skill 安装的跳过环境变量：

```bash
LOVRABET_SKIP_NPX_SKILLS=1 lovrabet update --latest
```
