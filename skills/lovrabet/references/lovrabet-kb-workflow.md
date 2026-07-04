# 知识库工作流

`kb` 命令用于当前应用下的 personal 知识库和可见知识检索。第一阶段支持 `list`、`detail`、`create`、`update`、`search`；不提供删除命令。

## 查看与检索

```bash
lovrabet kb list --format compress
lovrabet kb detail --id <id> --format compress
lovrabet kb search --query "订单审批" --topk 5 --format compress
```

`kb search` 会返回当前用户可见的公司知识和个人知识，使用 `scope` 区分来源。

## 写入顺序

1. 先确认知识库 title 和 content；内容必须是用户明确提供或已经在对话中确认的信息。
2. 将正文写入当前 workspace 下的 UTF-8 Markdown 或文本文件，例如 `.lovrabet/kb/<safe-title>.md`。
3. 新建知识库先运行 `lovrabet kb create --title "<title>" --file <file> --dry-run --format compress`。
4. 用户明确要求保存时，再运行不带 `--dry-run` 的 `lovrabet kb create`。
5. 更新知识库时先用 `lovrabet kb list`、`lovrabet kb search --query ...` 或用户提供的 ID 定位条目。
6. 更新前运行 `lovrabet kb detail --id <id> --format compress`。
7. 更新先运行 `lovrabet kb update --id <id> --file <file> --dry-run --format compress`。
8. 用户明确要求保存时，再运行正式 `lovrabet kb update`。
9. 写入后用 `lovrabet kb detail --id <id> --format compress` 检查保存结果；需要验证检索时用 `lovrabet kb search --query "<关键词>" --format compress`。

## 创建

知识正文来自本地 UTF-8 文本或 Markdown 文件，不支持内联正文参数：

```bash
lovrabet kb create --title "订单审批规则" --file ./approval.md --dry-run
lovrabet kb create --title "订单审批规则" --file ./approval.md
```

## 更新

更新前先读详情，确认当前标题、正文、版本和 RAG 状态：

```bash
lovrabet kb detail --id <id> --format compress
lovrabet kb update --id <id> --file ./approval-v2.md --dry-run
lovrabet kb update --id <id> --title "新版订单审批规则" --file ./approval-v2.md
```

省略 `--title` 时，CLI 会读取当前条目并保留原标题。

## RAG 状态

create/update 返回后要检查 `ragStatus` 和 `ragErrorMessage`。如果状态仍在处理或有错误，只能说条目已经写入，不能声称知识检索或聊天使用已经端到端可用。

推荐：

```bash
lovrabet kb detail --id <id> --format compress
lovrabet kb search --query "刚写入的关键词" --format compress
```

确认命中后再把检索可用性写进交付说明。
