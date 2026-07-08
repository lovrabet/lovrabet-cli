# OCR 识别工作流

`ocr` 命令用于对远程 URL 或本地文件执行 OCR 识别。适用场景包括发票识别、证照识别、表格图片识别、合同或附件内容提取，以及把识别结果整理后写入业务数据。

## 命令概览

| 目标 | 命令 | 风险 |
|------|------|------|
| 识别远程 URL | `lovrabet ocr recognize --scene <scene> --image-url <url>` | write |
| 识别本地文件 | `lovrabet ocr recognize --scene <scene> --image-file <local-path>` | write |

需要机器可读输出时统一加 `--format compress`；需要截取字段时再叠加 `--jq`。

`ocr recognize` 支持 dry-run 预览。正式识别前先用 `--dry-run` 确认目标应用、输入方式和服务端调用链路；dry-run 不上传文件、不调用 OCR。

## 识别场景

支持的 `--scene`：

| scene | 适用 |
|-------|------|
| `invoice` | 发票、票据 |
| `general` | 通用图片或文档文本 |
| `form` | 表格图片 |
| `idCard` | 证照、身份类图片 |

## 输入方式

识别远程 URL：

```bash
lovrabet ocr recognize --scene invoice --image-url <url> --dry-run --format compress
lovrabet ocr recognize --scene invoice --image-url <url> --format compress
```

识别本地文件：

```bash
lovrabet ocr recognize --scene invoice --image-file ./invoice.png --dry-run --format compress
lovrabet ocr recognize --scene invoice --image-file ./invoice.png --format compress
```

本地文件识别会复用文件上传能力，自动串联：

1. `file upload --file <local-path>`
2. `file query-url --filepath <filePath>`
3. `ocr recognize --image-url <temporary-url>`

文件上传和临时 URL 的细节见 [文件上传工作流](lovrabet-file-workflow.md)。

当前 OCR 输入必须二选一：`--image-file` 或 `--image-url`。当前服务端上传接口不返回稳定 `fileId`，OCR 接口也不接受 `fileId`，所以 CLI 不暴露 `--file-id`；不要把 `filePath` 当作 `fileId` 使用。

## 结果处理

OCR 返回以运行态识别结果为准，常见字段包括 `type`、`text`、`lines`、`requestId`，本地文件识别还会补充 `sourceFile.fileName` 和 `sourceFile.filePath`。

处理建议：

1. 向用户总结识别结论时，优先提取关键业务字段，不粘贴大段原文。
2. 需要写入数据集时，先 `dataset detail --code <datasetCode>` 确认字段。
3. 使用 `data create` 或 `data update` 前先 dry-run。
4. 发票、证照、合同等敏感材料不要写入日志、配置文件或 Skill 文档示例。

示例：识别发票并准备写入数据集。

```bash
lovrabet ocr recognize --scene invoice --image-file ./invoice.png --format compress
lovrabet dataset detail --code <datasetCode> --format compress
lovrabet data create --code <datasetCode> --params '{"invoiceNo":"<识别后的发票号>","amount":100}' --dry-run
```

## 常见错误

| 现象 | 处理 |
|------|------|
| 本地文件不存在 | 检查 `--image-file` 路径，使用真实本地文件 |
| 文件超过 50 MB | 拆分或压缩文件；本地文件识别会先走当前运行态上传接口 |
| URL 过期或不可访问 | 重新执行 `file query-url` 获取新的临时 URL |
| OCR 场景不匹配 | 换用 `general`、`form`、`invoice` 或 `idCard` |
