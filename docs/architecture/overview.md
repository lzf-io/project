# 架构概览

## 系统整体架构

```mermaid
graph TB
    subgraph 客户端层
        A[Web 前端]
        B[移动端 App]
        C[第三方系统]
    end
    
    subgraph 接入层
        D[API Gateway]
        E[负载均衡]
    end
    
    subgraph 应用层
        F[用户服务]
        G[订阅服务]
        H[支付服务]
        I[通知服务]
    end
    
    subgraph 数据层
        J[(主数据库)]
        K[(缓存 Redis)]
        L[(消息队列)]
    end
    
    subgraph 外部服务
        M[Stripe]
        N[邮件服务]
        O[短信服务]
    end
    
    A --> D
    B --> D
    C --> D
    D --> E
    E --> F
    E --> G
    E --> H
    E --> I
    
    F --> J
    G --> J
    H --> J
    I --> L
    
    F --> K
    G --> K
    
    H --> M
    I --> N
    I --> O
```

## 技术栈

### 后端
- **语言：** Java 17
- **框架：** Spring Boot 3.x
- **数据库：** MySQL 8.0
- **缓存：** Redis 7.x
- **消息队列：** RabbitMQ / Kafka

### 前端
- **框架：** React 18 / Vue 3
- **UI 库：** Ant Design / Element Plus
- **状态管理：** Redux / Pinia

### 基础设施
- **容器化：** Docker + Kubernetes
- **CI/CD：** GitHub Actions / GitLab CI
- **监控：** Prometheus + Grafana
- **日志：** ELK Stack

## 设计原则

!!! success "核心原则"
    - 高可用：服务无单点故障
    - 可扩展：支持水平扩展
    - 安全性：数据加密、权限控制
    - 可观测：全链路监控和日志

---

**最后更新：** 2026-03-12
