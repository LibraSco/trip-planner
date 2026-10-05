# LangChain 智能旅行助手

基于 **LangChain + MCP + FastAPI** 的多智能体旅行规划助手，支持真实景点搜索、天气查询、酒店推荐和行程生成。本项目由 HelloAgents `SimpleAgent` 版本迁移而来，核心架构已改为 LangChain Agent。

## ✨ 功能特点

- 🤖 **LangChain 多智能体规划**: 景点 Agent、天气 Agent、酒店 Agent、规划 Agent 四步协作生成详细行程
- 🗺️ **MCP 接入高德地图**: 通过 `langchain-mcp-adapters` 调用 `amap-mcp-server` 的 16 个工具
- 🧠 **原生工具调用**: LangChain Agent 自动选择并调用 `maps_text_search` / `maps_weather` 等工具
- 🎨 **完整前后端**: Vue3 + TypeScript + Vite 前端，FastAPI 后端
- 📱 **完整行程要素**: 每日景点、交通、住宿、三餐、天气与预算建议

## 🏗️ 技术栈

### 后端
- **智能体框架**: LangChain（`langchain.agents.create_agent`）
- **LLM**: `langchain-openai ChatOpenAI`，兼容 OpenAI / DeepSeek 等 OpenAI 协议接口
- **MCP**: `langchain-mcp-adapters` + `amap-mcp-server`
- **API**: FastAPI / uvicorn / Pydantic v2

### 前端
- **框架**: Vue 3 + TypeScript
- **构建工具**: Vite
- **UI 组件库**: Ant Design Vue
- **地图服务**: 高德地图 JavaScript API
- **HTTP 客户端**: Axios

## 🗂️ 后端架构

```
FastAPI 路由
├── trip.py  ──────>  MultiAgentTripPlanner.plan_trip()
├── map.py   ──────>  AmapService（POI / 天气 / 路线 / 地理编码）
└── poi.py   ──────>  AmapService / UnsplashService（详情 / 搜索 / 图片）

Agent 层
  MultiAgentTripPlanner
   ├── attraction_agent  景点搜索（共享高德 MCP 工具）
   ├── weather_agent     天气查询（共享高德 MCP 工具）
   ├── hotel_agent       酒店推荐（共享高德 MCP 工具）
   └── planner_agent     行程规划（无工具，只聚合上游结果）

服务层
  llm_service    ChatOpenAI 单例
  amap_service   MultiServerMCPClient ──> uvx amap-mcp-server ──> 16 个 BaseTool
  unsplash_service     景点图片
```

### 规划流程

1. `await get_amap_tools()` 按需异步加载 MCP 工具（单例缓存）
2. 用 `create_agent(model, tools, system_prompt)` 创建四个 Agent
3. 依次执行：
   - 景点 Agent：根据城市与偏好搜索真实景点
   - 天气 Agent：查询目的城市天气
   - 酒店 Agent：按住宿偏好搜索酒店
   - 规划 Agent：整合前三步结果，输出完整 `TripPlan` JSON
4. `_parse_response` 解析 JSON；失败时自动降级为 `_create_fallback_plan`

改造后的关键差异：
- `SimpleAgent` → `langchain.agents.create_agent`，返回可 `invoke / ainvoke` 的 Agent
- `HelloAgentsLLM` → `ChatOpenAI`
- `MCPTool` → `MultiServerMCPClient.get_tools()` 返回的 `BaseTool` 列表
- 工具调用格式 `[TOOL_CALL:...]` → LangChain 原生 tool calling
- `plan_trip()` 为 `async def`，全部 Agent 调用使用 `await agent.ainvoke(...)`

## 📁 项目结构

