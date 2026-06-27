# Service Tree 业务服务树

Service Tree 把低层 `data` / `sql` / `bff` 命令映射成面向业务的多级命令，例如 `lovrabet crm customer list`。本地 registry 位于 `~/.lovrabet/service.json`；首期 JSON 文件通常由 Git 管理，再通过 `service import` 安装到本机。

## 本地管理命令

```bash
lovrabet service validate --file ./lovrabet-services/crm.service.json
lovrabet service import --file ./lovrabet-services/crm.service.json --dry-run
lovrabet service import --file ./lovrabet-services/crm.service.json
lovrabet service list
lovrabet service detail --service crm
lovrabet service export --service crm --file ./lovrabet-services/crm.service.json
lovrabet service remove --service crm
```

`service validate` 只校验 JSON 协议，不写入本地 registry。`service import` 会把 manifest 标准化后写入 `~/.lovrabet/service.json`，同一个 `service.code` 再次导入会替换旧版本。`service export` 从本地 registry 导出 Git 可管理的 manifest 文件。`service remove` 只移除本机 registry 条目，不删除 Git 中的 manifest 文件，也不会访问服务端。

## 发现与诊断

导入后的业务服务会出现在 CLI 发现面：

```bash
lovrabet --help
lovrabet schema --format compress
lovrabet doctor
```

`lovrabet --help` 会列出本地已导入服务及其多级业务命令；`lovrabet schema` 会把动态业务命令作为机器可读 command schema 导出；`lovrabet doctor` 会展示本机 registry 路径、加载状态、服务数、命令数和每个服务的 manifest 来源文件。

## 动态业务命令

导入后，CLI 会在静态命令未命中时尝试匹配本地 Service Tree：

```bash
lovrabet project issues list --status new --format compress
lovrabet project issues detail 445 --format compress
lovrabet project solution list --requirement-id 445 --format compress
lovrabet project comment list --requirement-id 445 --format compress
```

动态命令仍走 Lovrabet CLI 的统一 runner：认证、appcode 决议、已发布应用访问限制、risk、dry-run、`--format` 和 `--jq` 都会生效。manifest 的 `appBindings` 可提供默认 app/appcode/env；用户显式传入的 `--appcode`、`--app`、`--env` 优先。

## 参数映射

manifest 中的业务字段名可以使用 camelCase，例如 `startDate`；CLI flag 统一使用 kebab-case：

```bash
lovrabet crm customer list --start-date 2026-01-01
```

`mapTo` 决定业务参数映射到底层 Lovrabet 原始参数，例如：

```json
{
  "flags.startDate": {
    "target": "where.created_at",
    "operator": "$gte"
  }
}
```

对应的底层参数等价于：

```bash
lovrabet data filter --code <datasetCode> --params '{"where":{"created_at":{"$gte":"2026-01-01"}}}'
```

`mapTo` 也支持运行时上下文来源，当前用于表达“我的数据”等业务语义：

```json
{
  "context.userId": {
    "target": "where.assignee_id",
    "operator": "$eq",
    "transform": "number"
  }
}
```

对应业务命令可写成：

```bash
lovrabet project issues mine
```

执行时 CLI 会用当前 AccessKey 查询登录用户信息，并把 `context.userId` 映射到底层 `data filter` 参数；不要在 manifest 中硬编码个人用户 ID。

## Dataset 解析

Service Tree 的 data target 推荐直接配置 `datasetCode`。如果只配置 `datatable`，动态命令执行时会在当前 app 的数据集列表中按物理表名解析对应 dataset code；解析失败时命令会停止，并提示补充 `datasetCode` 或确认表名。

## 边界

- Service Tree 本地 registry 是运行态发现和执行入口，不是服务端配置中心。
- `service import/export` 只管理本机 `~/.lovrabet/service.json`，不会上传服务端。
- 动态命令不会绕过应用发布状态和当前 AK 权限。
- rabetbase-cli 不管理 runtime registry；如需研发态元数据辅助，应显式使用 rabetbase 的 dataset/sql/bff/page 命令。
