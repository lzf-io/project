# REST API 文档

## API 概览

Base URL: `https://api.example.com/v1`

所有 API 需要在请求头中携带认证 Token：
```http
Authorization: Bearer {access_token}
```

---

## 1. 产品管理 API

### 1.1 创建产品

```http
POST /api/products
Content-Type: application/json

{
  "type": "physical",          // physical | digital
  "name": "产品名称",
  "description": "产品描述",
  "category_id": 101,
  "price": 99.00,
  "images": [
    "https://cdn.example.com/image1.jpg"
  ],
  "metadata": {
    "brand": "品牌名",
    "model": "型号"
  }
}
```

**响应：**
```json
{
  "code": 0,
  "message": "success",
  "data": {
    "id": 123456,
    "type": "physical",
    "name": "产品名称",
    "status": "active",
    "created_at": "2026-03-12T10:00:00Z"
  }
}
```

### 1.2 查询产品列表

```http
GET /api/products?page=1&size=20&type=digital&status=active
```

**响应：**
```json
{
  "code": 0,
  "data": {
    "items": [
      {
        "id": 123456,
        "name": "产品名称",
        "type": "digital",
        "price": 99.00,
        "thumbnail": "https://cdn.example.com/thumb.jpg"
      }
    ],
    "total": 100,
    "page": 1,
    "size": 20
  }
}
```

### 1.3 查询产品详情

```http
GET /api/products/{product_id}
```

### 1.4 更新产品

```http
PUT /api/products/{product_id}
Content-Type: application/json

{
  "name": "新产品名称",
  "price": 149.00
}
```

### 1.5 删除产品

```http
DELETE /api/products/{product_id}
```

---

## 2. AI 生成 API

### 2.1 文生图

```http
POST /api/generate/text-to-image
Content-Type: application/json

{
  "prompt": "一只可爱的小猫在花园里玩耍，阳光明媚，高清写实风格",
  "quantity": 4,
  "options": {
    "width": 1024,
    "height": 1024,
    "style": "realistic",      // realistic | anime | abstract
    "negative_prompt": "模糊，低质量"
  }
}
```

**响应：**
```json
{
  "code": 0,
  "data": {
    "task_id": "task_abc123xyz",
    "status": "pending",
    "estimated_time": 60,
    "credits_locked": 40
  }
}
```

### 2.2 图生图

```http
POST /api/generate/image-to-image
Content-Type: application/json

{
  "source_image_url": "https://cdn.example.com/source.jpg",
  "prompt": "转换为水彩画风格",
  "quantity": 1,
  "options": {
    "strength": 0.75,          // 0.0 - 1.0
    "style": "watercolor"
  }
}
```

**响应：**
```json
{
  "code": 0,
  "data": {
    "task_id": "task_def456uvw",
    "status": "pending"
  }
}
```

### 2.3 AI 增强

```http
POST /api/generate/enhance
Content-Type: application/json

{
  "image_url": "https://cdn.example.com/original.jpg",
  "enhancement_type": "upscale",  // upscale | denoise | colorize
  "options": {
    "scale_factor": 2            // 2x | 4x
  }
}
```

### 2.4 查询生成任务状态

```http
GET /api/generate/tasks/{task_id}
```

**响应：**
```json
{
  "code": 0,
  "data": {
    "task_id": "task_abc123xyz",
    "status": "completed",        // pending | processing | completed | failed
    "progress": 100,
    "results": [
      {
        "image_url": "https://cdn.example.com/result1.jpg",
        "thumbnail_url": "https://cdn.example.com/result1_thumb.jpg"
      },
      {
        "image_url": "https://cdn.example.com/result2.jpg",
        "thumbnail_url": "https://cdn.example.com/result2_thumb.jpg"
      }
    ],
    "credits_used": 40,
    "created_at": "2026-03-12T10:00:00Z",
    "completed_at": "2026-03-12T10:01:23Z"
  }
}
```

### 2.5 取消生成任务

```http
POST /api/generate/tasks/{task_id}/cancel
```

**响应：**
```json
{
  "code": 0,
  "message": "任务已取消，积分已退回"
}
```

---

## 3. 积分管理 API

### 3.1 查询积分余额

```http
GET /api/credits/balance
```

**响应：**
```json
{
  "code": 0,
  "data": {
    "balance": 500,
    "locked": 40,
    "available": 460,
    "level": "premium",
    "updated_at": "2026-03-12T10:00:00Z"
  }
}
```

### 3.2 查询积分交易记录

```http
GET /api/credits/transactions?page=1&size=20&type=deduct
```

**参数：**
- `type`: `lock` | `deduct` | `unfreeze` | `recharge`

**响应：**
```json
{
  "code": 0,
  "data": {
    "items": [
      {
        "id": 789,
        "type": "deduct",
        "amount": -40,
        "balance_after": 460,
        "description": "文生图生成消耗",
        "task_id": "task_abc123xyz",
        "created_at": "2026-03-12T10:01:23Z"
      }
    ],
    "total": 50,
    "page": 1
  }
}
```

