# Augment2Api

# Using this item will quickly result in your account being banned, so it is no longer recommended.

**English** | [中文](README_zh.md)

> A middleware service for connecting to Augment API, providing OpenAI-compatible interfaces with support for Claude3.7、Claude4 model calls.

[![GitHub Stars](https://img.shields.io/github/stars/linqiu919/augment2api?style=flat-square)](https://github.com/linqiu919/augment2api/stargazers)
[![GitHub Forks](https://img.shields.io/github/forks/linqiu919/augment2api?style=flat-square)](https://github.com/linqiu919/augment2api/network/members)
[![GitHub Issues](https://img.shields.io/github/issues/linqiu919/augment2api?style=flat-square)](https://github.com/linqiu919/augment2api/issues)
[![Docker Pulls](https://img.shields.io/docker/pulls/linqiu1199/augment2api?style=flat-square)](https://hub.docker.com/r/linqiu1199/augment2api)

## 📋 Table of Contents

- [Features](#-features)
- [Important Notice](#-important-notice)
- [Supported Models](#-supported-models)
- [Environment Variables](#-environment-variables)
- [Quick Start](#-quick-start)
- [API Usage](#-api-usage)
- [Admin Interface](#-admin-interface)
- [Batch Add Tokens](#-batch-add-tokens)
- [Feedback](#-feedback)
- [Star History](#-star-history)

## ✨ Features

- 🔄 OpenAI-compatible API interface
- 🤖 Support for Claude-Sonnet-3.7 model(In Chat Mode),Cluade-4-Sonnet(In Agent Mode)
- 📡 Support for streaming/non-streaming output
- 🎛️ Simple multi-token management interface
- 🗄️ Redis-based token storage
- 🔍 Batch token and tenant URL detection and updates
- 📥 API for batch token addition

## ⚠️ Important Notice

> **Risk Warning**: Using this project may result in your account being flagged, restricted, or banned. Use at your own risk!

- The system determines the usage mode based on the input model name. In `AGENT mode`, all tool calls are blocked, using the model's native capabilities to respond, otherwise conversations may be interrupted by tool calls
- Default concurrency control is applied, limiting each token to a maximum of `1 request every 3 seconds`, with default `Block Token` cooldown rules
- Augment's `Agent` mode is very powerful. We recommend using the official plugin in your editor for an experience comparable to `Cursor`

## 🤖 Supported Models

```bash
Model names ending with -chat use CHAT mode for responses

Model names ending with -agent use AGENT mode for responses  

Other model names default to CHAT mode
```

## 🔧 Environment Variables

| Variable          | Description                    | Required | Example                                     |
|-------------------|--------------------------------|----------|---------------------------------------------|
| REDIS_CONN_STRING | Redis connection string        | ✅ Yes    | `redis://default:password@localhost:6379`  |
| ACCESS_PWD        | Admin panel access password    | ✅ Yes    | `your-access-password`                      |
| AUTH_TOKEN        | API access authentication token| ❌ No     | `your-auth-token`                          |
| ROUTE_PREFIX      | API request prefix             | ❌ No     | `your_api_prefix`                          |
| CODING_MODE       | Debug mode switch              | ❌ No     | `false`                                    |
| CODING_TOKEN      | Debug token                    | ❌ No     | `empty`                                    |
| TENANT_URL        | Debug tenant URL               | ❌ No     | `empty`                                    |
| PROXY_URL         | HTTP proxy address             | ❌ No     | `http://127.0.0.1:7890`                   |
| REMOVE_FREE       | Remove free accounts switch    | ❌ No     | `false`                                    |

> **Tip**: If the page fails to get tokens, you can set `CODING_MODE=true` and configure `CODING_TOKEN` and `TENANT_URL` to use a specific token and tenant URL (limited to single token usage).

### REMOVE_FREE Environment Variable

Since Augment doesn't automatically switch to Free plan after trial ends, causing conversations to respond with plan switching prompts, if you need to automatically remove these accounts, you can set `REMOVE_FREE=true`. The system will automatically identify and disable free accounts during batch detection.

If the response contains the following content:
- "Your subscription for account"
- "is inactive"  
- "update your plan here to continue using Augment"

The token will be automatically marked as unavailable and will not participate in subsequent API call allocations.

This helps improve API success rates and stability, avoiding request failures due to free account limitations.

## 🚀 Quick Start

### Option 1: Using Docker

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

### Option 2: Using Docker Compose

1. **Clone the project**
```bash
git clone https://github.com/linqiu1199/augment2api.git
cd augment2api
```

2. **Configure environment variables**

Create a `.env` file:

```env
# Set Redis password (required)
REDIS_PASSWORD=your-redis-password

# Set admin panel access password (required)
ACCESS_PWD=your-access-password

# Set api authentication token (optional)
AUTH_TOKEN=your-auth-token
```

3. **Start services**
```bash
docker-compose up -d
```

This will start both Redis and Augment2Api services and automatically handle network connections between them.

### Getting Tokens

1. Visit `http://ip:27080/` to access the admin login page, enter your access password to enter the admin panel
2. Click the `Add TOKEN` menu

<img width="1576" alt="Admin Interface" src="https://img.imgdd.com/d3c389de-c894-4c1a-9b2e-2bc1c28b0f03.png" />

3. Follow these steps to get tokens:
   - Click to get authorization link
   - Copy the authorization link and open it in browser
   - Login with email (domain email also works)
   - Copy the `augment code` to the authorization response input box, click get token
   - Token should appear normally in the TOKEN list

<img width="1576" alt="Get Token" src="https://img.imgdd.com/8d7949fe-e9ee-41ad-bebd-2e56e8c7737f.png" />

4. Start conversation testing

> **Tips**:
> - If conversation returns 503 error, run `batch detection` once to update tenant URLs before testing conversations (tenant URL error)
> - If conversation returns 401 error, run `batch detection` once to disable invalid tokens before testing conversations (account banned)

## 📖 API Usage

### Get Model List

```bash
curl -X GET http://localhost:27080/v1/models
```

### Chat Interface

```bash
curl -X POST http://localhost:27080/v1/chat/completions \
-H "Content-Type: application/json" \
-d '{
  "model": "claude-3.7",
  "messages": [
    {"role": "user", "content": "Hello, please introduce yourself"}
  ]
}'
```

## 🎛️ Admin Interface

Visit `http://localhost:27080/` to open the admin login page. After logging in, you can interactively get and manage tokens.

## 📥 Batch Add Tokens

### Without AUTH_TOKEN set

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

### With AUTH_TOKEN set

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

## 💬 Feedback

🐞 [Telegram Group](https://t.me/+AfGumJADbLYzYzE1)

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
  <strong>If this project helps you, please give us a ⭐ Star!</strong>
</div>
