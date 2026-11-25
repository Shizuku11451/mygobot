# mygobot

一个轻量的纯 Go 个人 AI Bot。同一份代码打包成桌面 App（macOS 优先）和手机 App（Android 优先），**每台设备都是完整的独立节点**——自带大脑、记忆、Skill、MCP、人格与渠道，不连服务器也能完整使用。

可选部署零依赖中继服务 `mygobot-relay`，实现多设备间**端到端加密（E2EE）**的数据同步、远程能力借用与渠道自动交接。中继服务器既看不到内容，也看不到文件名。

```text
  Mac 节点（完整）                       手机节点（完整）
  大脑·记忆·Skill·MCP·渠道               大脑·记忆·Skill·MCP·渠道
  shell / 文件 / stdio MCP              定位 / 通知 / 分享 / 相机
        │        端到端加密（账号密钥）          │
        └──────► mygobot-relay（可选）◄────────┘
                 仅可见：密文、哈希、设备数、在线状态、流量大小
```

## 核心特性

- **每台设备都是完整节点**：支持 OpenAI（Chat Completions 及兼容厂商）和 Claude（Anthropic Messages）双协议，涵盖流式输出、思考过程、工具调用、代理配置与熔断保护。
- **极简省上下文**：内置渐进式加载（Skill 按需读正文、MCP / 远程工具通过 `tools_load` 按需注入 schema）、大结果本地分页缓存、三级历史压缩、`small_model` 分工与 Prompt Cache 友好排序。
- **多渠道与安全确认**：支持 App 原生 UI、Telegram、微信（直连腾讯开源 iLink 协议）。`shell`、`write_file` 等危险工具每次均需在当前交互界面（App 卡片 / TG 按钮 / 微信确认码）人工确认，不做“总是允许”。
- **可选多端互联**：
  - **数据同步**：纯文件（Markdown / JSON / 64KB 分块 JSONL）原子写入与 Lamport 时钟同步，无数据库。
  - **远程工具**：手机可安全借用电脑的 `shell` / 文件 / stdio MCP，支持心跳保活与 `/stop` 杀整组子进程。
  - **自动交接**：单个 `host` 租约 + 两层 fencing（中继 CAS + 客户端 AAD），电脑常驻兜底、手机仅在无电脑在线且前台时接管。
- **一等公民端到端加密**：全部使用 Go 标准库（X25519、Ed25519、HKDF、AES-256-GCM、HMAC），支持 6 位配对码 + SAS 校验、基于 `prev` 因果链的设备事件 DAG、离线上线可收的 `keywrap`、吊销轮换与恢复码。

## 轻量目标与技术栈

整个项目仅 **3 个直接依赖**（`github.com/egoist/mygo`、`github.com/modelcontextprotocol/go-sdk`、`rsc.io/qr`），中继服务器**零第三方依赖**。

| 组件 | 依赖 | 体积目标 | 空闲内存目标 |
|---|---|---|---|
| `mygobot-relay` | 仅 Go 标准库 | < 8 MB（静态编译） | < 15 MB |
| App（桌面 / 手机） | `mygo`、`go-sdk`、`rsc.io/qr` | macOS < 25 MB | macOS < 70 MB |

- **语言与构建**：Go 1.27+，`CGO_ENABLED=0`
- **客户端界面**：MyGo 原生 UI（跨 macOS / Android，后续扩展 Windows / Linux / iOS）
- **存储**：Markdown + JSON / JSONL 文件（原子写入与崩溃容错）

## 项目进度

当前处于设计与预研阶段，完整架构、加密协议与分阶段里程碑见：

- [PLAN.md](./PLAN.md)
- [GitHub Issue #1：mygobot 计划（v7）](https://github.com/Shizuku11451/mygobot/issues/1)

### 里程碑概览

- [ ] **预研**：MyGo macOS / Android 真机验证（数据目录、中文 IME、长文本复制、切后台恢复）、微信主动推送验证
- [ ] **P0 骨架**：双协议 LLM 客户端、agent 循环、`store` 原子写与会话分块、`cmd/chat`
- [ ] **P1 大脑**：内置工具、危险工具确认、SSRF 防护、人格与记忆快照、省上下文策略
- [ ] **P2 App**：macOS / Android 原生页面、托盘、钥匙串、Android 设备工具
- [ ] **P3 渠道 + 扩展**：Telegram、微信 iLink、IM 危险工具确认、Skill、MCP、`tools_load`
- [ ] **P4 加密 + 同步**：`e2e/`、中继与配额、配对 SAS、设备事件 DAG、`keywrap`、恢复码、同步日志
- [ ] **P5 远程工具 + 交接**：远程工具心跳/取消、`host` 租约与 fencing
- [ ] **P6 吊销 + 上线**：吊销轮换与续传、部署配置、体积与内存验收
- [ ] **P7 其他平台**：Windows / Linux / iOS