### 3.3 充值积分

```http
POST /api/credits/recharge
Content-Type: application/json

{
  "package_id": "pkg_100",      // 套餐 ID
  "payment_method": "alipay"    // alipay | wechat | stripe
}
```

**响应：**
```json
{
  "code": 0,
  "data": {
    "order_id": "order_xyz789",
    "payment_url": "https://pay.example.com/order_xyz789",
    "amount": 99.00,
    "credits": 1000
  }
}
```

---

## 4. 限制与配额 API

### 4.1 查询用户配额

```http
GET /api/limits/quota
```

**响应：**
```json
{
  "code": 0,
  "data": {
    "level": "free",
    "daily_quota": {
      "limit": 3,
      "used": 2,
      "remaining": 1,
      "reset_at": "2026-03-13T00:00:00Z"
    },
    "monthly_quota": {
      "limit": 50,
      "used": 15,
      "remaining": 35
    },
    "credit_balance": 100
  }
}
```

### 4.2 检查生成权限

```http
POST /api/limits/check
Content-Type: application/json

{
  "action": "generate_text_to_image",
  "quantity": 4
}
```

**响应：**
```json
{
  "code": 0,
  "data": {
    "allowed": false,
    "reason": "daily_quota_exceeded",
    "current_usage": 3,
    "limit": 3,
    "upgrade_suggestion": {
      "level": "basic",
      "daily_quota": 20,
      "price": 19.90
    }
  }
}
```

---

## 5. DS 推荐 API（Dropshipping）

### 5.1 获取推荐产品列表

```http
GET /api/ds/recommend?category=women_clothing&limit=20
```

**响应：**
```json
{
  "code": 0,
  "data": {
    "quota": {
      "level": "pro",
      "monthly_limit": 500,
      "used": 120,
      "remaining": 380
    },
    "items": [
      {
        "id": "ds_aliexpress_123456",
        "title": "Summer Casual Dress Women",
        "image_url": "https://ae01.alicdn.com/...",
        "supplier": "aliexpress",
        "supplier_product_id": "1005004123456789",
        "price_analysis": {
          "supply_price": 15.99,
          "suggested_retail_price": 39.99,
          "profit_margin": 60.0,
          "profit_amount": 24.00
        },
        "market_data": {
          "sales_30d": 5420,
          "rating": 4.8,
          "reviews_count": 1230
        },
        "shipping": {
          "estimated_days": "15-25",
          "free_shipping": true
        },
        "tags": ["trending", "high_profit"]
      }
    ],
    "total": 150
  }
}
```

### 5.2 查询产品详情

```http
GET /api/ds/products/{ds_product_id}
```

**响应：**
```json
{
  "code": 0,
  "data": {
    "id": "ds_aliexpress_123456",
    "title": "Summer Casual Dress Women",
    "description": "详细描述...",
    "images": [
      "https://ae01.alicdn.com/image1.jpg",
      "https://ae01.alicdn.com/image2.jpg"
    ],
    "variants": [
      {
        "sku": "SKU001",
        "color": "Red",
        "size": "S",
        "price": 15.99,
        "stock": 999
      }
    ],
    "supplier_info": {
      "name": "AliExpress Seller",
      "rating": 4.9,
      "response_rate": 98.5
    }
  }
}
```

### 5.3 添加推荐产品到商户库

```http
POST /api/ds/products/add
Content-Type: application/json

{
  "ds_product_id": "ds_aliexpress_123456",
  "customize": {
    "retail_price": 49.99,
    "inventory_management": "auto",
    "auto_sync": true
  }
}
```

**响应：**
```json
{
  "code": 0,
  "data": {
    "product_id": 789012,
    "status": "active",
    "quota_consumed": 1,
    "quota_remaining": 379
  }
}
```

### 5.4 查询推荐历史

```http
GET /api/ds/history?page=1&size=20
```

**响应：**
```json
{
  "code": 0,
  "data": {
    "items": [
      {
        "id": 123,
        "ds_product_id": "ds_aliexpress_123456",
        "product_id": 789012,
        "recommended_at": "2026-03-12T10:00:00Z",
        "added_at": "2026-03-12T10:05:00Z",
        "status": "active"
      }
    ]
  }
}
```

---

## 6. 套餐管理 API

### 6.1 查询套餐信息

```http
GET /api/subscription
```

**响应：**
```json
{
  "code": 0,
  "data": {
    "level": "pro",
    "status": "active",
    "ds_quota": {
      "monthly_limit": 500,
      "used_current_month": 120,
      "remaining": 380,
      "reset_at": "2026-04-01T00:00:00Z"
    },
    "features": {
      "ai_generation": true,
      "batch_generation": true,
      "max_batch_size": 16,
      "priority_queue": "high",
      "api_access": true
    },
    "billing": {
      "amount": 299.00,
      "currency": "CNY",
      "billing_cycle": "monthly",
      "next_billing_date": "2026-04-12T00:00:00Z"
    },
    "start_date": "2026-03-12T10:00:00Z",
    "expire_date": "2026-04-12T10:00:00Z"
  }
}
```

