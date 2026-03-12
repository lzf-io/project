# 产品创建与AI生成管理系统 - 技术文档

**目标用户：** 独立站商户（Dropshipping 电商卖家）

## 📚 文档概览

本文档详细描述了为独立站商户提供的产品创建、AI 图片生成及 Dropshipping 产品推荐平台的技术设计。

### 核心业务

1. **AI 图片生成**（对接第三方 AI API）
   - 文生图、图生图、AI 增强
   - 积分制消费模式

2. **DS 产品推荐**（Dropshipping）
   - 推荐热门代发货产品
   - 套餐限制推荐数量
   - 利润分析与竞品分析

3. **商业化设计**
   - 积分购买与消费
   - 套餐分级与升级引导
   - 锁定/扣减/解冻机制

## 🚀 快速开始

### 在线查看

**文档站：** https://lzf-io.github.io/project/product-creation-system/

### 本地预览

```bash
# 安装依赖
pip install mkdocs-material

# 启动本地服务器
cd product-creation-system
mkdocs serve

# 访问 http://localhost:8000
```

### 在 IDEA 中编辑

1. 用 IntelliJ IDEA 打开 `product-creation-system` 目录
2. 安装 **Mermaid** 插件（Settings → Plugins）
3. 编辑 Markdown 文件，右侧实时预览（`Cmd/Ctrl + Shift + P`）
4. Mermaid 图表自动渲染

## 📁 文档结构

```
docs/
├── index.md                      # 项目概述、业务模式、核心功能
├── architecture/
│   ├── overview.md              # 系统架构、技术栈、服务设计
│   └── workflows.md             # 核心流程（含时序图）
│       ├── 产品创建流程
│       ├── AI 生成流程（文生图、图生图、失败处理）
│       ├── 积分管理流程（锁定、扣减、解冻）
│       ├── 限制与升级引导
│       ├── DS 推荐流程（配额检查、添加产品、配额重置）
│       └── 批量生成流程
├── api/
│   └── rest-api.md              # REST API 完整文档
│       ├── 产品管理 API
│       ├── AI 生成 API
│       ├── 积分管理 API
│       ├── DS 推荐 API
│       └── 套餐管理 API
└── guides/
    └── deployment.md            # 部署指南
```

## 🎯 关键设计亮点

### 1. 积分锁定机制
```
发起生成 → 锁定积分 → 生成成功 → 扣减 + 解锁
                  ↘ 生成失败 → 解冻（全额退回）
```

**解决问题：**
- 防止生成过程中积分不足
- 生成失败不扣费
- 并发安全（数据库事务 + 行锁）

### 2. DS 推荐套餐限制
```
免费版：10个/月
基础版：100个/月（¥99）
专业版：500个/月（¥299）
企业版：无限制（¥999）
```

**商业逻辑：**
- 套餐限制 DS 推荐数量
- AI 生成仍需消费积分（独立计费）
- 超限引导升级

### 3. 第三方 AI 集成
```
商户请求 → 队列 → 第三方 AI API → 回调 → 通知商户
```

**关键设计：**
- 异步处理（RabbitMQ）
- WebSocket 实时推送进度
- 失败重试 + 超时处理

## 📊 核心数据模型

### 商户 (MERCHANT)
- 关联套餐 (SUBSCRIPTION)
- 关联积分账户 (CREDIT_ACCOUNT)

### 套餐 (SUBSCRIPTION)
- level: free / basic / pro / enterprise
- ds_quota_monthly: 每月推荐配额
- ds_used_current_month: 当月已使用

### 积分账户 (CREDIT_ACCOUNT)
- balance: 可用余额
- locked: 锁定金额

### 生成任务 (GENERATION_TASK)
- mode: text_to_image / image_to_image / enhance
- status: pending / processing / completed / failed
- credits_cost: 消耗积分

### DS 推荐 (DS_RECOMMENDATION)
- supplier: aliexpress / cj_dropshipping
- profit_margin: 利润空间

## 🔧 技术栈

| 分类 | 技术 |
|------|------|
| 后端语言 | Java 17 / Python 3.10 |
| 框架 | Spring Boot 3.x |
| 数据库 | MySQL 8.0 + Redis 7.x |
| 消息队列 | RabbitMQ |
| 前端 | React 18 + TypeScript |
| UI 库 | Ant Design Pro |
| AI 服务 | Midjourney API / DALL-E 3 |
| DS 供应商 | AliExpress API / CJ Dropshipping |
| 支付 | Stripe / PayPal |

## 📝 更新日志

### 2026-03-12
- ✅ 初始版本创建
- ✅ 完整的业务场景分析
- ✅ 系统架构设计
- ✅ 核心流程时序图（8个主要流程）
- ✅ REST API 完整文档（6大模块）
- ✅ 数据模型 ER 图

## 🤝 贡献指南

### 文档更新流程

1. 编辑 `docs/` 下的 Markdown 文件
2. 本地预览：`mkdocs serve`
3. 提交更改：
   ```bash
   git add .
   git commit -m "更新：xxx"
   git push
   ```
4. GitHub Actions 自动部署

### Mermaid 图表编辑

**在线编辑器：** https://mermaid.live/

**快速参考：**
- 流程图：`graph TB` / `graph LR`
- 时序图：`sequenceDiagram`
- ER 图：`erDiagram`
- 状态图：`stateDiagram-v2`

## 📧 联系方式

- **项目负责人：** zhengfang
- **最后更新：** 2026-03-12

---

**文档站：** https://lzf-io.github.io/project/product-creation-system/
