# Memos 项目综合介绍

> 本文档为 ancha-meng/memos 仓库的综合性项目介绍，涵盖项目概述、技术栈、目录结构、核心功能模块、构建部署与开发指南。

---

## 1. 项目概述

### 定位

Memos 是一个开源、自托管的轻量级笔记应用，专注于短格式思维记录（short-form thinking）。日常笔记、链接、工作日志和代码片段汇入一条按时间排序的 Markdown 时间线，运行在用户自控的基础设施上，无需重量级工作区的开销。

### 核心价值

- **快速捕捉** — 用 Markdown 书写，附加媒体，无需选择标题、文件夹或模板即可保存。
- **轻量组织** — 通过时间线、搜索、标签和置顶重新查阅笔记。
- **选择性分享** — 保持笔记私有或仅发布选定的内容。
- **完全掌控** — 自托管，零遥测，MIT 开源许可。

### 许可证

[MIT License](../LICENSE) — Copyright (c) 2025 Memos。

### 关键特性一览

| 特性 | 说明 |
| --- | --- |
| Markdown 时间线 | 以 Markdown 为核心的按时间排序的笔记流 |
| 多数据库支持 | SQLite、MySQL、PostgreSQL |
| 附件管理 | 图片、视频、音频上传与存储（本地 / S3） |
| AI 集成 | OpenAI / Gemini 语音转文字与音频 LLM |
| MCP 服务 | Model Context Protocol 端点，支持 AI 代理直接操作 Memo |
| 多空间（Space） | 实例级协作边界，支持成员管理与角色控制 |
| 标签与提及 | `#tag` 层级标签、`@mention` 用户提及 |
| CEL 过滤器 | 基于 Common Expression Language 的 Memo 查询过滤 |
| SSO 集成 | OAuth2 身份提供商 |
| Webhook | Memo 生命周期事件触发的 Webhook 通知 |
| 实时更新 | SSE（Server-Sent Events）推送 |
| 多语言 | i18n 国际化支持 |
| Docker 部署 | 多架构镜像（amd64 / arm64 / arm/v7） |

---

## 2. 技术栈

### 后端（Go）

| 技术 | 版本 | 用途 |
| --- | --- | --- |
| Go | 1.27.0 | 编程语言 |
| Echo | v5 | HTTP 框架 |
| Connect RPC | connectrpc.com/connect v1.20.0 | 浏览器端 gRPC 替代方案 |
| gRPC-Gateway | v2.30.0 | gRPC 到 HTTP/JSON 网关 |
| Protocol Buffers | google.golang.org/protobuf v1.36.12 | 接口定义与序列化 |
| Cobra / Viper | v1.10.2 / v1.21.0 | CLI 框架与配置管理 |
| goldmark | v1.8.5 | Markdown 解析器 |
| CEL-Go | v0.31.0 | 表达式过滤引擎 |
| JWT | golang-jwt/jwt/v5 v5.3.1 | 认证令牌 |
| AWS SDK Go v2 | v1.43.6 | S3 对象存储 |
| MCP SDK | modelcontextprotocol/go-sdk v1.7.0 | MCP 协议服务端 |
| OpenAI Go | v3.51.0 | OpenAI API 客户端 |
| Google GenAI | v1.68.0 | Gemini API 客户端 |

### 数据库驱动

| 数据库 | 驱动 | 版本 |
| --- | --- | --- |
| SQLite | modernc.org/sqlite | v1.56.0（纯 Go 实现，无 CGO） |
| MySQL | go-sql-driver/mysql | v1.10.0 |
| PostgreSQL | lib/pq | v1.12.3 |

### 前端（React + TypeScript）

| 技术 | 版本 | 用途 |
| --- | --- | --- |
| React | 19.2 | UI 框架 |
| TypeScript | 7.0 | 类型系统 |
| Vite | 8.0 | 构建工具与开发服务器 |
| Tailwind CSS | v4 | 样式框架 |
| React Query | v5 (TanStack) | 服务端状态管理 |
| React Router | v7 | 路由 |
| Connect RPC Web | @connectrpc/connect v2.1 | API 客户端 |
| Biome | v2.4 | 代码格式化与 Lint |
| Vitest | v4.1 | 单元测试 |
| CodeMirror | v6 | Markdown 编辑器 |
| i18next | v26 | 国际化 |
| highlight.js | v11 | 代码高亮 |
| KaTeX | v0.16 | 数学公式渲染 |
| Mermaid | v11 | 图表渲染 |
| Leaflet / MapLibre | — | 地图组件 |

