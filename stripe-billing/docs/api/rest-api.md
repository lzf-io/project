# REST API 规范

## API 设计原则

### RESTful 风格
- 使用标准 HTTP 方法（GET、POST、PUT、DELETE）
- 资源命名使用复数形式
- URL 路径表示资源层级关系

### 示例

| 方法 | 路径 | 描述 |
|------|------|------|
| GET | `/api/users` | 获取用户列表 |
| GET | `/api/users/{id}` | 获取单个用户 |
| POST | `/api/users` | 创建用户 |
| PUT | `/api/users/{id}` | 更新用户 |
| DELETE | `/api/users/{id}` | 删除用户 |

## 请求格式

### 请求头
```http
Content-Type: application/json
Authorization: Bearer {access_token}
X-Request-ID: {uuid}
```

### 请求体示例
```json
{
  "name": "张三",
  "email": "zhangsan@example.com",
  "phone": "13800138000"
}
```

## 响应格式

### 成功响应
```json
{
  "code": 0,
  "message": "success",
  "data": {
    "id": 123,
    "name": "张三",
    "email": "zhangsan@example.com"
  },
  "timestamp": 1710234567890
}
```

### 错误响应
```json
{
  "code": 40001,
  "message": "用户不存在",
  "errors": [
    {
      "field": "userId",
      "message": "无效的用户 ID"
    }
  ],
  "timestamp": 1710234567890
}
```

## HTTP 状态码

| 状态码 | 含义 | 使用场景 |
|--------|------|----------|
| 200 | OK | 请求成功 |
| 201 | Created | 资源创建成功 |
| 204 | No Content | 删除成功（无返回内容） |
| 400 | Bad Request | 请求参数错误 |
| 401 | Unauthorized | 未认证 |
| 403 | Forbidden | 无权限 |
| 404 | Not Found | 资源不存在 |
| 500 | Internal Server Error | 服务器错误 |

## 分页

### 请求参数
```
GET /api/users?page=1&size=20&sort=created_at,desc
```

### 响应格式
```json
{
  "code": 0,
  "data": {
    "items": [...],
    "total": 100,
    "page": 1,
    "size": 20,
    "has_more": true
  }
}
```

## 过滤和搜索

```
GET /api/users?status=active&name=张三&created_after=2024-01-01
```

## 版本控制

使用 URL 路径版本：
```
/api/v1/users
/api/v2/users
```

---

**最后更新：** 2026-03-12
