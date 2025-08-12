# Augment2Api
# 使用本项目将很快导致您的账号被封禁，因此不再建议您使用此项目，我们已有新的项目可用，需要您自己发现。

[English](README.md) | **中文**

> 一个用于连接 Augment API 的中间层服务，提供 OpenAI 兼容的接口，支持 Claude3.7、Claude4 模型的调用。

[![GitHub Stars](https://img.shields.io/github/stars/linqiu919/augment2api?style=flat-square)](https://github.com/linqiu919/augment2api/stargazers)
[![GitHub Forks](https://img.shields.io/github/forks/linqiu919/augment2api?style=flat-square)](https://github.com/linqiu919/augment2api/network/members)
[![GitHub Issues](https://img.shields.io/github/issues/linqiu919/augment2api?style=flat-square)](https://github.com/linqiu919/augment2api/issues)
[![Docker Pulls](https://img.shields.io/docker/pulls/linqiu1199/augment2api?style=flat-square)](https://hub.docker.com/r/linqiu1199/augment2api)

## 📋 目录

- [功能特点](#-功能特点)
- [使用须知](#-使用须知)
- [支持模型](#-支持模型)
- [环境变量配置](#-环境变量配置)
- [快速开始](#-快速开始)
- [API 使用](#-api-使用)
- [管理界面](#-管理界面)
- [批量添加Token](#-批量添加token)
- [问题反馈](#-问题反馈)
- [Star History](#-star-history)

## ✨ 功能特点

- 🔄 提供 OpenAI 兼容的 API 接口
- 🤖 支持 Claude-Sonnet-3.7(Chat 模式下)，Claude-4-Sonnet (Agent 模式下)
- 📡 支持流式/非流式输出 (Stream/Non-Stream)
- 🎛️ 支持简洁的多Token管理界面管理
- 🗄️ 支持 Redis 存储 Token
- 🔍 支持批量检测Token和租户地址并更新
- 📥 支持接口批量添加Token

## ⚠️ 使用须知

> **风险提示**：使用本项目可能导致您的账号被标记、风控或封禁，请自行承担风险！

- 默认根据传入模型名称确定使用使用模式，`AGENT模式`下屏蔽所有工具调用，使用模型原生能力回答，否则对话会被工具调用截断
- 默认添加并发控制，单Token`3秒`内最多请求 `1次`,默认添加`Block Token`冷却规则
- `Augment`的`Agent`模式很强，推荐你在编辑器中使用官方插件，体验不输`Cursor`

## 🤖 支持模型

```bash
传入模型名称以 -chat 结尾,使用CHAT模式回复

传入模型名称以 -agent 结尾,使用AGENT模式回复

其他模型名称默认使用CHAT模式
```

## 🔧 环境变量配置

| 环境变量              | 说明             | 是否必填 | 示例                                        |
|-------------------|----------------|------|-------------------------------------------|
| REDIS_CONN_STRING | Redis 连接字符串    | ✅ 是    | `redis://default:password@localhost:6379` |
| ACCESS_PWD        | 管理面板访问密码       | ✅ 是    | `your-access-password`                    |
| AUTH_TOKEN        | API 访问认证 Token | ❌ 否    | `your-auth-token`                         |
| ROUTE_PREFIX      | API 请求前缀       | ❌ 否    | `your_api_prefix`                         |
| CODING_MODE       | 调试模式开关         | ❌ 否    | `false`                                   |
| CODING_TOKEN      | 调试使用Token      | ❌ 否    | `空`                                       |
| TENANT_URL        | 调试使用租户地址       | ❌ 否    | `空`                                       |
| PROXY_URL         | HTTP代理地址       | ❌ 否    | `http://127.0.0.1:7890`                   |
| REMOVE_FREE       | 移除免费账户开关       | ❌ 否    | `false`                                   |

> **提示**：如果页面获取Token失败，可以配置`CODING_MODE`为true,同时配置`CODING_TOKEN`和`TENANT_URL`即可使用指定Token和租户地址，仅限单个Token

### REMOVE_FREE 环境变量说明

由于 Augment 不会在试用结束之后自动切换 Free 计划，导致对话响应提示切换计划信息，如果你需要自动移除这些账号，可以设置 `REMOVE_FREE=true`，此时系统在批量检测过程中会自动识别并禁用免费账户。

如果检测到响应包含以下内容：
- "Your subscription for account"
- "is inactive"  
- "update your plan here to continue using Augment"

则该Token会被自动标记为不可用状态，不再参与后续的API调用分配。

这有助于提高API的成功率和稳定性，避免因免费账户限制导致的请求失败。

## 🚀 快速开始

### 方式一：使用 Docker 运行

```bash
docker run -d \
  --name augment2api \
  -p 27080:27080 \
  -e REDIS_CONN_STRING="redis://default:yourpassword@your-redis-host:6379" \
  -e ACCESS_PWD="your-access-password" \
  -e AUTH_TOKEN="your-auth-token" \
  --restart always \
  linqiu1199/augment2api
```

### 方式二：使用 Docker Compose 运行

1. **克隆项目**
```bash
git clone https://github.com/linqiu1199/augment2api.git
cd augment2api
```

2. **配置环境变量**

创建 `.env` 文件：

```env
# 设置Redis密码 必填
REDIS_PASSWORD=your-redis-password

# 设置面板访问密码 必填
ACCESS_PWD=your-access-password

# 设置api鉴权token 非必填
AUTH_TOKEN=your-auth-token
```

3. **启动服务**
```bash
docker-compose up -d
```

这将同时启动 Redis 和 Augment2Api 服务，并自动处理它们之间的网络连接。

### 获取Token

1. 访问 `http://ip:27080/` 进入管理页面登录页，输入访问密码进入管理面板
2. 点击`添加TOKEN`菜单

<img width="1576" alt="管理界面" src="https://img.imgdd.com/d3c389de-c894-4c1a-9b2e-2bc1c28b0f03.png" />

3. 按照以下步骤获取Token：
   - 点击获取授权链接
   - 复制授权链接到浏览器中打开
   - 使用邮箱进行登录（域名邮箱也可）
   - 复制`augment code`到授权响应输入框中，点击获取token
   - TOKEN列表中正常出现数据

<img width="1576" alt="获取Token" src="https://img.imgdd.com/8d7949fe-e9ee-41ad-bebd-2e56e8c7737f.png" />

4. 开始对话测试

> **提示**：
> - 如果对话报错503，请执行一次`批量检测`更新租户地址再进行对话测试（租户地址错误）
> - 如果对话报错401，请执行一次`批量检测`禁用无效token再进行对话测试（账号被封禁）

## 📖 API 使用

### 获取模型列表

```bash
curl -X GET http://localhost:27080/v1/models
```

### 对话接口

```bash
curl -X POST http://localhost:27080/v1/chat/completions \
-H "Content-Type: application/json" \
-d '{
  "model": "claude-3.7",
  "messages": [
    {"role": "user", "content": "你好，请介绍一下自己"}
  ]
}'
```

## 🎛️ 管理界面

访问 `http://localhost:27080/` 可以打开管理界面登录页面，登录之后即可交互式获取、管理Token。

## 📥 批量添加Token

### 未设置 AUTH_TOKEN 时

```bash
curl -X POST http://localhost:27080/api/add/tokens \
-H "Content-Type: application/json" \
-d '[
  {
    "token": "token1",
    "tenantUrl": "https://tenant1.com"
  },
  {
    "token": "token2",
    "tenantUrl": "https://tenant2.com"
  }
]'
```

### 设置 AUTH_TOKEN 时

```bash   
curl -X POST http://localhost:27080/api/add/tokens \
-H "Content-Type: application/json" \
-H "Authorization: Bearer your-auth-token" \
-d '[
  {
    "token": "token1",
    "tenantUrl": "https://tenant1.com"
  },
  {
    "token": "token2",
    "tenantUrl": "https://tenant2.com"
  }
]'    
```

## 💬 问题反馈

🐞 [Telegram交流群](https://t.me/+AfGumJADbLYzYzE1)

## 📈 Star History

<a href="https://www.star-history.com/#linqiu1199/augment2api&Date">
 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=linqiu919/augment2api&type=Date&theme=dark" />
   <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=linqiu919/augment2api&type=Date" />
   <img alt="Star History Chart" src="https://api.star-history.com/svg?repos=linqiu919/augment2api&type=Date" />
 </picture>
</a>

---

<div align="center">
  <strong>如果这个项目对您有帮助，请给我们一个 ⭐ Star!</strong>
</div> 
