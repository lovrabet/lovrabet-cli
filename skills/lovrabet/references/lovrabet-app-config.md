# app-config — 运行态 app-config key 状态检查

按当前应用和配置 key 做只读状态检查，确认运行态 app-config key 是否已配置、当前 AccessKey 是否可读取、应用路由是否正确。

app-config 用于应用维度的运行态配置管理。`app-config get` 只检查指定 key 的配置状态和可读性，默认不输出 value，以降低配置内容进入日志、命令参数或 Agent 对话的风险。

## app-config get

```bash
lovrabet app-config get <key>
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `<key>` | string | 是 | app-config key |

**风险等级**：`read`

**输出**：只返回脱敏元数据：

```json
{
  "ok": true,
  "command": "lovrabet app-config get",
  "risk": "read",
  "data": {
    "appCode": "app-xxxxxxxx",
    "key": "example_api_key",
    "tags": ["example"],
    "hasValue": true,
    "valueRedacted": true,
    "gmtModified": "2026-06-23T10:00:00+08:00",
    "gmtCreate": ""
  }
}
```

## 边界

- `lovrabet app-config get` 不支持 `--reveal`，不会输出明文 value。
- `lovrabet` 不提供 app-config 的 set/delete/list 管理入口。
- 不要把 app-config value 写入 `.lovrabet.json`、缓存、日志、命令参数或业务 Skill 入参。
- 它只用于检查 key 是否已配置、权限是否有效、appCode 路由是否正确，不作为业务 Skill 默认取值入口。
- 示例 key 仅用于说明；实际执行必须使用调用方明确给出的 key，不能猜测、按 tags 查找或默认使用示例 key。
- 业务需要使用 app-config value 时，在目标 BFF 中通过 `context.appConfig.get(...)` 读取并消费，见 [lovrabet-bff-workflow.md](lovrabet-bff-workflow.md)；客户端或 Agent 不先 reveal 再传参。
- 输出中的 `valueRedacted: true` 只表示 CLI 已脱敏展示，不表示后端加密状态。

## 示例

```bash
lovrabet app-config get example_api_key --format compress
```

若 key 不存在、当前 AK 无权限或运行态只读接口不可用，命令会返回错误。错误排查只基于 key、appCode、traceId 等定位信息，不应要求用户提供或回显明文 value。

## 参考

- [SKILL.md](../SKILL.md)
- [lovrabet-bff-workflow.md](lovrabet-bff-workflow.md)
- [lovrabet-output-format-jq.md](lovrabet-output-format-jq.md)
