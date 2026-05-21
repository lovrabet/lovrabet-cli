# skill pull / push

同步运行态 Skill 到本地 agent skill 目录，并将本地个人 Skill 改动推回平台。

## 拉取

```bash
lovrabet skill pull
lovrabet skill pull --code sales_playbook
```

默认拉取当前 App 下当前 AK 可见的 `personal` 和 `company` Skill，不拉取 `builtin` 或 `public`。

拉取目录：

```text
~/.lovrabet/cache/<env>/<ak_fingerprint>/skills/<appCode>/<scope>/<skillCode>/
  SKILL.md
  lovrabet.skill.json
```

- `SKILL.md` 是后端 Skill `content` 原文
- `lovrabet.skill.json` 保存 `appCode`、`skillCode`、`scope`、版本、标签、content hash 等 Lovrabet 元数据
- 同名 `skillCode` 的 personal 和 company 副本都会保留

## 本地链接

有效 Skill 会链接到：

```text
~/.agents/skills/<appCode>--<skillCode>
~/.claude/skills/<appCode>--<skillCode>
```

personal 与 company 同名时 personal 优先；只有 company 时链接 company。CLI 只替换指向 `~/.lovrabet` cache 的已管理 symlink，不覆盖普通文件、目录或外部 symlink。

## 推送

```bash
lovrabet skill push --dir ~/.lovrabet/cache/production/<ak_fingerprint>/skills/app-1/personal/sales_playbook
lovrabet skill push --dir ./app-1--sales_playbook
```

`push` 读取目录下的 `SKILL.md`。有 `lovrabet.skill.json` 时优先使用其中的 `skillCode`、名称、描述、标签和版本；没有元数据时从目录名推导 `skillCode`，并去掉当前 App 的 `<appCode>--` 前缀。

`push` 只创建或更新 personal Skill。若元数据 scope 是 `company`、`builtin` 或 `public`，命令会在调用后端前失败。Skill 子命令包含 `install`、`pull`、`push`。
