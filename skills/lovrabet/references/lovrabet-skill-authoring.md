# Skill 创建、更新与发布工作流

`lovrabet skill create/validate/install/list/push` 用于当前应用下 personal/company 业务 Skill 的本地开发和同步。Builtin Skill 由 sandbox 镜像或 CLI Built-in Skill 安装源提供，不通过 `lovrabet skill install` 拉取。

## 目录约定

所有新建或更新的 Skill 都在当前 workspace 的 `.agents/skills/<skillCode>/` 下完成。

- `skillCode` 使用简短英文 kebab-case，不超过 64 字符。
- `SKILL.md` 必须位于 Skill 目录根部。
- `lovrabet.skill.json` 必须位于 Skill 目录根部。
- 需要较长规则、schema、示例或脚本时，放入 `references/`、`scripts/` 或 `assets/`，并在 `SKILL.md` 中说明读取时机。
- `SKILL.md` frontmatter 中 `name` 固定写稳定 `skillCode`，不要写中文；中文或业务展示名写顶层 `displayName`，即使展示名暂时等于 `skillCode` 也应显式保留。
- 顶层 `example` 是一条用户可以直接发送、用于触发该 Skill 的推荐话术，例如 `example: 上传这份产品资料`。
- `example` 可选；缺失、空白或不是单个标量时，`lovrabet skill validate` 会给出 warning，但不阻断 push。
- 不要把正文说明、内部 CLI 命令、执行步骤或多个示例写入 `example`，也不要自动生成该字段。

## 新建

```bash
lovrabet skill create --name "<skill-name>" --type read --dir .agents/skills/<skillCode>
lovrabet skill validate --dir .agents/skills/<skillCode> --strict
lovrabet skill push --dir .agents/skills/<skillCode> --format compress
```

`type` 按用途选择 `read`、`write` 或 `trainer`。`displayName` 是给人看的展示名，`description` 是触发依据，要说明 Skill 做什么、什么时候使用、什么时候不要使用；`example` 则是一句推荐给用户直接发送的最简触发话术。

## 更新

本地已有目录时，直接编辑 `.agents/skills/<skillCode>/SKILL.md` 和必要资源。当前 workspace 没有目标 Skill 时，先安装当前应用可见的业务 Skill：

```bash
lovrabet skill install --code <skillCode> --format compress
lovrabet skill validate --dir .agents/skills/<skillCode> --strict
lovrabet skill push --dir .agents/skills/<skillCode> --format compress
```

如果安装得到的是 company Skill，默认不要覆盖 company 源；新建或更新 personal 副本，通过 `lovrabet skill push` 保存为个人 Skill。

## 发布扫描告警

personal `skill push --dry-run` 会用 `visibility=PRIVATE` 调用 SkillHub publish validate，提前返回正式发布会遇到的 errors 和 warnings。errors 始终阻断；正式 push 遇到 warnings 时会展示告警并停止。人工复核后，personal 或 company push 都需显式添加 `--confirm-warnings`，提交未经改写的同一份包。CLI 不根据 warning 文案自行分类。

发布扫描告警是打包反馈，不是修改业务 Skill 的授权。不要为了通过扫描删除、替换或改写业务 Skill 中的标识、映射、路径、规则、scripts 或 references。先确认告警内容：

- 真实问题（例如明文凭证、违规扩展名或内容格式不匹配）：personal push 不会因 warning 自动停止，仍需根据返回结果尽快修复；涉及凭证时还必须轮换。
- 已确认的误报：保留源码，不要创建内容不同的临时发布副本。

CLI 不自动创建脱敏包，也不自动改写源 Skill。

## 发布公司级版本

本地目录确认可交付后，可提交公司级 Skill 新版本审核：

```bash
lovrabet skill validate --dir .agents/skills/<skillCode> --strict
lovrabet skill push --scope company --dir .agents/skills/<skillCode> --dry-run
lovrabet skill push --scope company --dir .agents/skills/<skillCode> --confirm-warnings --format compress
```

`push --scope company` 使用 SkillHub `NAMESPACE_ONLY` 可见性提交审核。命令成功只表示新版本已提交 review；审核通过前不要声称该版本已对所有成员生效。需要本机 Agent 使用生效版本时，审核后再执行 `lovrabet skill install --code <skillCode>`。

## 内容要求

- `SKILL.md` 面向未来 Agent，使用精简 Markdown。
- 正文写可执行流程、命令、边界和输出要求。
- 不引用 RuntimeAgent 原生 `skill_load`、`skill_update`，也不要把不存在的 pull 类 Skill 操作写成 `lovrabet` 命令。
- 不把明文凭证、cookie、AK、用户私有路径写入 Skill。
- 不把临时调试记录、变更日志或面向人的安装说明放入 `SKILL.md`。

## 发布后检查

`lovrabet skill push` 成功后，使用下面命令确认本地和云端可见性：

```bash
lovrabet skill list --scope all --format compress
lovrabet skill list --local --scope all --format compress
```

命令失败时报告 CLI 错误和当前 appCode，不声称已保存。