### Protocol Buffers 代码生成

| 工具 | 输出 |
| --- | --- |
| buf (v2) | Go (`proto/gen/`)、TypeScript (`web/src/types/proto/`)、OpenAPI (`proto/gen/openapi.yaml`) |
| protocolbuffers/go | Go protobuf 消息 |
| grpc/go | gRPC 服务端桩 |
| connectrpc/go | Connect 处理器 |
| grpc-ecosystem/gateway | gRPC-Gateway 反向代理 |
| google-gnostic-openapi | OpenAPI 规范 |
| bufbuild/es | TypeScript protobuf 消息 |

### 容器化

| 技术 | 用途 |
| --- | --- |
| Docker (Alpine 3.21) | 多阶段构建，非 root 用户运行 |
| Docker Compose | 单容器本地部署 |
| Multi-arch | amd64 / arm64 / arm/v7 |

---

## 3. 目录结构

```
memos/
├── .github/                    # GitHub 配置
│   ├── ISSUE_TEMPLATE/         #   Issue 模板（bug_report、feature_request）
│   └── FUNDING.yml             #   赞助配置
│
├── cmd/                        # 应用入口
│   └── memos/                  #   Cobra/Viper CLI 与服务器启动
│       ├── main.go             #     根命令、配置绑定、服务器启动
│       ├── log.go              #     结构化日志（slog）配置
│       ├── main_test.go        #     入口测试
│       └── log_test.go         #     日志测试
│
├── internal/                   # 应用私有包（不对外暴露）
│   ├── ai/                     #   AI 集成
│   │   ├── ai.go               #     Provider 接口（OpenAI / Gemini）
│   │   ├── audio/              #     WebM 音频处理
│   │   ├── audiollm/           #     音频 LLM（Gemini）
│   │   ├── stt/                #     语音转文字（OpenAI）
│   │   ├── resolver.go         #     Provider 解析器
│   │   └── models.go           #     AI 模型定义
│   ├── base/                   #   基础工具（UID 匹配等）
│   ├── email/                  #   邮件发送
│   ├── filter/                 #   CEL 过滤引擎（Memo 查询）
│   ├── httpgetter/             #   HTML 元数据抓取
│   ├── idp/                    #   身份提供商（OAuth2 SSO）
│   ├── markdown/               #   Markdown 解析器（goldmark 扩展）
│   │   ├── ast/                #     自定义 AST 节点
│   │   ├── extensions/         #     #tag、@mention、GFM linkify 扩展
│   │   ├── parser/             #     解析器（tag、mention、math、emoji）
│   │   ├── renderer/           #     Markdown 渲染器
│   │   └── markdown.go         #     Service 接口与实现
│   ├── motionphoto/            #   动态照片处理
│   ├── profile/                #   实例配置 Profile
│   ├── scheduler/              #   定时任务调度器
│   ├── storage/                #   存储驱动（S3）
│   ├── testutil/               #   测试工具
│   ├── util/                   #   通用工具
│   ├── version/                #   版本信息
│   └── webhook/                #   Webhook 执行与安全控制
│
├── proto/                      # Protocol Buffer 定义
│   ├── api/v1/                 #   公共 API 服务定义
│   │   ├── memo_service.proto  #     Memo CRUD、评论、关系、反应
│   │   ├── attachment_service.proto  # 附件管理
│   │   ├── auth_service.proto  #     认证（登录、令牌、SSO）
│   │   ├── user_service.proto  #     用户管理
│   │   ├── space_service.proto #     空间管理
│   │   ├── ai_service.proto    #     AI 服务
│   │   ├── instance_service.proto  # 实例配置
│   │   ├── idp_service.proto   #     身份提供商
│   │   └── common.proto        #     公共消息
│   ├── store/                  #   内部存储 proto 消息
│   ├── gen/                    #   生成代码（Go / OpenAPI）
│   │   ├── api/v1/             #     生成的 Go gRPC/Gateway 代码
│   │   ├── store/              #     生成的存储 proto 代码
│   │   └── openapi.yaml        #     OpenAPI 规范
│   ├── openapi_embed.go        #   OpenAPI YAML go:embed
│   ├── buf.yaml                #   buf 配置
│   ├── buf.gen.yaml            #   代码生成配置
│   └── buf.lock                #   依赖锁定
│
├── server/                     # HTTP 服务器
│   ├── server.go               #   Echo 服务器、路由注册、优雅关闭
│   ├── cors.go                 #   CORS 中间件
│   ├── auth/                   #   认证模块
│   │   ├── authenticator.go    #     JWT / PAT 认证
│   │   ├── token.go            #     令牌生成与验证
│   │   ├── context.go          #     上下文身份传递
│   │   └── extract.go          #     令牌提取
│   ├── router/                 #   路由处理器
│   │   ├── api/v1/             #     REST API v1 服务
│   │   │   ├── v1.go           #       Gateway 注册、Connect 处理器
│   │   │   ├── acl_config.go   #       公共端点 ACL 配置
│   │   │   ├── memo_service.go #       Memo CRUD 服务
│   │   │   ├── attachment_service.go  # 附件服务
│   │   │   ├── auth_service.go #       认证服务
│   │   │   ├── user_service.go #       用户服务
│   │   │   ├── space_service.go#       空间服务
│   │   │   ├── ai_service.go   #       AI 服务
│   │   │   ├── instance_service.go  # 实例服务
│   │   │   ├── sse_hub.go      #       SSE 推送中心
│   │   │   └── ...             #       其他服务文件
│   │   ├── mcp/                #     MCP 服务（Model Context Protocol）
│   │   │   ├── service.go      #       MCP 服务构造与路由
│   │   │   ├── catalog.go      #       工具目录（策展操作白名单）
│   │   │   ├── adapter.go      #       工具调用到 API 请求适配器
│   │   │   ├── openapi.go      #       OpenAPI 规范解析
│   │   │   ├── validation.go   #       参数验证
│   │   │   ├── origin.go       #       Origin 头安全检查
│   │   │   └── result.go       #       结果标准化
│   │   ├── fileserver/         #     原生 HTTP 文件服务（缩略图、Range）
│   │   └── frontend/           #     SPA 静态文件服务
│   ├── runner/                 #   Memo payload 重建
│   ├── notification/           #   通知系统
│   └── test/                   #   服务器测试工具
│
├── store/                      # 数据存储层
│   ├── store.go                #   Store 门面（缓存、驱动、配置）
│   ├── driver.go               #   Driver 接口
│   ├── memo.go                 #   Memo 数据访问
│   ├── attachment.go           #   附件数据访问
│   ├── user.go                 #   用户数据访问
│   ├── space.go                #   空间数据访问
│   ├── reaction.go             #   反应数据访问
│   ├── inbox.go                #   收件箱数据访问
│   ├── migration/              #   数据库迁移
│   ├── db/                     #   数据库驱动实现
│   │   ├── sqlite/             #     SQLite 驱动
│   │   ├── mysql/              #     MySQL 驱动
│   │   └── postgres/           #     PostgreSQL 驱动
│   ├── cache/                  #   缓存层
│   └── seed/                   #   Demo 种子数据
│
├── web/                        # 前端 SPA
│   ├── package.json            #   依赖与脚本
│   ├── vite.config.ts          #   Vite 配置
│   ├── biome.json              #   Biome 格式化配置
│   └── src/
│       ├── App.tsx             #     应用根组件
│       ├── main.tsx            #     入口
│       ├── connect.ts          #     Connect RPC 客户端与认证拦截器
│       ├── auth-state.ts       #     令牌存储与跨标签页同步
│       ├── components/         #     UI 组件（68+ 组件）
│       │   ├── MemoEditor/     #       Markdown 编辑器
│       │   ├── MemoView/       #       Memo 展示
│       │   ├── MemoContent/    #       Markdown 渲染
│       │   ├── AppSidebar/     #       侧边栏
│       │   ├── Settings/       #       设置面板
│       │   ├── AttachmentLibrary/  #   附件库
│       │   ├── CalendarView/   #       日历视图
│       │   └── ...
│       ├── hooks/              #     React Query hooks（30+ hooks）
│       ├── contexts/           #     React Context（客户端状态）
│       ├── pages/              #     路由页面（18 页面）
│       ├── router/             #     React Router 配置
│       ├── layouts/            #     布局组件
│       ├── themes/             #     CSS 主题（OKLch 色彩）
│       ├── locales/            #     i18n 翻译文件
│       ├── types/              #     TypeScript 类型（含生成 proto）
│       ├── lib/                #     工具库
│       └── utils/              #     通用工具
│
├── scripts/                    # 构建与部署脚本
│   ├── Dockerfile              #   多阶段 Docker 构建
│   ├── compose.yaml            #   Docker Compose 配置
│   ├── build.sh                #   本地构建脚本
│   ├── entrypoint.sh           #   Docker 入口脚本（权限降级）
│   ├── install.sh              #   安装脚本
│   └── release_smoke_test.sh   #   发布冒烟测试
│
├── docs/                       # 文档
│   ├── adr/                    #   架构决策记录
│   │   ├── 0001-tag-syntax-and-recognition.md
│   │   ├── 0002-username-format-and-references.md
│   │   └── 0003-space-uid-allocation-and-format.md
│   ├── design/                 #   设计文档
│   │   └── multi-spaces.md
│   ├── glossary.md             #   领域术语表
│   ├── configuration-provisioning.md  # 配置供应
│   └── PROJECT_OVERVIEW.md     #   本文档
│
├── .golangci.yaml              # Go Lint 配置
├── .gitignore                  # Git 忽略
├── .dockerignore               # Docker 忽略
├── AGENTS.md                   # AI 编码代理指南
├── CONTEXT.md                  # 领域语言定义
├── CODEOWNERS                  # 代码所有者
├── CHANGELOG.md                # 变更日志
├── README.md                   # 项目 README
├── LICENSE                     # MIT 许可证
├── go.mod                      # Go 模块定义
├── go.sum                      # Go 依赖校验
├── release-please-config.json  # 发布自动化配置
└── .release-please-manifest.json
```

