# Helium-CTF

官网：https://aihaiqi.cn/helium-ctf/

**一款面向 CTF 选手的离线桌面工具箱**

Helium-CTF 是一个专为 CTF（Capture The Flag）网络安全比赛打造的桌面应用，集成了编码解码、Web 探测、取证分析、AI 解题、AWD 攻防等 22+ 个功能模块。它从最初的 Electron 应用重写为 **Rust + Tauri 2 + Vue 3** 架构，Windows 安装包仅 **3.35 MB**，支持完全离线使用。

> "下次打比赛断网的时候，可以试试。"

---

## 目录

- [为什么选择 Helium-CTF](#为什么选择-helium-ctf)
- [核心功能](#核心功能)
  - [工作台](#工作台)
  - [赛事管理](#赛事管理)
  - [解题工具](#解题工具)
  - [本地环境](#本地环境)
  - [AI 解题工作区](#ai-解题工作区)
  - [AWD 攻防模式](#awd-攻防模式)
  - [记录与笔记](#记录与笔记)
  - [扩展插件](#扩展插件)
  - [系统设置](#系统设置)
- [技术架构](#技术架构)
- [安全设计](#安全设计)
- [版本历史](#版本历史)
- [安装与使用](#安装与使用)
- [免责声明](#免责声明)
- [反馈](#反馈)

---

## 为什么选择 Helium-CTF

### 从 Electron 到 Tauri：一次彻底的瘦身

这个工具最初是一个 Electron 应用，主进程是 5452 行 `.cjs`，asar 一解开源码就躺在那里——对于安全从业者来说，多少有点别扭。第十届「强网杯」打完，作者决定用 Rust 重写整个主进程。

| 对比项 | Before (Electron) | After (Tauri 2 + Rust) |
|--------|-------------------|------------------------|
| 主进程 | Node.js，约 5452 行 .cjs | Rust，一万多行原生机器码 |
| 渲染引擎 | 自带 Chromium + Node | 系统 WebView（WebView2 / WKWebView / WebKitGTK） |
| 源码保护 | asar 可解包 | LTO + strip 原生编译 |
| 安装包体积 | 数百 MB | **3.35 MB** |
| 前端 | Vue 3（21 个视图未改动） | Vue 3（完全兼容） |

重写过程中，前端通过 `getBridge()` 访问 `window.ctfDesktop` 的桥接层被保留，新增了一个 `tauri-bridge.js` 按原 preload 的通道映射表重建命名空间、扁平别名、invoke 和 on，**前端业务代码一行未改**。

真正的工作量在 Rust 侧：按 21 个视图里实际调用的接口来枚举通道，没被调到的先不做。省下来的不是行数，是每一处都要想清楚的安全边界。

---

## 核心功能

Helium-CTF 拥有 **22 个主页面**，按使用场景分为七个分组。除了插件市场、赛事日历和 AI 解题，其余功能**全部在本地计算，断网照用**。

### 工作台

#### 启动器
应用的默认首页。顶部一个搜索框，支持三种输入方式：
- **工具名**：直接搜索并打开工具
- **功能关键词**：如 "base64"、"jwt"、"端口"
- **带参数的指令前缀**：输入 `b64 SGVsbG8=` 回车，结果直接出来，不用先打开工具再粘贴

下方按「指令 / 工具」分组展示已装的 76 个插件，底部显示已装数量和快捷键提示。

![启动器，工具名、关键词或指令前缀都能直接跑](https://mmbiz.qpic.cn/sz_mmbiz_png/pboI5PACA0a7axzfx0yNv5zTmkPWE0Ks6yP7Yicu6ia6Ry0PWCrU35y1rFYdHpdRvSAcvyrPhXQ01uyu3qQXbqEz2xHNKBwp5rjmI40QAjH4Y/640?wx_fmt=png)

#### AI 解题
大模型解题工作区，每道题一个标签页，独立记对话和 token 用量，多题并行互不阻塞。详见下文 [AI 解题工作区](#ai-解题工作区)。

---

### 赛事管理

#### 比赛面板
比赛当天的总览页：
- 顶部：本地战绩速览（已解 Flag / 待解 / 今日解出 / 笔记数）
- 中间：进行中的赛事和倒计时，24 小时内开始的赛事单独一块并预告开赛通知
- 右上角：一键导出 Markdown Writeup

![比赛面板，倒计时、战绩与 Writeup 导出](https://mmbiz.qpic.cn/sz_mmbiz_png/pboI5PACA0a7baRSJC6fEWNbycq1oQRzFHd9oiasSficQlicX1QQk3eJhficqMN9TkaxMv4LPw9mpBeRKpsTklR2RQySiaLct7keaiawsSAAtx7eQ/640?wx_fmt=png)

#### 赛事日历
所有赛事的日历视图和列表视图，可切月、跳今天。数据在线拉取后缓存到本地，**断网也能翻历史**，支持手动刷新和清除缓存。

![赛事日历](https://mmbiz.qpic.cn/sz_mmbiz_png/pboI5PACA0bOsO2hbMJ7yIuKZWXX5pVJSrlIOFHNoPnxlp5RicB7vtEcJWSq6QqBQsLlfGFLqL1euL6uFATkfkiaHJiazLf0zn5cXdeVocSdUQ/640?wx_fmt=png)

#### 题目下载
赛前批量拉题，统一支持三个平台：
- **i春秋**：固定支持强网杯、羊城杯、网信杯、湾区杯，登录凭据从浏览器 Console 复制一条命令即可拿到
- **CTFd**：填站点地址加 API Token
- **CyberPeace**：用平台账号密码，登录时密码按站点前端的方式 DES 加密

绑定凭据后一键拉取全部题目和附件，按「赛项 / 分类 / 题目」落目录，生成 `WriteUp.md` 模板，**已存在的文件自动跳过，不会覆盖你写了一半的 WriteUp**。凭据只存在本机数据目录。

每个账号的绑定信息、拉过的题、勾选状态各自缓存，换账号也不乱。

![题目下载页，平台的绑定配置](https://mmbiz.qpic.cn/sz_mmbiz_png/pboI5PACA0bxibHMhEaTXa33sic8o3dRU3W3ozJEtnZ4Xjeict2maTMhAAibzIibkHEH6jCJN9RfZEQxicu8tpqAj2UusHp94azYJKl2dEy45O21o/640?wx_fmt=png)

#### 开赛提醒
配合赛事日历一起用：订了比赛，**赛前 30 分钟弹系统通知**，界面里也推一条。提醒过的赛事记在本地，重启不会重复弹。

![开赛前 30 分钟的系统通知](https://mmbiz.qpic.cn/mmbiz_png/pboI5PACA0ZI4aV28M7iaFydHVMicTYq5icmWZdOAbania9rvb3j4ewJ5sZh6bRnjT5tEiaEDicXAoXCkfKQT1ssmSwqqg7ciaFCibcicyUtbPFMXpfE/640?wx_fmt=png)

---

### 解题工具

#### 编码识别
粘贴密文自动识别编码类型（Base64 / Hex / URL / ROT13 等），点结果直接解码。识别不准可手动选，支持 HTML 实体、Unicode 转义、Morse、二进制、八进制、十六进制 ASCII。解码结果可存笔记或复制。

![编码识别，自动判断类型后点一下就能解](https://mmbiz.qpic.cn/mmbiz_png/pboI5PACA0bibbenD5nEs9JjN9t6pn7DwM3l7kpQR73qX1y0JMKjiaiaMcvO5RZxTofJFmiaxVt6NeqZMS4hb3p9Br1NPGibtun8CgqlgK2pnHYI/640?wx_fmt=png)

#### Payload 速查
常用 payload 速查表，按 SQL 注入、XSS、文件包含、命令执行等分类浏览或搜索，一条点复制。支持添加自定义 payload，本地保存。

#### 反弹 Shell
选类型（bash / nc / python / perl 等）、选编码、填 IP 和端口，实时生成反弹命令和对应的监听端命令，均可一键复制或存笔记。

#### 端口扫描
TCP 连接探测常见 CTF 端口。内置常见端口预设，也支持自定义（`22,80,443` 或 `1-1000`），带进度条和可停止，扫出的开放端口可存笔记或复制。

#### JWT 工具
三个页签：
- **解析**：看 header / payload / signature、算法、过期时间
- **伪造**：改 payload 重新生成 token，支持 none 算法
- **弱密钥爆破**：字典跑 secret

---

### 本地环境

#### 文件分析
打开本地题目文件，显示大小、类型、修改时间、SHA-256，带 Hex 和文本预览，并按文件类型推荐后续分析工具。**选中的文件本会话内只读授权**，可直接交给插件，不用在插件里再选一次。

![文件分析，摘要、哈希、预览和工具推荐在同一页](https://mmbiz.qpic.cn/sz_mmbiz_png/pboI5PACA0am2XL0JSibCurDfhEdf78Ricibgq2bUbMVVAK7jKkd2RVvxuLrGicyicySRR6nE3uWGFwDFezvkDtCVjAh7s0FOoqDPnPEH1UsSmRQ/640?wx_fmt=png)

#### 工具链
把本机 14 个命令行工具接进界面：

```
file, strings, xxd, exiftool, binwalk, 7z, unzip,
pngcheck, zsteg, tshark, readelf, objdump, nm, checksec
```

三步走完：选文件（会话内授权）→ 选工具（探测到哪个亮哪个）→ 执行（追加参数走白名单）。输出带大小和超时限制。

#### 本机靶场
检测本机 Docker 里跑着的练习环境，列出运行中的容器和已发布端口，按端口一键打开 `http://127.0.0.1:端口`。只执行 `docker ps` / `docker images` 两条只读命令，**不执行 run / exec / start / stop**。

#### 接口请求
由**主进程代发 HTTP 请求**，不受浏览器 CORS 和混合内容限制，能看到真实的重定向链和 Set-Cookie。支持 curl 导入导出、请求历史复用、会话级 Cookie。默认拒绝内网地址，打靶机需在页面单独开「允许内网」开关。

![接口请求，重定向链、Cookie 与 curl 双向转换](https://mmbiz.qpic.cn/sz_mmbiz_png/pboI5PACA0aJeia9iconPGQgwYRdapjEG8RuRFHOKuUNjXicMwGBU1s0pNqHkfxiajbIJwVECkU8ibcI8KrmGDcLvG5ibFPtP0pAibFqicnqjRrib2hQ/640?wx_fmt=png)

---

### AI 解题工作区

> 这是 1.0.3 版本中**最大的新增**，不是改版，是一个全新的页面级模块。

以前解题只有工具，AI 只在网页上问。现在靶场里开一道题，直接在同一块屏幕上和模型磨。

**解题面板**显示：思考中还是已解出、多少条消息、烧了多少 token、跑了多久、上次动是什么时候。解出来的题，flag 直接挂在卡上。

**它能自己动手。** 设置里打开「AI 代码执行」和「外部工具链」，模型就能真的跑 Python，调 `file`、`strings`、`binwalk` 这些命令，而不是甩给你一段看着对的代码。它调了什么、拿到了什么，一条条以卡片列在会话里，哪一步开始跑偏，看得见。

![设置里打开「AI 代码执行」和「外部工具链」](https://mmbiz.qpic.cn/mmbiz_png/pboI5PACA0ZeFyywBecNNV7DyYtmAl38nAyR1Zia1sYOemXc25PEtUz6zoM8dxQdyWfpe08QiaOICrVTARGzyQwibF8Eiclxl6mLFeCtibkNbH7U/640?wx_fmt=png)

顶部有「⚡ 自动执行」开关，打开后 AI 自己往下跑，不用每步点确认。

**找到 flag 之后还有一道校验。** 点保存时会检查这个 flag 有没有真的出现在本机工具或脚本的输出里，没出现就提醒先核对——免得把模型编出来的一串字符当答案交上去。确认之后存进 Flag 记录，同时写一份笔记，带上题目名和来源。

**四步上手：**

1. **配置模型**：点右上角「模型配置」，填 API Base、Key，选好模型保存
2. **上传附件**：输入框旁的「📎 文件」或「📁 文件夹」，把题目传上来
3. **（可选）开本地能力**：设置 → 安全与本地能力，打开「AI 代码执行」和「外部工具链」
4. **开始解题**：描述题目，或者留空让它自己看附件，点发送

![AI 解题配置步骤](https://mmbiz.qpic.cn/sz_mmbiz_png/pboI5PACA0YaA6kkLniasVCeaC7TibXVyN767g2A9HMO6GqyQEnvGbPvLwM8xicfic8hQKo4alWRnW5wice5YX7nQp1yYukic2WHfuSbycRhPMnhY/640?wx_fmt=png)

**安全边界：**
- API Key 只存在 Rust 侧主进程，渲染层永远拿不到明文
- 自定义 Base URL 要先过 SSRF 校验，防止把 Key 发到内网
- 代码执行是**显式开关，默认关**
- 工作区路径写死只能落在本题目录
- 每次工具调用默认要用户点「允许执行」，也可以开自动执行
- `read_file` 只能读本题附件和绑定的赛题目录，`list_dir` 上限 200 项
- `run_python` 命令超过 8000 字符直接拒，单条工具输出截到 8000 字符
- Pwn / Reverse 的 Linux ELF 走 WSL，pwntools 装在专用 venv 里，不碰系统 Python

**支持的模型：** DeepSeek、Kimi、通义、智谱、OpenAI 是预设，任意兼容 OpenAI 协议的端点填地址就能用。

---

### AWD 攻防模式

> AWD 最耗人的不是打，是盯。几十个目标，每一轮都在变。谁被打了、谁的防御掉了、哪个 flag 还没交，全靠人脑记。三个人盯屏幕，两小时之后还是会漏。

新版加的 AWD 模式，干的就是这件事：接上比赛平台，让它自己跑。

填三个东西：**平台 URL、比赛平台分配的 Token、总预算**（按 tokens 算，最低 100000）。

![AWD 作战界面](https://mmbiz.qpic.cn/mmbiz_png/pboI5PACA0ZvTn44iccaHGP9e8RoxZk50FdhByn0hM1TB5Q0EthxKelvqBH7cibccmAGjsHgnK7Cdic2HfO638hrFXruAL0KCU2cgfAXDnVw6w/640?wx_fmt=png)

- **顶部状态栏**：当前状态、得分、排名、预算消耗百分比、Flag 找到/提交数、作战单元活跃数。暂停、恢复、停止都在这条上
- **左侧**：指挥所和作战单元，每个单元有自己的角色和任务，可手动指定目标，也可自动跑
- **右侧**：得分榜和攻击日志，一行行往下滚
- **情报库**：一个单元摸到的靶场地址、可用漏洞、拿到的凭据，别的单元也能用，不用每队重新摸一遍

**预算是硬约束。** 消耗到阈值会自动收手，不会把一个月的额度在一场比赛里烧完。

---

### 记录与笔记

#### Flag 记录
按赛事记录每题的 flag 和解题状态，顶部统计总题数 / 已解 / 未解。支持按赛事和分类筛选，添加、编辑、删除记录。

#### 笔记
本地解题笔记，左侧列表右侧编辑，支持 Markdown 纯文本，Ctrl+S 保存，存到数据目录的 `.storage.json`。可记思路、flag 格式、payload 备忘。最后一键导出 Markdown 格式的 Writeup。

---

### 扩展插件

#### 工具库
内置 **76 个插件** 的全量浏览和安装，覆盖编码转换、古典与现代密码、Web payload、取证、杂项、工作台，以及 Web、Pwn、Reverse、Crypto、Misc 五种题型。

双源设计：
- **内置目录**：永远在本地，离线可用
- **远端市场**：补充新插件和版本更新，断网时自动降级成只有内置，并在页面上写清楚原因

支持搜索、按分类筛选、一键安装全部、检查更新。

#### 已装工具
管理已安装的插件：启用 / 停用 / 卸载，查看每个插件的存储占用和数据目录，可单开插件窗口或打开插件目录。

---

### 系统设置

设置页把所有开关按分组排开：主题、离线模式、自启动、工具链、靶场、AI 代码执行、遥测（**默认全关**，需用户手动开启）。顶部有分组快速定位条。

#### 应用内反馈
不用再去 GitHub 开 issue，也不用找邮箱。关于页上有个「提交反馈」按钮，点开就是反馈窗口。选类型：BUG 反馈、功能建议、其它。填一段描述，联系方式选填。登录之后联系方式自动隐藏。

#### 其他
- **帮助页**：页面导览、全局与页面级快捷键表、常见问题，每个条目能直接跳到对应页面
- **关于页**：版本信息、数据目录、内置工具数量，打包版只显示用户需要的信息
- **运行自检**：设置页的「运行自检」跑一遍能看到各模块状态

---

## 技术架构

### 整体结构

```
┌─────────────────────────────────────┐
│           Vue 3 前端 (21 视图)        │
│    ┌─────────┐    ┌──────────────┐  │
│    │ 主窗口   │    │ 插件窗口      │  │
│    │ (CSP    │    │ (ctfplugin://)│  │
│    │  script-│    │              │  │
│    │  src    │    │              │  │
│    │  'self')│    │              │  │
│    └────┬────┘    └──────┬───────┘  │
│         │                │          │
│    tauri-bridge.js (按原 preload 重建)│
│         │                │          │
├─────────┴────────────────┴──────────┤
│         Rust 主进程 (Tauri 2)       │
│  ┌─────┐ ┌─────┐ ┌─────┐ ┌──────┐ │
│  │ IPC │ │安全 │ │AI   │ │更新  │ │
│  │ 入口 │ │模块 │ │工作区│ │模块  │ │
│  │ (单  │ │(工具 │ │(多层 │ │(签名│ │
│  │ 入口)│ │链/靶 │ │沙箱)│ │ 校验)│ │
│  │      │ │场/请 │ │      │ │      │ │
│  │      │ │求器) │ │      │ │      │ │
│  └─────┘ └─────┘ └─────┘ └──────┘ │
│         10993 行 Rust 代码          │
└─────────────────────────────────────┘
```

### IPC 设计：所有能力，只留一个入口

Rust 侧只有**四个 Tauri command**。主界面能用的所有通道，都汇总到一个 `ipc` 命令里，按 `app:info`、`toolchain:run` 这样的通道名分发。

返回值统一成两种形状：
- 成功：`{ ok: true, data }`
- 失败：`{ ok: false, code, error }`

前端不用为每个接口写一套错误处理。

**入口的第一件事不是分发，是认人：**

```rust
if window.label() != "main" {
    return json!({ "ok": false, "code": "E_PERM",
                   "error": "插件窗口无权调用主进程接口" });
}
```

插件跑在独立窗口里，它拿不到这组通用通道，只能用白名单里的能力。

### 插件系统安全

插件页面不是用 `file://` 加载的。Tauri 注册了一个自定义协议 `ctfplugin://`，资源根目录固定在数据目录下的 `plugins/<slug>`。

```rust
let base = paths::normalize_path(&plugins::local_dir(slug));
let target = paths::normalize_path(&base.join(rel));
if !target.starts_with(&base) {
    return None; // 路径穿越 → 404
}
```

先规范化，再比较前缀。插件目录里凡是能跳出根目录的路径，一律 404。

身份也不靠插件自报。窗口 label 是 `plugin-<slug>`，主进程从 label 反推 slug。权限按插件自己 `plugin.json` 里声明的 permissions 分发，没声明的直接 `E_PERM`。

| 能力 | 主窗口 | 插件窗口 |
|------|--------|----------|
| 通用通道 | 全部可用 | 拒绝 E_PERM |
| 插件能力 | 不适用 | 按 permissions 白名单 |
| 文件读取 | 用户选中即会话级只读 | fs 权限 + 用户选文件 |
| 本机命令 | 工具链四道闸 | shell 权限 + 同一套闸 |

主窗口的 CSP 是 `script-src 'self'`，不允许内联脚本。插件协议单独放宽了 inline（因为内置插件都是单个 HTML 文件），但这条放宽**只作用于插件协议**，主窗口不受影响。

> **跨进程边界只留一个入口，是后面所有权限判断的前提。入口多了，就一定有一个忘了判断。**

---

## 安全设计

### 工具链：四道闸

能跑本机命令的工具链是天生危险的能力，设置了四道防线：

1. **开关默认关闭**：settings 里的开关不打开，通道直接返回禁用状态，一条命令都不执行
2. **命令白名单**：只认 14 条（file, strings, xxd, exiftool, binwalk, 7z, unzip, pngcheck, zsteg, tshark, readelf, objdump, nm, checksec），bin 名写死在代码里
3. **参数白名单**：追加参数只接受 `-flag` 形式或纯值，分号、管道、与号、变量符、反引号、重定向、通配符、感叹号、引号和空格一律拒绝
4. **参数数组直连**：用参数数组启动进程，不经 shell。文件必须是用户这次会话里明确选过的，或位于下载、桌面、文档目录内

再加三条限制：单路输出最多 256 KB，超时最多 120 秒，每次执行写一条审计日志。

Windows 上还有个细节：GUI 程序 spawn 控制台程序会连弹十几个黑框，加 `CREATE_NO_WINDOW` 标志才安静下来。

### 本机靶场：比工具链还窄

只执行两条固定命令 `docker ps` 和 `docker images`，参数写死，不接受任何用户输入。`docker run / exec / start / stop` 一个都不做。

「一键开题」也不接受任意 URL，只接受容器 ID 和端口。主进程先核对容器在运行、端口确实是它声明的映射，然后拼出 `http://127.0.0.1:端口`。

### 请求器：放在主进程发

在渲染层发请求会被 CORS 拦，也拿不到真实重定向链。放到 Rust 侧发，这些限制都不在了，所以收口更要紧：

- 方法走白名单
- 请求头拒绝 CRLF 注入，host 和 content-length 等受保护头会被剔除并告知用户
- 跨域重定向自动摘除 authorization、cookie、proxy-authorization
- Cookie 只存在进程内存里，按域和路径匹配，Secure 只发给 https，**不落盘**
- 地址默认拒绝回环、内网、链路本地和云元数据地址
- 打靶机要连 127.0.0.1，需用户自己在页面开「允许内网」，且只影响这一页
- 响应体文本超过 4 MB、二进制超过 8 MB 就截断并标记，不假装完整

### AI 解题：能动手但跑不出笼子

- API Key 只存主进程内存和数据目录，渲染层永远拿不到明文
- 代码执行全局开关默认关
- 工作区路径写死只能落在本题目录
- 每次工具调用默认要用户点「允许执行」
- `read_file` 只能读本题附件和绑定的赛题目录
- `list_dir` 上限 200 项
- `run_python` 命令超 8000 字符直接拒
- 单条工具输出截到 8000 字符
- WSL 是单独的口子，pwntools 装在专用 venv 里，不碰系统 Python

### 内置 Payload 加密存储

上一版把 payload 和反弹 shell 明文写在代码里，Windows Defender 会一直盯着这个 exe。这一版把数据抽出来，XOR + base64 存成单独的 json，运行时再解回字符串。**功能一模一样，只是把长相收拾干净了。**

---

## 版本历史

### v1.0.3 (2026-11-06)

当前版本。从 1.0.2 到现在攒了一个大版本，新增三个页面级模块：

- **AI 解题工作区**：全新的页面，不是改版。模型配置、代码执行、工具调用逐次确认、flag 校验、自动生成 Writeup
- **AWD 攻防模式**：指挥所 + 作战单元 + 情报库，预算硬约束，自动攻防
- **应用内反馈**：关于页直接提交 BUG / 功能建议，不用去 GitHub 开 issue
- **界面重排**：导航重新分组，工具栏拆两层，卡片填满剩余高度
- **安全加固**：内置 payload 改为 XOR + base64 加密存储，消除 Windows Defender 误杀
- **F5 接管**：桌面端按 F5 不再触发 WebView2 整页刷新，AI 会话、当前页面、未保存输入不再丢失
- **删除确认统一**：所有删除操作统一到一个确认弹窗，危险动作红色按钮
- **拖拽上传改进**：文件和文件夹一起排队，文件夹递归，结构不丢
- **更新提示**：更新提示条会带上这一版改了什么

SHA-256：`17934D6D8FA996323A4D6522E8859EDC0AA78FDFC5C090126245EB7A3C101983`

### v1.0.2 (2026-10)

从 Electron 到 Tauri 的彻底重写：

- 主进程从 Node.js (5452 行 .cjs) 重写为 Rust (一万多行)
- 安装包从数百 MB 缩至 3.2 MB
- 前端 Vue 3 的 21 个视图一行未改，通过 `tauri-bridge.js` 桥接
- 76 个内置插件全部兼容
- 单入口 IPC 设计，插件窗口权限隔离
- 工具链四道闸、本机靶场只读、请求器主进程代发
- 完整的 minisign 签名校验更新链路

---

## 安装与使用

### 系统要求

- **Windows 10/11**（x64）
- WebView2 运行时（Windows 11 已内置，Windows 10 一般也已预装）
- 可选：Docker Desktop（本机靶场功能）
- 可选：WSL + pwntools venv（AI 解题的 Pwn/Reverse 功能）

### 安装

1. 下载仓库根目录下的 [Helium-CTF_1.0.3_x64-setup.exe](Helium-CTF_1.0.3_x64-setup.exe)（3.35 MB），或从[官网](https://aihaiqi.cn/helium-ctf/)获取
2. （可选）校验 SHA-256：
   ```powershell
   certutil -hashfile Helium-CTF_1.0.3_x64-setup.exe SHA256
   ```
3. 双击安装，覆盖安装不会清除已有配置

### 应用内更新

设置 → 启动与更新 → 检查更新 → 下载更新 → 重启安装。更新包是签名校验过的，校验不过不会装。

### 升级到 1.0.3

**升级不会清你的配置。** 主机、工具预设、AI 会话、拉过的赛题、Flag 记录都在本地数据目录，覆盖安装只换程序本体。

---

## 免责声明

Helium-CTF 是一个 **CTF 比赛辅助工具**，仅供安全研究和教育用途。

- 请勿用于任何未经授权的系统或网络
- 使用本工具进行的一切操作，后果由使用者自行承担
- 内置的 payload 和工具仅用于合法的 CTF 竞赛环境

---

## 反馈

- **应用内反馈**：关于页 → 提交反馈（推荐，可自动附加运行环境信息）
- **官网**：[aihaiqi.cn/helium-ctf](https://aihaiqi.cn/helium-ctf/)

---

## 许可协议

[MIT License](LICENSE)

---

<p align="center">
  <strong>Helium-CTF</strong> · Rust + Tauri 2 + Vue 3<br>
  少给一次 shell，比多加十行校验管用。<br><br>
  作者参赛经历：第二届「湾区杯」· 第六届「长城杯」· 2026「羊城杯」· 第十届「强网杯」<br>
  赛场上缺什么，就补什么。
</p>
