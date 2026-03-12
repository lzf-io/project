# 系统架构设计

## 整体架构

```mermaid
graph TB
    subgraph 客户端层
        A[Web 前端<br/>React/Vue]
        B[移动端 App<br/>iOS/Android]
        C[管理后台<br/>Admin Panel]
    end
    
    subgraph CDN/负载均衡
        D[CDN<br/>CloudFlare]
        E[Load Balancer<br/>Nginx]
    end
    
    subgraph API 网关层
        F[API Gateway<br/>Spring Cloud Gateway]
        G[认证中心<br/>OAuth 2.0]
    end
    
    subgraph 业务服务层
        H[产品服务<br/>Product Service]
        I[AI 生成服务<br/>Generation Service]
        J[推荐服务<br/>Recommendation Service]
        K[积分服务<br/>Credits Service]
        L[限制服务<br/>Limit Service]
        M[用户服务<br/>User Service]
        N[支付服务<br/>Payment Service]
    end
    
    subgraph AI 计算层
        O[文生图引擎<br/>Stable Diffusion]
        P[图生图引擎<br/>ControlNet]
        Q[AI 增强<br/>ESRGAN]
        R[推荐算法<br/>Deep Learning]
    end
    
    subgraph 数据存储层
        S[(MySQL 主库<br/>业务数据)]
        T[(MySQL 从库<br/>读副本)]
        U[(Redis 集群<br/>缓存/会话)]
        V[(MongoDB<br/>生成记录)]
        W[对象存储<br/>OSS/S3]
    end
    
    subgraph 消息与任务
        X[消息队列<br/>RabbitMQ]
        Y[任务调度<br/>Celery]
    end
    
    subgraph 监控与日志
        Z[监控<br/>Prometheus]
        AA[日志<br/>ELK Stack]
        AB[链路追踪<br/>Jaeger]
    end
    
    A --> D
    B --> D
    C --> E
    D --> E
    E --> F
    
    F --> G
    F --> H
    F --> I
    F --> J
    F --> K
    F --> L
    F --> M
    F --> N
    
    H --> S
    I --> X
    I --> V
    J --> R
    J --> T
    K --> S
    K --> U
    L --> U
    M --> S
    N --> S
    
    X --> Y
    Y --> O
    Y --> P
    Y --> Q
    
    H --> W
    I --> W
    
    H --> Z
    I --> Z
    J --> Z
    
    H --> AA
    I --> AA
    
    F --> AB
    
    style F fill:#635bff,color:#fff
    style I fill:#635bff,color:#fff
    style K fill:#00d084,color:#fff
    style L fill:#ff9800,color:#000
```

---

## 技术选型

### 后端技术栈

| 组件 | 技术选型 | 说明 |
|------|---------|------|
| 开发语言 | Java 17 | 主业务服务 |
| 开发语言 | Python 3.10 | AI 相关服务 |
| 应用框架 | Spring Boot 3.x | 微服务框架 |
| API 网关 | Spring Cloud Gateway | 统一接入层 |
| 服务注册 | Nacos / Consul | 服务发现 |
| 配置中心 | Nacos / Apollo | 配置管理 |
| 认证授权 | Spring Security + OAuth 2.0 | 安全框架 |

### 数据存储

| 组件 | 技术选型 | 用途 |
|------|---------|------|
| 关系数据库 | MySQL 8.0 | 业务数据（产品、用户、订单） |
| 缓存 | Redis 7.x Cluster | 会话、限流、热点数据 |
| 文档数据库 | MongoDB 6.0 | 生成任务记录、日志 |
| 对象存储 | AWS S3 / 阿里云 OSS | 图片、文件存储 |
| 搜索引擎 | Elasticsearch 8.x | 产品搜索、日志分析 |

### AI 与算法

| 组件 | 技术选型 | 用途 |
|------|---------|------|
| 文生图 | Stable Diffusion XL | 核心生成引擎 |
| 图生图 | ControlNet + Img2Img | 图像转换 |
| 超分辨率 | ESRGAN / Real-ESRGAN | 图像增强 |
| 推荐算法 | 协同过滤 + DNN | 个性化推荐 |
| GPU 调度 | NVIDIA Triton | 模型推理服务 |

### 消息与任务

| 组件 | 技术选型 | 用途 |
|------|---------|------|
| 消息队列 | RabbitMQ 3.x | 异步任务、事件通知 |
| 任务调度 | Celery (Python) | AI 生成任务执行 |
| 分布式锁 | Redisson | 并发控制 |

### 监控与运维