---

## 4. 核心功能模块

### 4.1 Markdown 解析器

**路径**: `internal/markdown/`

基于 [goldmark](https://github.com/yuin/goldmark) 构建的 Markdown 解析与渲染服务，扩展了 Memos 特有的语法。

**核心服务** (`markdown.go`):
- `Service` 接口提供：`ExtractAll`（一次解析提取全部元数据）、`ExtractTags`、`ExtractProperties`、`RenderMarkdown`、`GenerateSnippet`、`ValidateContent`、`RenameTag`
- `ExtractAll` 在单次 AST 遍历中提取：标签、提及、图片目标、托管附件引用、内容属性（标题、是否有链接/代码/任务列表/未完成任务）

**自定义扩展** (`extensions/`):
| 扩展 | 说明 |
| --- | --- |
| `TagExtension` | `#tag` 标签解析，支持 `/` 分隔的层级标签（如 `#book/fiction`） |
| `MentionExtension` | `@username` 用户提及解析 |
| `NewGFMLinkify()` | GFM 风格的自动链接（URL / Email） |

**解析器** (`parser/`):
- `tag.go` / `tag_unicode_tables.go` — 标签解析器与 Unicode 码点表
- `mention.go` — 提及解析器
- `math.go` — 行内/块级数学公式（`$...$` / `$$...$$`）
- `emoji.go` — Emoji 识别
- `gfm_email.go` — GFM 邮箱自动链接

**AST 节点** (`ast/`):
- `TagNode`、`MentionNode`、`InlineMathNode`、`BlockMathNode`、`GFMEmailNode`

**渲染器** (`renderer/`):
- `MarkdownRenderer` — 将 AST 渲染回 Markdown 文本

### 4.2 AI 集成

**路径**: `internal/ai/`

支持多提供商的 AI 集成，主要服务于语音转文字和音频理解。

**Provider 类型** (`ai.go`):
| Provider | 标识 | 实现路径 |
| --- | --- | --- |
| OpenAI | `OPENAI` | `stt/openai/` |
| Gemini | `GEMINI` | `audiollm/gemini/` |

**子模块**:
| 模块 | 路径 | 功能 |
| --- | --- | --- |
| 语音转文字 (STT) | `stt/openai/` | 使用 OpenAI Whisper API 将音频转为文字 |
| 音频 LLM | `audiollm/gemini/` | 使用 Google Gemini API 进行音频理解与内容生成 |
| 音频处理 | `audio/webm.go` | WebM 格式音频解析与处理（基于 ebml-go） |
| Provider 解析 | `resolver.go` | 根据配置解析 AI Provider 实例 |

**API 服务** (`server/router/api/v1/ai_service.go`):
- 通过 `AIService` proto 定义暴露 AI 功能给前端和 API 消费者
- 支持 OpenAI 和 Gemini 两种 Provider 的动态配置

### 4.3 MCP 服务

**路径**: `server/router/mcp/`

基于 [Model Context Protocol](https://modelcontextprotocol.io/) 的服务端实现，通过 `/mcp` 端点暴露 Memo 操作工具集。

**设计原则**: 工具调用在进程内转发到现有 REST API 执行。MCP 服务不拥有自己的 Store 或服务逻辑，每个工具从生成的 OpenAPI 文档派生，工具调用被翻译为对应的 `/api/v1/...` HTTP 请求并在同一 Echo 服务器上执行。

**核心文件**:
| 文件 | 职责 |
| --- | --- |
| `service.go` | 构造 MCP 服务器、注册工具、绑定 Streamable HTTP 处理器到 `/mcp` 路由 |
| `catalog.go` | 策展操作白名单、工具命名、输入/输出 Schema 组装、方法推导注解 |
| `adapter.go` | 将工具调用翻译为 `/api/v1/...` 请求并在进程内执行 |
| `openapi.go` | 解析 OpenAPI 规范、构建操作注册表、解析 `$ref` Schema |
| `validation.go` | 验证工具调用参数 |
| `origin.go` | Origin 头检查（DNS-rebinding 防护） |
| `result.go` | API 响应标准化为对象形状的 `structuredContent` |

**暴露的工具** (20 个):
| OpenAPI 操作 | MCP 工具 |
| --- | --- |
| `MemoService_ListMemos` | `memo_list_memos` |
| `MemoService_CreateMemo` | `memo_create_memo` |
| `MemoService_GetMemo` | `memo_get_memo` |
| `MemoService_UpdateMemo` | `memo_update_memo` |
| `MemoService_DeleteMemo` | `memo_delete_memo` |
| `MemoService_ListMemoComments` | `memo_list_memo_comments` |
| `MemoService_CreateMemoComment` | `memo_create_memo_comment` |
| `MemoService_ListMemoAttachments` | `memo_list_memo_attachments` |
| `MemoService_SetMemoAttachments` | `memo_set_memo_attachments` |
| `MemoService_ListMemoReactions` | `memo_list_memo_reactions` |
| `MemoService_UpsertMemoReaction` | `memo_upsert_memo_reaction` |
| `MemoService_DeleteMemoReaction` | `memo_delete_memo_reaction` |
| `MemoService_ListMemoRelations` | `memo_list_memo_relations` |
| `MemoService_SetMemoRelations` | `memo_set_memo_relations` |
| `AttachmentService_ListAttachments` | `attachment_list_attachments` |
| `AttachmentService_CreateAttachment` | `attachment_create_attachment` |
| `AttachmentService_GetAttachment` | `attachment_get_attachment` |
| `AttachmentService_DeleteAttachment` | `attachment_delete_attachment` |
| `UserService_ListMemoViews` | `user_list_memo_views` |
| `AuthService_GetCurrentUser` | `auth_get_current_user` |

**传输与协议**:
- 端点: `POST /mcp`（同一路径也支持 `GET`/`DELETE`）
- 传输: Streamable HTTP，无状态，JSON 响应
- 协议版本: `2026-07-28` 向下兼容至 `2024-11-05`
- 认证: `Authorization: Bearer <token>` 转发到进程内 API 请求
- 请求大小限制: 256 MiB

### 4.4 附件管理

**路径**: `server/router/api/v1/attachment_service*.go`, `server/router/fileserver/`, `internal/storage/`

完整的附件生命周期管理，包括上传、存储、缩略图、元数据提取和文件服务。

**上传**:
- 分块上传（`attachment_upload.go`, `attachment_upload_state.go`）— 支持大文件分块传输
- 上传完成后自动触发缩略图生成与元数据提取

**存储**:
- 本地文件系统（默认）
- S3 兼容对象存储（`internal/storage/s3/`，基于 AWS SDK Go v2）
- 存储类型通过实例设置动态切换

**文件服务** (`server/router/fileserver/`):
- 原生 HTTP 文件服务（`http.ServeContent`）
- 支持 HTTP Range 请求（视频/音频流式播放）
- 缩略图按需生成
- 并发限制（信号量: 3 个缩略图 / 2 个图片处理）

**元数据**:
- 图片 EXIF 提取（`attachment_exif_test.go`）
- 媒体元数据（`attachment_media_metadata.go`）— 分辨率、时长、编解码器
- Motion Photo 处理（`attachment_motion.go`, `internal/motionphoto/`）

### 4.5 Memo CRUD

**路径**: `server/router/api/v1/memo_service*.go`, `store/memo.go`

Memo 是 Memos 的核心领域对象，围绕短格式 Markdown 笔记展开。

**核心操作**:
| 操作 | API 方法 | 服务文件 |
| --- | --- | --- |
| 创建 Memo | `MemoService_CreateMemo` | `memo_service.go`, `memo_create_helpers.go` |
| 获取 Memo | `MemoService_GetMemo` | `memo_service.go` |
| 列出 Memo | `MemoService_ListMemos` | `memo_service.go`, `memo_service_query.go` |
| 更新 Memo | `MemoService_UpdateMemo` | `memo_service.go`, `memo_update_helpers.go` |
| 删除 Memo | `MemoService_DeleteMemo` | `memo_service.go` |
| 列出评论 | `MemoService_ListMemoComments` | `memo_service_comments.go` |
| 创建评论 | `MemoService_CreateMemoComment` | `memo_service_comments.go` |
| 列出附件 | `MemoService_ListMemoAttachments` | `memo_attachment_service.go` |
| 设置附件 | `MemoService_SetMemoAttachments` | `memo_attachment_service.go` |
| 列出反应 | `MemoService_ListMemoReactions` | `reaction_service.go` |
| 创建/更新反应 | `MemoService_UpsertMemoReaction` | `reaction_service.go` |
| 删除反应 | `MemoService_DeleteMemoReaction` | `reaction_service.go` |
| 列出关系 | `MemoService_ListMemoRelations` | `memo_relation_service.go` |
| 设置关系 | `MemoService_SetMemoRelations` | `memo_relation_service.go` |
| 分享 Memo | `MemoService_GetSharedMemo` | `memo_share_service.go` |
| 链接元数据 | `MemoService_GetLinkMetadata` | `memo_service_link_metadata.go` |

**关联功能**:
- **评论** — Memo 上的线程化评论
- **反应** — Emoji 反应
- **关系** — Memo 间的引用/回复关系
- **附件** — 图片、视频、音频文件
- **提及** — `@username` 自动通知
- **标签** — `#tag` 层级标签
- **Webhook** — Memo 创建/更新/删除时触发
- **SSE 推送** — 实时通知连接的客户端
- **可见性** — PRIVATE / PROTECTED / PUBLIC
- **分享** — 通过分享令牌公开访问

**查询过滤** (`internal/filter/`):
- 基于 CEL (Common Expression Language) 的表达式过滤引擎
- 支持 `memo_service_filter.go` 中的查询参数到 CEL 表达式的转换
- 支持按标签、内容、可见性、时间范围、空间等维度过滤

### 4.6 认证与授权

**路径**: `server/auth/`, `server/router/api/v1/auth_service*.go`, `internal/idp/`

| 功能 | 说明 |
| --- | --- |
| JWT 访问令牌 | 短期访问令牌 |
| 刷新令牌 | 长期刷新令牌（HttpOnly Cookie） |
| 个人访问令牌 (PAT) | API 令牌，用于脚本和 MCP 客户端 |
| SSO | OAuth2 身份提供商集成（`internal/idp/oauth2/`） |
| ACL | `acl_config.go` 定义公共端点与受保护端点 |
| 实例访问模式 | PRIVATE / PUBLIC 模式切换 |

### 4.7 存储层

**路径**: `store/`

统一的数据库抽象层，支持三种数据库驱动。

**架构**:
```
store.Store (门面)
  ├── Driver 接口 (driver.go)
  │   ├── db/sqlite/   — SQLite 驱动 (modernc.org/sqlite)
  │   ├── db/mysql/    — MySQL 驱动 (go-sql-driver/mysql)
  │   └── db/postgres/ — PostgreSQL 驱动 (lib/pq)
  ├── Cache 层 (cache/)
  │   └── TTL=10min, MaxItems=1000
  └── Migration (migration/)
      └── 每种驱动独立的增量迁移 + LATEST.sql
```

**数据模型**:
Memo、User、Space、Attachment、Reaction、Inbox、MemoRelation、MemoComment、MemoShare、UserSetting、InstanceSetting、IdentityProvider、CustomIcon、SpaceIcon、MemoViewIcon 等。

### 4.8 定时调度器

**路径**: `internal/scheduler/`

提供定时任务调度能力，支持 cron 表达式和日历语义解析。

---

## 5. 构建与部署

### 5.1 Docker 部署（推荐）

**快速启动**:

```bash
docker run -d \
  --name memos \
  -p 5230:5230 \
  -v ~/.memos:/var/opt/memos \
  neosmemo/memos:stable
```

**Docker Compose** (`scripts/compose.yaml`):

```yaml
services:
  memos:
    image: neosmemo/memos:stable
    container_name: memos
    volumes:
      - ~/.memos/:/var/opt/memos
    ports:
      - 5230:5230
```

```bash
docker compose -f scripts/compose.yaml up -d
```

**Docker 镜像特性**:
- 多阶段构建：Go 1.27.0 Alpine 编译 → Alpine 3.21 运行
- 非 root 用户运行（UID/GID 10001，通过 `su-exec` 权限降级）
- 多架构支持：amd64 / arm64 / arm/v7
- 端口: 5230
- 数据卷: `/var/opt/memos`
- 环境变量: `MEMOS_PORT`、`MEMOS_DSN`、`MEMOS_DSN_FILE`、`MEMOS_UID`、`MEMOS_GID`、`TZ`

### 5.2 本地构建

**后端**:

```bash
# 使用构建脚本
sh scripts/build.sh

# 或直接 go build
go build -o ./build/memos ./cmd/memos

# 运行
./build/memos --port 8081 --driver sqlite --data ~/.memos
```

**前端生产构建** (嵌入到后端):

```bash
cd web
pnpm install
pnpm release    # 输出到 ../server/router/frontend/dist
```

**完整单体构建** (前端 + 后端):

```bash
# 1. 构建前端
cd web && pnpm install && pnpm release && cd ..

# 2. 构建后端（嵌入前端静态文件）
go build -o ./build/memos ./cmd/memos

# 3. 运行
./build/memos --port 5230 --driver sqlite --data ~/.memos
```

### 5.3 CLI 参数与环境变量

| 参数 | 环境变量 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `--port` | `MEMOS_PORT` | 8081 | 服务端口（Docker 中默认 5230） |
| `--addr` | `MEMOS_ADDR` | (空) | 监听地址 |
| `--unix-sock` | `MEMOS_UNIX_SOCK` | (空) | Unix Socket 路径（覆盖 addr/port） |
| `--data` | `MEMOS_DATA` | (空) | 数据目录 |
| `--driver` | `MEMOS_DRIVER` | sqlite | 数据库驱动（sqlite/mysql/postgres） |
| `--dsn` | `MEMOS_DSN` / `MEMOS_DSN_FILE` | (空) | 数据源名称 |
| `--instance-url` | `MEMOS_INSTANCE_URL` | (空) | 实例外部 URL |
| `--demo` | `MEMOS_DEMO` | false | Demo 模式 |
| `--log-level` | `MEMOS_LOG_LEVEL` | info | 日志级别（debug/info/warn/error） |
| `--webhook-private-network-allowlist` | — | (空) | Webhook 私有网络白名单 |

---

## 6. 开发指南

### 6.1 环境要求

| 工具 | 版本 | 用途 |
| --- | --- | --- |
| Go | 1.27.0 | 后端编译 |
| Node.js | >= 24 | 前端构建 |
| pnpm | 11.0.1 | 前端包管理 |
| buf | latest | Proto 代码生成 |
| golangci-lint | v2.13.1 | Go Lint |
| Docker | — | 容器化部署（可选） |

### 6.2 环境搭建

```bash
# 1. 克隆仓库
git clone https://github.com/ancha-meng/memos.git
cd memos

# 2. 安装 Go 依赖
go mod download

# 3. 安装前端依赖
cd web && pnpm install && cd ..

# 4. 生成 Proto 代码（如修改了 .proto 文件）
cd proto && buf generate && cd ..
```

### 6.3 前后端联调

Memos 采用前后端分离开发模式，前端开发服务器代理 API 请求到后端。

**步骤 1: 启动后端开发服务器**

```bash
go run ./cmd/memos --port 8081 --driver sqlite --data ~/.memos
```

后端在 `http://localhost:8081` 启动，提供：
- REST API: `/api/v1/*`
- Connect RPC: `/memos.api.v1.*`
- MCP 端点: `/mcp`
- 文件服务: `/file/*`
- SSE: `/api/v1/sse`
- 健康检查: `/healthz`

**步骤 2: 启动前端开发服务器**

```bash
cd web
pnpm dev
```

前端在 `http://localhost:3001` 启动，API 请求代理到 `http://localhost:8081`。

### 6.4 测试

**后端测试**:

```bash
# 全部 Go 测试
go test ./...

# Store 测试（含 TestContainers 数据库驱动测试）
go test -v ./store/...

# 服务器测试（带竞态检测）
go test -v -race ./server/...

# 内部包测试（带竞态检测）
go test -v -race ./internal/...

# 运行特定测试
go test -v -run TestFoo ./pkg/...

# MCP 服务测试
go test ./server/router/mcp/...
```

**前端测试**:

```bash
cd web
pnpm test                    # Vitest 单元测试
pnpm test:watch              # 监听模式
pnpm test:coverage           # 覆盖率报告
```

### 6.5 代码规范

#### Go 代码规范

- **错误包装**: 使用 `errors.Wrap(err, "context")`（`github.com/pkg/errors`），禁止 `fmt.Errorf`
- **服务错误**: 使用 `status.Errorf(codes.X, "message")`
- **导入分组**: 标准库 → 第三方 → `github.com/usememos/memos`
- **导出注释**: 所有导出标识符必须有文档注释，godot 强制标点
- **禁止包级可变状态**: 除非周围包已使用该模式
- **Lint**: `golangci-lint run`（配置: `.golangci.yaml`）
- **自动修复**: `golangci-lint run --fix`（含 goimports）
- **go mod**: `go mod tidy -go=1.27.0`

#### 前端代码规范

- **绝对导入**: 使用 `@/` 前缀
- **格式化**: Biome — 2 空格缩进、双引号、分号、140 字符行宽
- **服务端状态**: 放入 React Query hooks（`web/src/hooks/`）
- **客户端状态**: 放入 React Context 或组件状态
- **样式**: Tailwind CSS v4 工具类 + `cn()` 类合并 + CVA 变体
- **UI 原语**: 优先复用 Radix 原语和现有组件
- **生成的 Proto TypeScript**: `web/src/types/proto/` 下禁止手动编辑和 Biome 重写
- **Lint**: `pnpm lint`（TypeScript 类型检查 + Biome lint）
- **格式化**: `pnpm format`

#### 数据库与 Proto 规范

- Schema 变更必须同时为 SQLite、MySQL、PostgreSQL 添加迁移，并更新各自的 `LATEST.sql`
- 新安装 SQL 与增量迁移必须保持等价
- Proto 字段变更必须保持兼容性（除非任务明确允许破坏性 API 变更）
- 禁止手动编辑生成的 proto 输出 — 修改 `.proto` 文件后运行 `buf generate`
- 新增公共 API 端点需添加到 `server/router/api/v1/acl_config.go`

### 6.6 变更路由

| 变更类型 | 更新位置 | 验证命令 |
| --- | --- | --- |
| Go 服务或路由 | `server/` 下服务代码，包级测试 | `go test -v -race ./server/...` |
| Store 或迁移 | `store/`，三种 DB 驱动迁移，`LATEST.sql` | `go test -v ./store/...` |
| 内部包逻辑 | `internal/` 相关包测试 | `go test -v -race ./internal/...` |
| 前端行为 | `web/src/` 下组件/hooks/contexts | `cd web && pnpm lint && pnpm test` |
| 前端生产输出 | Vite 配置或发布敏感 UI | `cd web && pnpm build` 或 `pnpm release` |
| Proto API | `.proto` 源文件 + 生成输出 | `cd proto && buf generate && buf lint` |
| 公共未认证路由 | `server/router/api/v1/acl_config.go` | 服务器测试或手动路由检查 |

### 6.7 CI 参考

| CI 流水线 | 工具版本 | 检查内容 |
| --- | --- | --- |
| 后端 | Go 1.27.0, golangci-lint v2.13.1 | `go mod tidy -go=1.27.0`、lint、测试分组（store/server/internal/other） |
| 前端 | Node 24, pnpm 11.0.1 | `pnpm lint`、`pnpm test`、`pnpm build` |
| Proto | buf | `buf lint`、`buf format` |
| Docker | — | `scripts/Dockerfile`，Alpine 3.21，多架构 amd64/arm64/arm/v7 |

### 6.8 项目术语

项目使用严格的领域语言，详见以下文档：
- [CONTEXT.md](../CONTEXT.md) — 领域语言定义（Memo ID/UID、Space、反应等）
- [docs/glossary.md](glossary.md) — 完整领域术语表（用户、标签、提及等）
- [docs/adr/](adr/) — 架构决策记录（标签语法、用户名格式、Space UID 分配）

---

## 参考链接

- [Memos 官方文档](https://usememos.com/docs)
- [Memos 在线 Demo](https://demo.usememos.com/)
- [MCP 协议规范](https://modelcontextprotocol.io/)
- [AGENTS.md](../AGENTS.md) — AI 编码代理指南
- [MCP 服务详细文档](../server/router/mcp/README.md)
- [调度器文档](../internal/scheduler/README.md)
- [过滤引擎文档](../internal/filter/README.md)
