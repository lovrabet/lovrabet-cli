# Lovrabet CLI Skill

面向 [skills](https://www.npmjs.com/package/skills) 开放生态的 **Lovrabet Runtime CLI** Agent Skill 发布仓库。

[Lovrabet](https://www.lovrabet.com)（中文名云兔）是企业级 AI-Native 业务系统生成平台。**Lovrabet Runtime CLI** 提供运行态数据查询、数据 CRUD、SQL 执行与 BFF 调用能力，让业务人员、集成开发者和 AI 助手可以直接在终端里访问应用真实数据与运行能力。

## 为什么需要这个 Skill

在运行态场景里，AI 助手不仅要“调用一个命令”，还要理解：

* 当前应用如何决议
* `dataset detail` 和 `data filter/getOne` 的使用顺序
* `data / sql / bff` 的不同边界
* Access Key、配置作用域和风险控制
* 结构化输出、`--format compress` 与 `--jq` 的正确用法

这个 Skill 把这些运行态工作流和约束固化下来，避免 AI 助手在真实项目里乱猜应用、乱猜字段、或错误使用写命令。

## 安装

推荐在项目根目录使用 `skills` CLI 安装：

```bash
npx skills add lovrabet/lovrabet-cli
```

安装后，Skill 位于 `skills/lovrabet/` 下，遵循标准技能包结构。

## 前置条件

使用 Skill 前，确保已安装 `lovrabet` CLI：

```bash
npm install -g @lovrabet/lovrabet-cli
```

并完成认证与应用配置：

```bash
lovrabet auth login
lovrabet app list
```

如需固定当前项目的默认应用，再执行：

```bash
lovrabet app init --appcode <code>
```

## 仓库结构

```text
README.md           ← 本说明
LICENSE
skills/lovrabet/
  SKILL.md          ← Skill 入口（意图路由 + 命令索引 + Agent 规则）
  references/       ← 命令参考文档
  guides/           ← 运行态工作流与排障指南
```

## 适用场景

如果你在做这些事情，这个 Skill 会特别有帮助：

* 在多应用上下文中找到正确的 app、dataset 和数据记录
* 用 `data filter` / `data getOne` 查看真实数据结构
* 用 `sql exec` 执行只读查询
* 用 `bff exec` 调试运行态函数
* 让 AI 助手按照 Lovrabet 运行态约束生成更稳妥的操作步骤

## 进一步了解

* 官网：[www.lovrabet.com](https://www.lovrabet.com)
* 开发文档：[open.lovrabet.com](https://open.lovrabet.com/)
* CLI 文档：`lovrabet --help`
* Skill 详细说明：[`skills/lovrabet/SKILL.md`](skills/lovrabet/SKILL.md)

## License

[MIT](LICENSE)
