# DocCraft

DocCraft 是面向公文、合同、档案和日常办公的本地桌面文档工作台。文件留在本机，打开、浏览、批注、转换和保存都不上传。

这里提供免费可用的安装包。源代码不在这个仓库，但欢迎真正参与本项目的人获取，申请方式见下方「参与开发」。

## 核心理念

DocCraft 用 Markdown 作为多种格式之间的中间结构，也作为结构化存储的媒介。

Word、Excel、PowerPoint、PDF、OFD、EPUB 这些格式先被读成同一份结构化中间态，再按需要写回或导出；标题层级、段落、表格、批注、页眉页脚都落在这套结构上，而不是各自为政的一堆二进制块。

这样做的目的是让同一个人、多个 agent 和多种工具能在同一份文档上协作：纯文本人能直接读懂，agent 能稳定定位和改写，格式转换也不必为每种格式写死一套补丁。办公场景里反复出现的「换个格式再改一遍」，本质是数据没有统一的落脚点，DocCraft 想把这个落脚点收敛到 Markdown。

## 下载

当前版本：**1.0.28**（2026-11-21）

系统：macOS 12 及以上，Apple 芯片（arm64）与 Intel（x86_64）通用

安装包在 [Release 1.0.28](https://github.com/zhyr/DocCraft-releases/releases/tag/v1.0.28)：

- 文件：`DocCraft-macos-universal.zip`
- SHA-256：`0068f006a8d9cb062131e072e90561af4638cb3d91667f567fcac7ca7aa051ef`

请先删除「应用程序」里的旧 DocCraft.app，再用访达双击这个 zip 解压，把解压出的 `DocCraft.app` 拖进「应用程序」。不要直接传送已经解压的 `.app`。

这个包已用 Developer ID 签名，并完成 Apple 公证、装订票据。

### 安装

1. 下载 zip 并解压。
2. 将 `DocCraft.app` 拖到「应用程序」。
3. 从「应用程序」打开。若系统提示该应用来自互联网，选择打开。

## 版本更新时间线

| 版本 | 发布日期 | 主要变化 | 安装包 |
| --- | --- | --- | --- |
| 1.0.28 | 2026-11-21 | Markdown 预览改为灰场浮白纸，修复窗口变宽后右侧像多出空白窗口的问题 | 通用包 |
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