```
backend/
├── app/
│   ├── agents/
│   │   └── trip_planner_agent.py   # 多智能体规划核心
│   ├── api/
│   │   ├── main.py                 # FastAPI 入口
│   │   └── routes/
│   │       ├── trip.py             # 旅行规划接口
│   │       ├── map.py              # 高德地图服务接口
│   │       └── poi.py              # POI 与图片接口
│   ├── services/
│   │   ├── amap_service.py         # MCP 客户端 + AmapService
│   │   ├── llm_service.py          # ChatOpenAI 单例
│   │   └── unsplash_service.py     # 图片搜索
│   ├── models/
│   │   └── schemas.py              # Pydantic 模型
│   └── config.py                   # 配置加载
├── requirements.txt
├── .env.example
└── run.py
```

## 🚀 快速开始

### 后端

1. 进入后端目录并创建虚拟环境：

```bash
cd backend
python -m venv venv
venv\Scripts\activate        # Windows
```

2. 安装依赖：

```bash
pip install -r requirements.txt
```

3. 配置环境变量：

```bash
copy .env.example .env       # Windows
```

修改 `.env`，至少配置：

| 变量 | 说明 | 示例 |
| --- | --- | --- |
| `LLM_MODEL_ID` | 使用的模型名 | `Deepseek-v4-flash` / `gpt-4o` |
| `LLM_API_KEY` | LLM API Key | `sk-xxx` |
| `LLM_BASE_URL` | OpenAI 兼容服务地址，**不要以 `/chat/completions` 结尾** | `https://api.deepseek.com/v1` |
| `LLM_TIMEOUT` | 超时秒数 | `60` |
| `AMAP_API_KEY` | 高德 Web 服务 Key | `xxx` |
| `CORS_ORIGINS` | 允许的前端地址 | `http://localhost:5173` |

4. 启动后端：

```bash
uvicorn app.api.main:app --reload --host 0.0.0.0 --port 8000
```

启动后访问 `http://localhost:8000/docs` 查看接口文档；`.env` 固定读取 `backend/.env`，不依赖启动目录。

### 前端（可选）

```bash
cd frontend
npm install
npm run dev
```

访问 `http://localhost:5173`。

## 📄 API 文档

启动后访问 `http://localhost:8000/docs`（Swagger）或 `/redoc`，主要端点：

| 方法 | 路径 | 说明 |
| --- | --- | --- |
| GET | `/api/trip/health` | 旅行规划健康检查（MCP 工具数量） |
| POST | `/api/trip/plan` | 生成旅行计划 |
| GET | `/api/map/poi?keywords=故宫&city=北京` | POI 搜索 |
| GET | `/api/map/weather?city=北京` | 天气查询 |
| POST | `/api/map/route` | 路线规划 |
| GET | `/api/poi/detail/{poi_id}` | POI 详情 |
| GET | `/api/poi/search?keywords=...&city=...` | POI 搜索 |
| GET | `/api/poi/photo?name=故宫` | 景点图片 |

## 🔧 常见问题

**LLM 请求返回 404 `Not Found`**

检查 `LLM_BASE_URL` 是否误填了完整地址（如 `.../v1/chat/completions`）。`ChatOpenAI` 会自动追加 `/chat/completions`，所以只需配置到根地址，例如 `https://chatapi.weixin.qq.com/openai/v1`。

**找不到 `uvx` 或 `amap-mcp-server`？**

先确认 `python -m pip show uv` 或 `uv --version` 可用，并确保 `uvx` 在 PATH 中。

**Agent 没有真实数据、返回“北京景点1”？**

说明整个链路兜底了。先确认 LLM API Key 与模型名正确，再逐层验证 `/api/map/poi`、`/api/map/weather` 可独立调用。

## 🙏 致谢

- [LangChain](https://github.com/langchain-ai/langchain)
- [langchain-mcp-adapters](https://github.com/langchain-ai/langchain-mcp-adapters)
- [amap-mcp-server](https://github.com/sugarforever/amap-mcp-server)
- [高德地图开放平台](https://lbs.amap.com/)

---

**项目前身**: HelloAgents 智能旅行助手（基于 `SimpleAgent`），现已迁移为 LangChain 架构。