| 组件 | 技术选型 | 用途 |
|------|---------|------|
| 监控告警 | Prometheus + Grafana | 指标监控 |
| 日志采集 | Filebeat + Logstash | 日志收集 |
| 日志存储 | Elasticsearch | 日志存储与查询 |
| 日志可视化 | Kibana | 日志分析 |
| 链路追踪 | Jaeger / Zipkin | 分布式追踪 |
| APM | Skywalking | 应用性能管理 |

---

## 核心服务设计

### 1. AI 生成服务

**职责：**
- 接收生成请求
- 任务调度与队列管理
- 进度跟踪与结果通知

**关键设计：**
- 异步处理：避免阻塞用户请求
- 优先级队列：付费用户优先
- 失败重试：自动重试 3 次
- 降级策略：AI 服务不可用时返回默认图

**技术实现：**
```java
@Service
public class GenerationService {
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    @Autowired
    private CreditsService creditsService;
    
    public TaskResponse generateTextToImage(GenerateRequest request) {
        // 1. 锁定积分
        creditsService.lockCredits(request.getUserId(), calculateCost(request));
        
        // 2. 创建任务记录
        Task task = taskRepository.save(new Task(request));
        
        // 3. 发送到消息队列
        rabbitTemplate.convertAndSend("ai.generation", task);
        
        return new TaskResponse(task.getId(), "pending");
    }
}
```

### 2. 积分服务

**职责：**
- 积分账户管理
- 积分锁定/扣减/解冻
- 交易记录

**关键设计：**
- 事务一致性：使用数据库事务
- 并发安全：行锁 + 乐观锁
- 幂等性：基于 transaction_id 去重

**数据库设计：**
```sql
CREATE TABLE credit_accounts (
    user_id BIGINT PRIMARY KEY,
    balance INT NOT NULL DEFAULT 0,
    locked INT NOT NULL DEFAULT 0,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    version INT NOT NULL DEFAULT 0,
    INDEX idx_user_id (user_id)
) ENGINE=InnoDB;

CREATE TABLE credit_transactions (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    task_id VARCHAR(64),
    type ENUM('lock', 'deduct', 'unfreeze', 'recharge'),
    amount INT NOT NULL,
    balance_before INT,
    balance_after INT,
    status ENUM('pending', 'completed', 'failed'),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_user_id (user_id),
    INDEX idx_task_id (task_id)
) ENGINE=InnoDB;
```

### 3. 限制服务

**职责：**
- 用户配额检查
- 限流控制
- 升级引导

**关键设计：**
- Redis 缓存：减少数据库查询
- 分层限制：按用户等级配置
- 软限制：超限提示而非拒绝

**Redis 数据结构：**
```
# 每日配额
daily_quota:{user_id}:{date} = usage_count
EXPIRE 86400  # 24小时过期

# 用户等级配额
user_level:{user_id} = {
    "level": "free",
    "daily_limit": 3,
    "monthly_limit": 50
}
```

---

## 性能优化

### 1. 数据库优化

**读写分离：**
```yaml
datasource:
  master:
    url: jdbc:mysql://master-db:3306/app
  slave:
    url: jdbc:mysql://slave-db:3306/app
```

**索引优化：**
- 产品表：`idx_user_id_created_at`
- 任务表：`idx_user_id_status_created_at`
- 交易表：`idx_user_id_created_at`

**分库分表：**
- 生成任务表按月分表
- 交易记录表按用户 ID hash 分库

### 2. 缓存策略

**多级缓存：**
```
浏览器缓存 → CDN → Redis → MySQL
```

**缓存预热：**
- 热门产品
- 用户等级配置
- 推荐结果

**缓存失效：**
```java
@CacheEvict(value = "products", key = "#productId")
public void updateProduct(Long productId, Product product) {
    productRepository.save(product);
}
```

### 3. AI 推理优化

**批量推理：**
- 合并多个用户请求
- 提高 GPU 利用率

**模型量化：**
- FP16 精度推理
- 降低显存占用

**动态缩放：**
- 根据负载自动扩容 GPU 实例

---

## 安全设计

### 1. 认证与授权

**JWT Token：**
```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**权限模型：**
```
用户 → 角色 → 权限
```

### 2. 数据加密

- **传输加密：** HTTPS/TLS
- **存储加密：** 敏感字段 AES-256
- **密码加密：** BCrypt

### 3. 防刷防爬

- **IP 限流：** Nginx + rate limit
- **签名验证：** API 请求签名
- **行为分析：** 异常检测

---

## 容灾与高可用

### 1. 服务高可用

**多实例部署：**
```yaml
replicas: 3
```

**健康检查：**
```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
```

### 2. 数据备份

- **MySQL：** 主从复制 + 每日全量备份
- **Redis：** AOF + RDB 持久化
- **对象存储：** 跨区域复制

### 3. 灾难恢复

**RTO：** 30 分钟  
**RPO：** 5 分钟

---

**最后更新：** 2026-03-12
