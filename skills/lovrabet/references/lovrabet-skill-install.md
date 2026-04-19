# skill install

通过官方 `skills` CLI 全局安装 `lovrabet` 运行态 Skill。

## 用法

```bash
lovrabet skill install
```

## 实际执行的外部命令

```bash
npx skills add lovrabet/lovrabet-cli -g -y
```

## 说明

- 该命令不依赖 `accessKey` 或 `appcode`
- 安装的是 GitHub 发布仓库中的 `skills/lovrabet/`
- 适用于首次安装、重装或刷新本地 Skill
- 需要本机可访问 npm 与 GitHub

## 跳过外部安装

测试或离线场景可设置：

```bash
LOVRABET_SKIP_NPX_SKILLS=1 lovrabet skill install
```

此时命令会跳过 `npx` 调用，并假定 Skill 已存在。
