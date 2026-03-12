# 部署指南

## 环境要求

### 硬件要求
- **CPU：** 4核及以上
- **内存：** 8GB 及以上
- **磁盘：** 100GB SSD

### 软件要求
- **操作系统：** Ubuntu 20.04+ / CentOS 8+
- **Java：** OpenJDK 17
- **MySQL：** 8.0+
- **Redis：** 7.0+
- **Docker：** 20.10+

## 部署流程

### 1. 准备环境

```bash
# 安装 Java
sudo apt install openjdk-17-jdk

# 安装 MySQL
sudo apt install mysql-server

# 安装 Redis
sudo apt install redis-server

# 启动服务
sudo systemctl start mysql
sudo systemctl start redis
```

### 2. 数据库初始化

```bash
# 创建数据库
mysql -u root -p << EOF
CREATE DATABASE app_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON app_db.* TO 'app_user'@'localhost';
FLUSH PRIVILEGES;
EOF

# 执行迁移脚本
mysql -u app_user -p app_db < scripts/schema.sql
```

### 3. 配置应用

```yaml title="application-prod.yml"
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/app_db
    username: app_user
    password: ${DB_PASSWORD}
  
  redis:
    host: localhost
    port: 6379

server:
  port: 8080

stripe:
  api-key: ${STRIPE_API_KEY}
  webhook-secret: ${STRIPE_WEBHOOK_SECRET}
```

### 4. 构建和部署

```bash
# 构建项目
./mvnw clean package -DskipTests

# 启动应用
java -jar target/app-1.0.0.jar --spring.profiles.active=prod
```

## Docker 部署

### Dockerfile

```dockerfile title="Dockerfile"
FROM openjdk:17-jdk-slim

WORKDIR /app

COPY target/app-1.0.0.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

### docker-compose.yml

```yaml title="docker-compose.yml"
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=prod
      - DB_PASSWORD=${DB_PASSWORD}
      - STRIPE_API_KEY=${STRIPE_API_KEY}
    depends_on:
      - mysql
      - redis

  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: app_db
    volumes:
      - mysql_data:/var/lib/mysql

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  mysql_data:
  redis_data:
```

### 部署命令

```bash
# 启动所有服务
docker-compose up -d

# 查看日志
docker-compose logs -f app

# 停止服务
docker-compose down
```

## 健康检查

```bash
# 检查应用状态
curl http://localhost:8080/actuator/health

# 预期响应
{
  "status": "UP"
}
```

## 监控和日志

### 日志位置
```
/var/log/app/application.log
```

### 日志级别
```yaml
logging:
  level:
    root: INFO
    com.example: DEBUG
  file:
    name: /var/log/app/application.log
```

---

**最后更新：** 2026-03-12
