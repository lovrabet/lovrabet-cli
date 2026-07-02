# workspace — 工作目录配置

`workspace` 用来给**当前工作目录**绑定默认应用。适合 WorkBuddy、Agent 项目目录、客户交付目录这类“打开一个目录就知道默认操作哪个应用”的场景。

它解决的是本地使用体验，不是平台权限配置：

- 配置文件固定写到当前目录的 `.lovrabet.json`
- 不写 AccessKey
- 不同步远端应用目录
- 不影响其他目录

## 命令

```bash
lovrabet workspace init --appcode <appcode> [--env daily]
lovrabet workspace init --app <name> [--env daily]
lovrabet workspace use --appcode <appcode> [--env daily]
lovrabet workspace use --app <name> [--env daily]
lovrabet workspace use --app <name> --appcode <appcode> [--env daily]
```

`init` 和 `use` 当前写入规则一致：

- `init`：第一次给目录建立 Lovrabet 应用上下文时更直观
- `use`：切换当前目录默认应用时更直观

## 参数

| 参数 | 说明 |
|------|------|
| `--app <name>` | 应用名称。CLI 会用当前 AK 在指定环境的已发布应用列表中解析 appcode |
| `--appcode <code>` | 直接写入 appcode，不需要远端查询 |
| `--env <env>` | 可选。写入当前目录配置，取值为 `production` / `development` / `daily` |
| `--yes` | 兼容无打扰脚本参数；当前命令不会交互确认 |

至少需要传 `--app` 或 `--appcode` 之一。

## 写入效果

直接用 appcode：

```bash
lovrabet workspace init --appcode app-64e32817 --env daily
```

当前目录 `.lovrabet.json`：

```json
{
  "env": "daily",
  "appcode": "app-64e32817"
}
```

用业务名称绑定默认应用：

```bash
lovrabet workspace use --app crm --env daily
```

当前目录 `.lovrabet.json`：

```json
{
  "env": "daily",
  "defaultApp": "crm",
  "apps": {
    "crm": {
      "appcode": "app-64e32817"
    }
  }
}
```

已知名称和 appcode 时可以避免远端查询：

```bash
lovrabet workspace use --app crm --appcode app-64e32817 --env daily
```

## 注意事项

- 不要主动创建或修改工作目录配置；只有用户明确要求“初始化当前目录”“记住这个应用”“绑定这个工作目录”等语义时，才执行 `workspace init/use`
- 如果连续多次在同一目录使用同一个 app 或 appcode，可以提醒用户是否要写入当前目录 `.lovrabet.json`，但必须先得到明确同意
- `--app <name>` 需要当前已有合法 AccessKey，因为 CLI 要查询当前 AK 可见的应用列表
- 未发布应用不能被绑定为工作目录默认应用
- 如果只是临时执行一次命令，优先直接在目标命令上传 `--app` 或 `--appcode`
- `app init` 和 `app use` 仅作为兼容入口保留；新脚本和文档都使用 `workspace init/use`
