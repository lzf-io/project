# 技术文档项目

技术方案、架构设计、API 文档的统一管理仓库。

## 📁 目录结构

```
project/
├── docs/
│   ├── architecture/       # 架构设计文档
│   ├── api/               # API 文档
│   └── guides/            # 使用指南
├── mkdocs.yml             # MkDocs 配置
└── README.md              # 本文件
```

## 🚀 本地预览

```bash
# 安装 MkDocs Material
pip install mkdocs-material

# 启动本地服务器
mkdocs serve

# 访问 http://localhost:8000
```

## 📝 写作指南

- 使用 IntelliJ IDEA 编辑 Markdown
- 图表优先使用 Mermaid（代码即文档）
- 复杂图表用 Draw.io 导出 SVG 放到 `images/` 目录

## 🔧 IDEA 插件推荐

- **Mermaid**（Mermaid 图表渲染）
- **PlantUML Integration**（UML 图支持）

## 📚 文档列表

### 架构设计
- [Stripe Billing 接入方案](docs/architecture/stripe-billing.md)

### API 文档
- [REST API 规范](docs/api/rest-api.md)

### 使用指南
- [部署指南](docs/guides/deployment.md)

---

**维护者：** zhengfang  
**更新时间：** 2026-03-12