### 6.2 查询可升级套餐

```http
GET /api/subscription/plans
```

**响应：**
```json
{
  "code": 0,
  "data": {
    "current_level": "basic",
    "available_plans": [
      {
        "level": "pro",
        "name": "专业版",
        "price": {
          "monthly": 299.00,
          "yearly": 2990.00,
          "yearly_discount": 17
        },
        "features": {
          "ds_quota_monthly": 500,
          "batch_generation_max": 16,
          "priority_queue": "high",
          "api_access": true,
          "dedicated_support": false
        }
      },
      {
        "level": "enterprise",
        "name": "企业版",
        "price": {
          "monthly": 999.00,
          "yearly": 9990.00
        },
        "features": {
          "ds_quota_monthly": -1,
          "batch_generation_max": -1,
          "priority_queue": "highest",
          "api_access": true,
          "dedicated_support": true
        }
      }
    ]
  }
}
```

### 6.3 升级套餐

```http
POST /api/subscription/upgrade
Content-Type: application/json

{
  "target_level": "pro",
  "billing_cycle": "monthly",
  "payment_method": "stripe"
}
```

**响应：**
```json
{
  "code": 0,
  "data": {
    "order_id": "order_sub_upgrade_123",
    "payment_url": "https://checkout.stripe.com/...",
    "amount": 299.00,
    "effective_immediately": true
  }
}
```

---

## 6. 用户等级 API

### 6.1 查询用户等级信息

```http
GET /api/users/level
```

**响应：**
```json
{
  "code": 0,
  "data": {
    "current_level": "free",
    "daily_quota": 3,
    "monthly_quota": 50,
    "credit_discount": 1.0,
    "features": [
      "text_to_image",
      "image_to_image"
    ],
    "upgrade_options": [
      {
        "level": "basic",
        "price": 19.90,
        "daily_quota": 20,
        "monthly_quota": 500,
        "credit_discount": 0.9,
        "additional_features": [
          "ai_enhance",
          "priority_queue"
        ]
      },
      {
        "level": "premium",
        "price": 49.90,
        "daily_quota": -1,
        "monthly_quota": -1,
        "credit_discount": 0.8,
        "additional_features": [
          "batch_generation",
          "api_access",
          "no_watermark"
        ]
      }
    ]
  }
}
```

### 6.2 升级会员等级

```http
POST /api/users/level/upgrade
Content-Type: application/json

{
  "target_level": "basic",
  "billing_period": "monthly",    // monthly | yearly
  "payment_method": "alipay"
}
```

**响应：**
```json
{
  "code": 0,
  "data": {
    "order_id": "order_upgrade_123",
    "payment_url": "https://pay.example.com/...",
    "amount": 19.90,
    "effective_date": "2026-03-12T10:00:00Z",
    "expire_date": "2026-04-12T10:00:00Z"
  }
}
```

---

## 错误码说明

| Code | 说明 | 处理建议 |
|------|------|----------|
| 0 | 成功 | - |
| 40001 | 参数错误 | 检查请求参数 |
| 40002 | 认证失败 | 重新登录 |
| 40003 | 权限不足 | 升级会员等级 |
| 40004 | 资源不存在 | 检查资源 ID |
| 40005 | 配额不足 | 等待重置或升级 |
| 40006 | 积分不足 | 充值积分 |
| 40007 | 生成任务失败 | 重试或联系客服 |
| 50001 | 服务器错误 | 稍后重试 |
| 50002 | AI 服务不可用 | 稍后重试 |
| 50003 | 数据库错误 | 联系客服 |

---

## WebSocket 实时通知

连接地址: `wss://api.example.com/v1/ws?token={access_token}`

### 生成进度通知

```json
{
  "type": "generation_progress",
  "task_id": "task_abc123xyz",
  "progress": 50,
  "status": "processing",
  "message": "正在生成第2张图片..."
}
```

### 生成完成通知

```json
{
  "type": "generation_completed",
  "task_id": "task_abc123xyz",
  "results": [
    {
      "image_url": "https://cdn.example.com/result1.jpg"
    }
  ]
}
```

### 积分变动通知

```json
{
  "type": "credits_changed",
  "balance": 460,
  "locked": 0,
  "change_amount": -40,
  "reason": "生成任务消耗"
}
```

---

## 请求限流

| API 类型 | 限流规则 |
|---------|---------|
| 查询类 API | 100次/分钟/用户 |
| 生成类 API | 10次/分钟/用户 |
| 充值/升级 API | 5次/分钟/用户 |

超过限流返回：
```json
{
  "code": 42901,
  "message": "请求过于频繁，请稍后再试",
  "retry_after": 60
}
```

---

**最后更新：** 2026-03-12
