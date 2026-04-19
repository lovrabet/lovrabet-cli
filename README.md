# Lovrabet CLI

[Lovrabet](https://www.lovrabet.com)（中文名云兔）是企业级 AI-Native 业务系统生成平台。`lovrabet` 是它的运行态 CLI，面向业务人员、交付实施、集成开发者和 AI 助手，提供应用切换、数据集发现、数据查询与写入、SQL 执行、BFF 调用等能力。

这个仓库同时提供两部分内容：

* `lovrabet` CLI 的公开安装入口
* 面向 AI IDE / Agent 的 `lovrabet` Skill 安装源

## 安装步骤

### 1. 安装 Lovrabet CLI

```bash
npm install -g @lovrabet/lovrabet-cli@latest
```

安装完成后，可先验证版本和帮助信息：

```bash
lovrabet --version
lovrabet --help
```

### 2. 安装 Lovrabet Skill

如果你在 Cursor、Claude Code、Codex、Windsurf 等支持 `skills` 的 AI 开发环境中使用 Lovrabet，推荐继续安装 Skill：

```bash
npx skills add lovrabet/lovrabet-cli -g -y
```

Skill 会把 Lovrabet 的命令使用顺序、应用决议规则、真实查数方法、风险边界和排障经验提供给 AI 助手，减少乱猜 app、乱猜字段、误用写命令的问题。

### 3. 登录并开始使用

```bash
lovrabet auth login
lovrabet app list
```

如需在当前项目固定默认应用：

```bash
lovrabet app init --appcode <code>
```

## CLI 能做什么

Lovrabet CLI 主要覆盖这些运行态场景：

* 应用目录与默认应用切换
* 数据集发现与字段结构查看
* `data filter`、`data getOne`、`data aggregate` 等真实查数能力
* `data create`、`update`、`delete` 等数据修正能力
* `sql exec` 的只读查询与校验
* `bff exec` 的运行态函数联调

对于做业务排查、交付联调、数据核对、补数修数、接口验证，CLI 通常比进后台点页面更直接。

## 推荐上手顺序

```bash
npm install -g @lovrabet/lovrabet-cli@latest
npx skills add lovrabet/lovrabet-cli -g -y
lovrabet auth login
lovrabet app list
lovrabet dataset list --name 客户
lovrabet dataset detail --code <datasetCode>
lovrabet data filter --code <datasetCode> --params '{"currentPage":1,"pageSize":20}'
```

如果要让 AI 助手帮你生成页面、排查字段格式、判断枚举/日期/数组/JSON 的真实结构，优先先跑 `dataset detail`，再跑 `data filter` 或 `data getOne` 看真实返回。

## Skill 适合什么场景

安装 Skill 后，AI 助手更适合处理这些任务：

* 先找正确的 app，再定位 dataset 和字段
* 用真实数据判断枚举、日期、数组、JSON、空值和 `_label` 字段格式
* 区分 `dataset`、`data`、`sql`、`bff` 的职责边界
* 按 Lovrabet CLI 既有约束生成更稳妥的命令和排障步骤

## 仓库结构

```text
README.md
LICENSE
NOTICE
skills/lovrabet/
  SKILL.md
  references/
  guides/
```

## 进一步了解

* 官网：[www.lovrabet.com](https://www.lovrabet.com)
* 开发文档：[open.lovrabet.com](https://open.lovrabet.com/)
* CLI 帮助：`lovrabet --help`
* Skill 说明：[`skills/lovrabet/SKILL.md`](skills/lovrabet/SKILL.md)

## License

Licensed under [Apache-2.0](LICENSE). Redistributions and derivative works must
retain the original copyright, license, and attribution notices required by the
Apache License.
