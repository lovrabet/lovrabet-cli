# skill install / create / validate / list / push

安装当前应用业务 Skill，创建本地运行态 Skill 草稿，检查必要元数据，查看云端或本地 CLI 管理的 Skill 列表，并将本地个人 Skill 改动推回平台。

## 创建本地草稿

```bash
lovrabet skill create --name invoice-review --type write
lovrabet skill create --name customer-summary --type read --target ./skills
lovrabet skill create --name invoice-review --dry-run
```

`create` 只写本地文件，不需要 AK、不需要 appcode、不上传、不发布、不写业务数据。默认生成：

```text
.agents/skills/<skill-name>/
  SKILL.md
  references/runtime-contract.md
  references/output-contract.md
```

生成的草稿带有 `【...】` 占位符。frontmatter `description` 应写模型触发语义，说明何时使用、不要使用的边界和关键词。CLI 只检查 `SKILL.md` 与 frontmatter 必要字段；正文、章节、references、占位符和输出协议不参与 CLI validate 或 push 阻断。

## 必要元数据检查

```bash
lovrabet skill validate --dir .agents/skills/invoice-review
lovrabet skill validate --dir .agents/skills/invoice-review --strict
```

`validate` 只检查本地目录是否具备上传和识别所需的最小信息。`--strict` 为兼容保留，不启用额外检查。

检查项：

- 根目录存在普通文件 `SKILL.md`。
- `SKILL.md` 以 frontmatter 开头。
- frontmatter 中 `name`、`description`、`metadata.type` 非空。
- `metadata.type` 必须是 `write`、`read` 或 `trainer`。

正文内容、章节名称、`references/*`、占位符、输出协议、明文凭证和外部链接不参与 CLI validate。

## 查看列表

```bash
lovrabet skill list
lovrabet skill list --scope personal
lovrabet skill list --scope company
lovrabet skill list --scope all
lovrabet skill list --code sales_playbook
lovrabet skill list --local
lovrabet skill list --local --scope all
```

默认等价于 `--scope all`，查询 SkillHub-backed 云端 personal 和 company 业务 Skill，不安装、不下载内容。Builtin Skill 由 sandbox 镜像内置提供，不通过远端 `skill list` 查询。

`--local` 只读取当前 env、AK、App 下 CLI 管理的本地 cache 与 agent 链接，依赖 `lovrabet.skill.json` 识别元数据；它不请求云端、不下载 package、不物化文件，也不清理旧目录。需要查看本地是否已经安装到机器上时用 `lovrabet skill list --local`，需要看云端是否存在时用不带 `--local` 的 `lovrabet skill list`。

## 安装业务 Skill

```bash
lovrabet skill install
lovrabet skill install --scope all
lovrabet skill install --scope personal
lovrabet skill install --scope company
lovrabet skill install --code sales_playbook
```

默认等价于 `--scope all`，安装当前 App 下当前 AK 可见的 `personal` 和 `company` 业务 Skill，不安装 `builtin`。

安装过程会使用当前环境和 AK 指纹隔离的 CLI 管理缓存：

```text
~/.lovrabet/cache/<env>/<ak_fingerprint>/skills/<appCode>/<scope>/<skillCode>/
  SKILL.md
  lovrabet.skill.json
```

- `SKILL.md` 是本地 Agent 可直接发现的入口文件；后端 Skill `content` 缺少 YAML frontmatter 时，CLI 会根据 Skill 元数据补齐 `name` 和 `description`
- `lovrabet.skill.json` 保存 `appCode`、`skillCode`、`scope`、版本、标签、content hash 等 Lovrabet 元数据
- 同名 `skillCode` 的 personal 和 company 副本都会保留，effective 链接优先 personal
- 完整 `skill install` 会移除当前 App 下远端已删除 Skill 对应的 CLI 管理链接和当前 AK 缓存目录；`--code <skillCode>` 只同步和清理指定 code

## 本地链接

有效 Skill 会链接到：

```text
~/.agents/skills/<appCode>--<skillCode>
~/.claude/skills/<appCode>--<skillCode>
```

personal 与 company 同名时 personal 优先；只有 company 时链接 company。CLI 只替换或移除指向 `~/.lovrabet` cache 的已管理 symlink，不覆盖或删除普通文件、目录或外部 symlink。

## 推送

```bash
lovrabet skill push --dir ~/.lovrabet/cache/production/<ak_fingerprint>/skills/app-1/personal/sales_playbook
lovrabet skill push --dir ./app-1--sales_playbook
```

`push` 读取目录下的 `SKILL.md`。有 `lovrabet.skill.json` 时优先使用其中的 `skillCode`、名称、描述、标签和版本；没有元数据时从目录名推导 `skillCode`，并去掉当前 App 的 `<appCode>--` 前缀。上传前会用该 `skillCode` 在当前 App namespace 下精确查询远端 Skill，命中时先把远端 scope、版本、状态、名称和描述写回 `lovrabet.skill.json`，再重新读取本地目录进入上传逻辑。

`push` 默认创建或更新 personal Skill。若远端刷新后的元数据 scope 是 `company`，默认 personal push 会在上传前失败，并提示使用公司级 push 工作流。Builtin Skill 不能 push，也不能通过 company scope 提交审核。

## 公司级发布

```bash
lovrabet skill push --scope company --dir .agents/skills/sales-playbook --dry-run
lovrabet skill push --scope company --dir .agents/skills/sales-playbook --confirm-warnings
```

`push --scope company` 读取本地 Skill 目录并向 SkillHub 提交公司级新版本审核。`--dry-run` 只调用 SkillHub publish validate，不创建版本或审核任务；正式提交使用 `visibility=NAMESPACE_ONLY`，成功后版本进入审核，不代表已经成为 effective Skill。审核通过并需要本机 Agent 使用时，再运行 `lovrabet skill install` 安装已生效版本。Skill 子命令包含 `install`、`create`、`validate`、`list`、`push`。

## 与 RuntimeAgent 原生 Skill 工具的关系

CLI 的 Skill 能力是目录同步：

- `skill install`：把当前应用下当前 AK 可见的 personal/company Skill 安装到本地 Agent Skill 目录。
- `skill create --name <name>`：生成本地自包含 Skill 草稿，不上传。
- `skill validate --dir <dir>`：检查 Skill 必要元数据。
- `skill list`：查看云端 Skill 列表；`--local` 查看 CLI 管理的本地 cache 和链接。
- `skill push --dir <dir>`：检查必要上传信息后读取本地目录并创建或更新 personal Skill。
- `skill push --scope company --dir <dir>`：检查本地目录并提交 company Skill 新版本审核。

RuntimeAgent 原生的 `skill_load`、`skill_update` 等工具是 Agent 服务内部的资源读写能力，不是 `lovrabet` CLI 命令。写命令步骤时不要把这些原生工具当作 CLI 子命令，也不要要求 coworker 直接调用它们。
