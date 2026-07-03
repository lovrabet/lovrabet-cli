# skill install

安装当前应用、当前租户可生效的业务 Skill 到用户级 Agent Skill 目录，让本机 Agent 可发现并使用这些业务 Skill。

## 用法

```bash
lovrabet skill install [--scope all|personal|company] [--code <skillCode>]
```

## 说明

- 需要 AccessKey 和 App Code。
- 默认等价于 `--scope all`，安装当前应用下当前用户可见的 personal/company 业务 Skill。
- personal 与 company 出现同一 `skillCode` 时，personal 版本生效。
- `--scope personal` 只安装个人业务 Skill；`--scope company` 只安装公司业务 Skill。
- `--code` 只安装指定业务 Skill，并清理该 code 对应的失效链接或 cache。
- 安装产物面向 Agent 消费，不建议直接编辑；新建或修改业务 Skill 时使用 `lovrabet skill create` / `lovrabet skill validate` / `lovrabet skill push`。

## CLI Built-in Skill

`lovrabet skill install` 不再安装 CLI 运行时依赖的内置 Skill。需要安装或刷新 CLI Built-in Skill 时使用：

```bash
lovrabet cli-skill install
```
