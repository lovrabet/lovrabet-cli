# 审批流运行态办理

Runtime CLI 只负责当前 AccessKey 用户的运行态任务查询和办理，不管理流程定义。当前用户由 `X-User-AK` 对应的服务端认证上下文确定，命令不接受 `--user-id`。

所有审批命令都需要 `appCode` 用于应用权限校验。可传 `--app-code <APP_CODE>`；未传时使用当前工作区解析出的应用。没有可用应用时先绑定工作区或显式传参。

## 查询待办

```bash
lovrabet approval todo --app-code <APP_CODE>
lovrabet approval todo --app-code <APP_CODE> --page 2 --page-size 20 --format compress
```

待办参数是 `appCode`、`currentPage` 和 `pageSize`，任务详情通过 query 传 `appCode`，批准、拒绝、转交通过请求体传 `appCode`。所有请求使用 `/client/` 入口和当前 User AK，应用范围由服务端校验。

## 查看任务

```bash
lovrabet approval task-detail --task-id <TASK_ID> --format json
```

详情中的 `canHandle`、`canCancel`、`transferCandidates`、`formData` 和流程上下文以 Runtime 返回为准。输出会遮罩敏感键；如响应包含 `variables`，其中的值只显示脱敏占位符。

## 审批和拒绝

先预览请求：

```bash
lovrabet approval approve --task-id <TASK_ID> --comment '同意' --dry-run
lovrabet approval reject --task-id <TASK_ID> --comment '需要补充材料' --dry-run
```

确认后执行：

```bash
lovrabet approval approve --task-id <TASK_ID> --comment '同意' --yes
lovrabet approval reject --task-id <TASK_ID> --comment '需要补充材料' --yes
```

`approve` 和 `reject` 都调用 Runtime 的 `/client/flow/approve`，区别只在请求体的 `approved` 值。流程后续路径由已发布流程定义决定，CLI 不自行解释。

## 转交

目标用户应来自任务详情的 `transferCandidates` 或其他已确认的 Runtime 资源：

```bash
lovrabet approval transfer \
  --task-id <TASK_ID> \
  --target-user-id <USER_ID> \
  --comment '请协助处理' \
  --dry-run

lovrabet approval transfer \
  --task-id <TASK_ID> \
  --target-user-id <USER_ID> \
  --comment '请协助处理' \
  --yes
```

最终授权和候选人合法性由 Runtime 校验。

## Variables 与安全

审批变量支持内联 JSON 或文件：

```bash
lovrabet approval approve --task-id <TASK_ID> \
  --variables '{"amount":100}' --yes
lovrabet approval approve --task-id <TASK_ID> \
  --variables @variables.json --yes
```

变量解析失败时不会发送请求。dry-run、错误重试命令和本地命令日志不会回显变量值、AccessKey、token 或 secret。

高风险命令在非交互模式必须传 `--yes`；`--dry-run` 只生成脱敏预览，不执行写请求。

## 表单数据与流程发起

对于 `FORM_FLOW` 表单流程，创建业务记录和发起审批是两个步骤。`data create` 成功只表示业务记录已创建，不表示流程已启动；需要审批的业务不能把它当作完整提交流程。

本 CLI 已提供审批任务的 `todo`、`task-detail`、`approve`、`reject`、`transfer`；没有表单 `start` 或流程实例查询命令。任务要求发起流程时，使用已授权的应用页面或可信业务 Skill 明确绑定的业务入口；不能猜测 `/client/flow/start`，也不能把 User AK 发往浏览器登录接口。先创建业务记录再由浏览器或其他已授权入口启动流程的既有顺序保持不变。

`FORM_FLOW` 启动的最小契约是 `appCode`、本次实际使用的单个 `pageId` 和已创建记录的 `dataId`；`datasetCode` 可省略，后端按页面的标准主表单解析，也可显式传入已确认的编码。此可选规则仅针对表单启动，普通 data 模型选择和独立流程合同不因此改变，也不能用 `datasetCode` 推断唯一页面或流程。

服务端会自动将 `processInstanceId` 绑定到表单业务记录。普通 `data create` / `data update` 不要手写或清空 `processInstanceId` / `process_instance_id`。记录中的 PID 只表示关联，不表示流程正在运行或已经完成。`instanceStatus`、`runningNode`、`nodeStartTime`、`nodeProcessUser`、`flowStatus` 不属于默认返回和同步维护的流程快照；如果模型实际配置了这些同名业务字段，仍按其业务契约使用，不能用它们代替流程查询。

提交超时或结果不确定时，先回读业务记录中的 PID，再使用真实已授权的流程查询入口核实。当前 CLI 没有流程实例查询命令，无法仅凭 CLI 断言运行或完成；待办为空也不能推断 `completed`，不要盲目重试启动。
