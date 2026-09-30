# trip-planner

## 📝 项目简介

本项目是一款智能旅行助手，支持根据偏好自动生成行程、地图可视化路线、实时预算计算与编辑，并可导出为 PDF 或图片。

An intelligent travel assistant that generates personalized itineraries, visualizes routes on maps, calculates real-time budgets, supports itinerary editing, and allows exporting to PDF or images.

## 🎯解决痛点
传统的旅行规划方式有几个痛点。
- 首先是**信息分散**。景点信息在旅游网站上，天气信息在天气网站上，酒店信息在预订网站上，你需要在多个网站之间切换，手动整合这些信息。
- 其次是**缺少个性化**。大部分攻略都是通用的，不考虑你的个人偏好、预算限制、出行时间等因素。
- 最后是**难以调整**。当你想修改行程时，可能需要重新规划整个行程，因为景点的顺序、时间安排、预算都是相互关联的。

本项目解决了这些痛点，你只需要告诉系统"我想去北京玩 3 天，喜欢历史文化，预算中等"，系统就能自动为你生成一个完整的行程计划，包括每天去哪些景点、在哪里吃饭、住哪个酒店、需要多少预算。而且这个计划是可以调整的，你可以删除不喜欢的景点，调整游览顺序，系统会自动更新地图和预算。

## ✨ 核心功能
本项目包含以下核心功能：
- （1）智能行程规划：用户输入目的地、日期、偏好等信息，系统自动生成包含景点、餐饮、酒店的完整行程计划。

- （2）地图可视化：在地图上标注景点位置、绘制游览路线，让行程一目了然。

- （3）预算计算：自动计算门票、酒店、餐饮、交通费用，显示预算明细。

- （4）行程编辑：支持添加、删除、调整景点，实时更新地图。

- （5）导出功能：支持导出为 PDF 或图片，方便保存和分享。
## 🛠️ 技术栈
- （1）**前端层 (Vue3+TypeScript)**：负责用户交互和数据展示，包括表单输入、结果展示、地图可视化。

- （2）**后端层 (FastAPI)**：负责 API 路由、数据验证、业务逻辑。

- （3）**智能体层 (HelloAgents)**：负责任务分解、工具调用、结果整合。包含 4 个专门的 Agent。

- （4）**外部服务层**：提供数据和能力，包括高德地图 API、Unsplash API、LLM API。

数据流转过程如下：用户在前端填写表单 → 后端验证数据 → 调用智能体系统 → 智能体依次调用景点搜索、天气查询、酒店推荐、行程规划 Agent → 每个 Agent 通过 MCP 协议调用外部 API → 整合结果返回前端 → 前端渲染展示。
## 📑系统架构
项目的结构参考如下，提供便于定位源码：
```
helloagents-trip-planner/
├── backend/                    # 后端代码
│   ├── app/
│   │   ├── agents/            # 智能体实现
│   │   ├── api/               # API路由
│   │   ├── models/            # 数据模型
│   │   ├── services/          # 服务层
│   │   └── config.py          # 配置文件
│   └── requirements.txt       # Python依赖
│
└── frontend/                   # 前端代码
    ├── src/
    │   ├── views/             # 页面组件
    │   ├── services/          # API服务
    │   ├── types/             # 类型定义
    │   └── router/            # 路由配置
    └── package.json           # npm依赖
```
## 🚀 快速开始

环境要求：
- Python 3.10 或更高版本
- Node.js 16.0 或更高版本
- npm 8.0 或更高版本
- 
**获取 API 密钥：**
你需要准备以下 API 密钥：
- LLM 的 API(OpenAI、DeepSeek 等)
- 高德地图 Web 服务 Key：访问 https://console.amap.com/ 注册并创建应用
- Unsplash Access Key：访问 https://unsplash.com/developers 注册并创建应用
**将所有 API 密钥放入.env文件。**

**启动后端**
```
# 1. 进入后端目录
cd 后端目录

# 2. 安装依赖
pip install -r requirements.txt

# 3. 配置环境变量
cp .env.example .env
# 编辑.env文件，填入你的API密钥

# 4. 启动后端服务
uvicorn app.api.main:app --reload
# 或者
python run.py
```
成功启动后，访问 http://localhost:8000/docs 可以看到 API 文档。

**启动前端**
```
# 1. 进入前端目录
cd 前端目录

# 2. 安装依赖
npm install

# 3. 启动前端服务
npm run dev
```
成功启动后，访问 http://localhost:5173 即可使用应用。

## 📄 许可证

MIT License


