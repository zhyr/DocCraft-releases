# DocCraft

DocCraft 是面向公文、合同、档案和日常办公的本地桌面文档工作台。文件留在本机，打开、浏览、批注、转换和保存都不上传。

这里提供免费可用的安装包。源代码不在这个仓库，但欢迎真正参与本项目的人获取，申请方式见下方「参与开发」。

## 核心理念

DocCraft 用 Markdown 作为多种格式之间的中间结构，也作为结构化存储的媒介。

Word、Excel、PowerPoint、PDF、OFD、EPUB 这些格式先被读成同一份结构化中间态，再按需要写回或导出；标题层级、段落、表格、批注、页眉页脚都落在这套结构上，而不是各自为政的一堆二进制块。

这样做的目的是让同一个人、多个 agent 和多种工具能在同一份文档上协作：纯文本人能直接读懂，agent 能稳定定位和改写，格式转换也不必为每种格式写死一套补丁。办公场景里反复出现的「换个格式再改一遍」，本质是数据没有统一的落脚点，DocCraft 想把这个落脚点收敛到 Markdown。

## 下载

当前版本：**1.0.30**（2026-11-28）

系统：macOS 12 及以上，Apple 芯片（arm64）与 Intel（x86_64）通用

安装包在 [Release 1.0.30](https://github.com/zhyr/DocCraft-releases/releases/tag/v1.0.30)：

- 文件：`DocCraft-macos-universal.zip`
- SHA-256：`613cfd618b2cca87764d5892a3cd0e35f52ee082e18c96f5a6399f3231f6b2e3`
- 版本哈希：`1c1b4007`

请先删除「应用程序」里的旧 DocCraft.app，再用访达双击这个 zip 解压，把解压出的 `DocCraft.app` 拖进「应用程序」。不要直接传送已经解压的 `.app`。

这个包已用 Developer ID 签名，并完成 Apple 公证、装订票据。

### 安装

1. 下载 zip 并解压。
2. 将 `DocCraft.app` 拖到「应用程序」。
3. 从「应用程序」打开。若系统提示该应用来自互联网，选择打开。

## MCP：让其他 AI Agent 读写本机文档

DocCraft 安装包内置 `doccraft-mcp`，通过 **Model Context Protocol（MCP）over stdio** 向 Claude Desktop、Cursor 及其他 MCP 客户端提供本机文档能力。无需启动 DocCraft 窗口，也无需编译源码或另外部署索引服务。

它可以用于：

- 读取文档元数据、目录和完整段落。
- 用 BM25 按问题检索相关片段，并拼接成可交给模型的上下文。
- 按段落、单元格、PDF/OFD 页码与坐标修改文件，或全文重写。
- 将多份文件建成不可变快照，跨文档检索、汇总和比较版本。

MCP 的解析和检索在本机完成。返回给 Agent 的内容是否会发往模型服务，取决于所用客户端和模型配置。

### 1. 确认内置服务

将 `DocCraft.app` 安装到「应用程序」后，服务入口为：

```bash
/Applications/DocCraft.app/Contents/MacOS/doccraft-mcp
```

无需手动启动 DocCraft。可在终端检查服务和工具定义：

```bash
/Applications/DocCraft.app/Contents/MacOS/doccraft-mcp --version
/Applications/DocCraft.app/Contents/MacOS/doccraft-mcp --list-tools
```

预期版本输出为 `doccraft-mcp 1.0.30`。如果应用安装在其他位置，将配置中的 `/Applications/DocCraft.app` 替换为实际路径。

不带参数运行时进入持续 stdio 服务，stdin 关闭后自动退出。日常使用时由 MCP 客户端自动启动，不需要把它作为后台服务常驻，也没有 HTTP 端口或 API Key。

### 2. 配置 MCP 客户端

在支持 JSON 配置的客户端中添加以下 `mcpServers` 配置：

```json
{
  "mcpServers": {
    "doccraft": {
      "command": "/Applications/DocCraft.app/Contents/MacOS/doccraft-mcp",
      "args": [],
      "env": {
        "QT_QPA_PLATFORM": "offscreen"
      }
    }
  }
}
```

常见配置位置：

- Claude Desktop：`~/Library/Application Support/Claude/claude_desktop_config.json`
- Cursor：用户级 `~/.cursor/mcp.json`，或项目级 `.cursor/mcp.json`
- 其他客户端：在其 MCP 设置中新增一个 **stdio** 服务，使用相同的 `command`、`args` 和可选 `env`

保存后重启客户端或重新加载 MCP 配置，然后在工具列表中确认能看到 `doccraft_get_metadata`、`doccraft_search_context` 等工具。

#### 配置项说明

| 配置项 | 必需 | 说明 |
| --- | --- | --- |
| `command` | 是 | 安装包内的 `doccraft-mcp` 绝对路径，不要写成源码构建目录 |
| `args` | 是 | 保持 `[]`；无参数表示进入 stdio 服务模式 |
| `QT_QPA_PLATFORM` | 否 | 设为 `offscreen` 可显式指定无界面模式；未设置时程序也默认使用该模式 |
| `DOCCRAFT_COLLECTION_ROOT` | 否 | 多文档集合与快照的存储根目录，必须是绝对路径；默认位于 DocCraft 应用数据目录下的 `collections` |

需要隔离集合存储时，可将 `env` 改为：

```json
{
  "QT_QPA_PLATFORM": "offscreen",
  "DOCCRAFT_COLLECTION_ROOT": "/Users/demo/Documents/DocCraftCollections"
}
```

请把 `/Users/demo` 换成自己的实际目录。文档工具的 `path` 参数也必须使用绝对文件路径。

### 3. 使用示例

#### 3.1 直接让 Agent 完成任务

配置完成后，可以直接对支持工具调用的 Agent 说：

> 使用 DocCraft MCP 检索 `/Users/demo/report.docx` 中与「研发投入增长」有关的段落。先用 `doccraft_search_context` 查找，再用 `doccraft_get_segment` 读取完整命中，最后按标题和页码总结依据。

> 使用 DocCraft MCP 修改 `/Users/demo/notes.md`：先检索「发布日期」，确认最新段落后再用 `doccraft_write_at` 替换为「发布日期：2026-11-21」，写入后重新检索确认，并返回新的 revision。

> 用 DocCraft MCP 将 `/Users/demo/a.docx`、`/Users/demo/b.pdf` 和 `/Users/demo/c.md` 创建为一个集合，再跨文件检索「研发成本」，汇总时标明来源文件。

Agent 会根据意图选择工具；下面列出需要手动集成或排查时的协议调用形式。

#### 3.2 检索并读取完整内容

先检索：

```json
{
  "jsonrpc": "2.0",
  "id": 12,
  "method": "tools/call",
  "params": {
    "name": "doccraft_search_context",
    "arguments": {
      "path": "/Users/demo/report.docx",
      "query": "研发投入为什么增长",
      "limit": 3
    }
  }
}
```

命中结果包含 `segmentId`、`page`、`heading`、`score` 和 `excerpt`。摘要最多 200 字符、300 tokens；需要完整原文时，将命中的 `segmentId` 作为下一调用的 `segment_id`：

```json
{
  "jsonrpc": "2.0",
  "id": 13,
  "method": "tools/call",
  "params": {
    "name": "doccraft_get_segment",
    "arguments": {
      "path": "/Users/demo/report.docx",
      "segment_id": "s2"
    }
  }
}
```

PDF/OFD 也可以用 1 起始的 `page` 代替 `segment_id`。需要直接生成模型上下文时，使用：

```json
{
  "jsonrpc": "2.0",
  "id": 15,
  "method": "tools/call",
  "params": {
    "name": "doccraft_build_context",
    "arguments": {
      "path": "/Users/demo/report.docx",
      "intent": "分析研发支出增长原因",
      "snippet": "研发投入和人员成本",
      "limit": 3
    }
  }
}
```

返回的 `result.structuredContent.context` 可直接作为 Agent 的文档上下文。`limit` 默认是 3，可设为 1–20。

#### 3.3 定位修改与全文重写

Markdown、TXT、HTML、DOCX 按最新检索得到的 segment 写入：

```json
{
  "jsonrpc": "2.0",
  "id": 20,
  "method": "tools/call",
  "params": {
    "name": "doccraft_write_at",
    "arguments": {
      "path": "/Users/demo/notes.md",
      "segment_id": "s2",
      "content": "由 Agent 替换后的段落。"
    }
  }
}
```

XLSX/CSV 使用 `cell`，例如 `"cell":"B2"`；PDF/OFD 使用 `page`，并可选传 `bounds:[x,y,width,height]`（单位 pt，左上角原点）。例如：

```json
{
  "jsonrpc": "2.0",
  "id": 24,
  "method": "tools/call",
  "params": {
    "name": "doccraft_write_at",
    "arguments": {
      "path": "/Users/demo/report.pdf",
      "page": 2,
      "bounds": [72, 96, 360, 48],
      "content": "Agent 页面覆盖文字"
    }
  }
}
```

需要防止覆盖他人的修改时，先计算当前文件 SHA-256，并在写入参数中传入 `expected_sha256`。成功结果中的 `revision` 是写后哈希，可作为下一次写入的 `expected_sha256`；`previousRevision`、`changeId` 和备份信息可用于追踪变更。

`doccraft_rewrite_document` 用于全文重写，支持 TXT、MD、HTML、DOCX、XLSX、CSV、PDF、OFD：

```json
{
  "jsonrpc": "2.0",
  "id": 30,
  "method": "tools/call",
  "params": {
    "name": "doccraft_rewrite_document",
    "arguments": {
      "path": "/Users/demo/notes.md",
      "content": "# 新标题\n\n由 Agent 全文重写。"
    }
  }
}
```

写入后可调用 `doccraft_get_changelog` 查看变更日志。每次成功写入前都会保留原文件备份，并强制刷新对应检索索引。

#### 3.4 多文档集合与跨文件检索

创建集合时传入显式文件列表，不会自动展开目录或 glob：

```json
{
  "jsonrpc": "2.0",
  "id": 50,
  "method": "tools/call",
  "params": {
    "name": "doccraft_create_collection",
    "arguments": {
      "name": "项目资料",
      "paths": [
        "/Users/demo/a.docx",
        "/Users/demo/b.pdf",
        "/Users/demo/c.md"
      ]
    }
  }
}
```

使用返回的 `collectionId` 检索当前快照：

```json
{
  "jsonrpc": "2.0",
  "id": 51,
  "method": "tools/call",
  "params": {
    "name": "doccraft_search_collection",
    "arguments": {
      "collectionId": "collection-id",
      "query": "研发投入",
      "limit": 3
    }
  }
}
```

`doccraft_build_collection_context` 可按 `tokenBudget` 拼接跨文件上下文；`doccraft_get_collection_segment` 可回读快照中的完整原文。源文件变化不会自动修改旧快照，需要调用 `doccraft_start_index_job` 重建，再用 `doccraft_get_index_job` 查询进度；`doccraft_cancel_index_job` 可取消任务，`doccraft_get_collection_changelog` 可比较两个快照。

### 4. 工具一览

| 场景 | 工具 |
| --- | --- |
| 文档信息 | `doccraft_get_metadata`、`doccraft_list_outline` |
| 单文件检索与上下文 | `doccraft_search_context`、`doccraft_get_segment`、`doccraft_build_context` |
| 单文件写入与记录 | `doccraft_write_at`、`doccraft_rewrite_document`、`doccraft_get_changelog` |
| 集合与快照 | `doccraft_create_collection`、`doccraft_list_collection`、`doccraft_search_collection`、`doccraft_build_collection_context`、`doccraft_get_collection_segment`、`doccraft_get_collection_changelog` |
| 异步索引 | `doccraft_start_index_job`、`doccraft_get_index_job`、`doccraft_cancel_index_job` |

工具执行结果位于 `result.structuredContent`，`result.content` 同时提供文本形式。工具失败通常返回 `result.isError=true`；协议错误使用 JSON-RPC 的 `error` 字段。

### 5. 手动 stdio 调试

一般不需要手动发送协议。若要验证安装，可向服务逐行发送 NDJSON：

```bash
printf '%s\n' \
'{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-03-26","capabilities":{},"clientInfo":{"name":"terminal-demo","version":"1.0"}}}' \
'{"jsonrpc":"2.0","method":"notifications/initialized"}' \
'{"jsonrpc":"2.0","id":2,"method":"tools/list","params":{}}' \
| /Applications/DocCraft.app/Contents/MacOS/doccraft-mcp
```

传输约定为 UTF-8、JSON-RPC 2.0、一行一个完整 JSON 对象；不要使用 `Content-Length` 帧头。stdout 只承载协议响应，stderr 是诊断日志。支持 MCP 协议版本 `2024-11-05`、`2025-03-26` 和 `2025-06-18`。

### 6. 写入安全与格式限制

- MCP 以**磁盘文件**为中心。GUI 中尚未保存的修改不可见；写入前应先在 GUI 保存并关闭该文档，完成后重新打开，避免旧编辑缓冲覆盖 Agent 的修改。
- 写入会改变 segment 内容和编号。每次定位写入前应重新检索，不要长期使用旧的 `segmentId`。
- PDF/OFD 定位写入是非破坏式页面覆盖层，不删除底层原始文字，也不触发正文重排；全文重写会从纯文本重新生成固定版式文档，不保留原页面布局。
- XLSX 单元格写入当前针对第一个工作表。DOCX 被替换段落的行内样式可能简化，HTML 序列化后标签格式可能被规范化。
- 集合用于批量读取和索引，不提供批量写入；单文件修改仍使用上述写入工具。
- 当前服务没有路径白名单和调用方授权模型。只应向可信客户端开放 DocCraft MCP，并限制 Agent 可访问的目录及可写入的文件。

### 7. 常见问题

- **提示找不到可执行文件**：确认已把当前安装包解压出的 `DocCraft.app` 放入「应用程序」，并检查 `command` 是否与实际安装位置一致。
- **提示缺少 `offscreen` 平台插件**：说明使用了旧包或不完整的应用包，请重新下载本页当前 zip 并完整替换旧应用，不要只复制其中一个可执行文件。
- **客户端看不到工具**：检查 JSON 是否合法、是否放在 `mcpServers.doccraft` 下，然后完全重启客户端；也可运行 `--list-tools` 验证服务本身。
- **检索不到刚修改的内容**：GUI 文件需先保存；集合文件变化后需重新建立并发布新快照。
- **路径解析失败**：所有文档路径和 `DOCCRAFT_COLLECTION_ROOT` 都必须使用绝对路径，集合的 `paths` 必须是显式文件数组。

## 版本更新时间线

| 版本 | 发布日期 | 主要变化 | 安装包 |
| --- | --- | --- | --- |
| 1.0.30 | 2026-11-28 | Quick Look 覆盖 421 种内容类型，可替代 Syntax 的常见源码预览场景；任意文件进入“打开方式”候选，点文件可见，保存支持无后缀与已有后缀；PDF 两阶段加载、退出卡死与拖放 Markdown 崩溃修复 | [Release](https://github.com/zhyr/DocCraft-releases/releases/tag/v1.0.30) |
| 1.0.29 | 2026-11-26 | 支持直接打开任意文本与代码文件并识别点文件；页面增删改整合进侧栏「页面」标签 | — |
| 1.0.28 | 2026-11-21 | 优化页面编辑与划线性能，移除重复图章入口，内置 MCP 文档检索与写入服务 | 通用包 |
| 1.0.27 | 2026-11-21 | 阅粒读伴：多行输入与回车发送、生成中提示与停止、重新生成、对话气泡布局；修复清空对话时的崩溃 | [Release](https://github.com/zhyr/DocCraft-releases/releases/tag/v1.0.27) |
| 1.0.24 | 2026-10-17 | 通用安装包，支持 macOS 12 及以上的 Intel 与 Apple 芯片 | [Release](https://github.com/zhyr/DocCraft-releases/releases/tag/v1.0.24) |
| 1.0.9 | 2026-10-18 | 阅读召回按页送出命中段落 | — |
| 1.0.8 | 2026-10-17 | 阅粒读伴与安装修复 | [Release](https://github.com/zhyr/DocCraft-releases/releases/tag/v1.0.8) |
| 1.0.7 | 2026-10-17 | 体验修复 | — |
| 1.0.6 | 2026-10-17 | 体验修复 | — |
| 1.0.5 | 2026-10-15 | 文本编辑与导航 | — |
| 1.0.4 | 2026-10-14 | 应用图标可选择颜色 | [Release](https://github.com/zhyr/DocCraft-releases/releases/tag/v1.0.4) |
| 1.0.3 | 2026-10-13 | 窗口顶部与导航 | [Release](https://github.com/zhyr/DocCraft-releases/releases/tag/v1.0.3) |
| 1.0.2 | 2026-10-13 | 页眉与页脚设置、窗口行为 | [Release](https://github.com/zhyr/DocCraft-releases/releases/tag/v1.0.2) |
| 1.0.1 | 2026-10-13 | 阅读助手首个公开安装包 | [Release](https://github.com/zhyr/DocCraft-releases/releases/tag/v1.0.1) |

## 能做什么

同一窗口可以打开 PDF、OFD、Markdown、纯文本、HTML、RTF、Word、Excel、PowerPoint 和常见图片，并先给出可翻页的预览。

PDF 与 OFD 在 DocCraft 里浏览和做版式编辑：插入文本、图片、链接和形状，高亮、批注、图章和签名，水印、页码、遮盖，撤销和重做，加密、合并、拆分、压缩，导出和打印。扫描页可以做文字识别。

编辑模式可以给页面设置页眉和页脚。六个位置分别是左上、眉中、右上、左下、页脚中间、右下。每个位置可以空着、写固定文字，或嵌入更新时间、网址、页码。嵌入内容在保存和导出时按当时的时间、文件路径和页序重写。浏览模式不能设置页眉和页脚。

启动时窗口中心对齐屏幕可用区域的中心。初始大小是 1200×860，可以拖拽改变。标题栏不再重复文件名，导航栏在窗口最上方，并避开关闭、最小化和缩放按钮。「官网」在右上角。Markdown 的预览和源码切换在阅读工具栏上。

设置里可以选择蓝色或红色应用图标。重启后，启动台和程序坞上的图标换成所选颜色。图标外侧不再带灰边和半透明方框。设置页只在有更高版本号时显示更新提示。

Markdown 和纯文本在 DocCraft 里改原文。Word、PowerPoint、Excel 和 RTF 在 DocCraft 里浏览；改内容时交给本机已经安装的办公软件。

阅读助手在侧边，不把文件上传到厂商。查找只在当前正文里检索，不调用模型。总结和提问会按文档取样后，发到你在设置里选中的那一家推理服务商自己的地址。

文本按 UTF-8 读取，认不出时再试 GB18030。

## 不能做什么

Word、幻灯片和表格的内容编辑不在 DocCraft 里完成。版式编辑只作用于 PDF 和 OFD。

性能表里的秒数是开发时用来挡住明显变慢构建的回归门槛，不是在你的电脑上承诺的时限。

## 技术栈

- 语言：以 C++20 为主，少量 Objective-C++（macOS 原生窗口、启动图标与文本层）
- 框架：Qt 6.8 Widgets 负责界面与版式渲染，CMake 构建
- PDF：CoreGraphics 原生渲染；PDFium 负责增量写回，保留原始字节并追加新修订
- OFD：自研 ZIP 与 XML 读写，国密 SM3/SM2 自行实现签名，不依赖 OpenSSL
- 文档格式：Markdown、Word、Excel、PowerPoint、ODF、UOF、EPUB、MHTML、LaTeX、VSDX 等由自研解析器转换；WPS 与旧版二进制在没有原生解析时才调用本机 LibreOffice
- 打包：universal2（arm64 + x86_64），最低 macOS 12；Developer ID 签名并通过 Apple 公证、装订票据
- 测试：CTest 回归套件，当前 33 项

C++ 与桌面端的人本来就少，所以参与门槛主要不在语言，而在是否愿意读代码、按现有约定改动。

## 参与开发

源代码不在这个仓库里直接发布，但愿意参与的人可以申请获取。两条渠道：

- 社区帖留言：[DocCraft 社区帖](https://haxitag.com/community/post/doccraft-1-0-5-icloud-mv8vy5i5)
- 微信公众号：扫描下方二维码，回复「DocCraft 参与」

![微信公众号二维码](https://haxitag.com/logo/qrcode_for_gh_f9203b130c32_344.jpg)

收到申请后会确认用途，再给出代码仓库的访问方式。

## 开源许可

DocCraft 以 MIT 许可开源，全文见 [LICENSE](https://github.com/zhyr/DocCraft-releases/blob/main/LICENSE)。安装包可以免费下载和使用。
