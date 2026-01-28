# nginx请求转发

nginx可以通过**location指令**配合**proxy_pass**实现根据请求地址代理到不同后端。以下是几种常见场景的配置示例：

## 一、基础配置模板

```nginx
http {
    # 后端服务器定义
    upstream backend1 {
        server 192.168.1.10:8080;
    }
    
    upstream backend2 {
        server 192.168.1.20:8080;
    }

    server {
        listen 80;
        server_name example.com;

        # 按路径规则分发
        location /api/v1/ {
            proxy_pass http://backend1;  # 以/api/v1开头的请求
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }

        location /api/v2/ {
            proxy_pass http://backend2;  # 以/api/v2开头的请求
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }

        location /static/ {
            proxy_pass http://static-backend/;  # 注意结尾的/号
        }

        # 默认路由
        location / {
            proxy_pass http://default-backend;
        }
    }
}
```

## 二、不同场景配置示例

### 1. 按路径前缀代理

```nginx
location ~ ^/user/(.*)$ {
    proxy_pass http://user-service/$1;  # /user/profile -> http://user-service/profile
}

location ~ ^/product/(.*)$ {
    proxy_pass http://product-service/$1;
}

# 精确匹配
location = /admin {
    proxy_pass http://admin-service;
}
```

### 2. 按域名代理

```nginx
server {
    listen 80;
    server_name api.example.com;
    
    location / {
        proxy_pass http://api-backend;
    }
}

server {
    listen 80;
    server_name admin.example.com;
    
    location / {
        proxy_pass http://admin-backend;
    }
}
```

### 3. 正则表达式匹配

```nginx
# 匹配图片请求
location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
    proxy_pass http://static-server;
}

# 排除某些路径
location ~ ^/(?!api|admin).*$ {
    proxy_pass http://web-frontend;
}
```

### 4. 带条件判断的代理

```nginx
# 根据参数代理
location /search {
    if ($arg_type = "product") {
        proxy_pass http://product-search;
    }
    if ($arg_type = "user") {
        proxy_pass http://user-search;
    }
    proxy_pass http://default-search;
}

# 根据请求方法代理
location /data {
    proxy_pass http://read-only-backend;
    
    # 只有POST请求转发到写服务
    if ($request_method = POST) {
        proxy_pass http://write-backend;
    }
}
```

### 5. 复杂场景：权重分配和负载均衡

```nginx
upstream main_backend {
    server backend1.example.com weight=3;
    server backend2.example.com weight=1;
}

upstream api_backend {
    server api1.example.com;
    server api2.example.com;
    server api3.example.com;
}

server {
    listen 80;
    
    location /api/ {
        proxy_pass http://api_backend;
        proxy_next_upstream error timeout http_500;
    }
    
    location / {
        proxy_pass http://main_backend;
        # 健康检查配置
        proxy_connect_timeout 5s;
        proxy_read_timeout 10s;
    }
}
```

## 三、完整配置文件示例

```nginx
user nginx;
worker_processes auto;

events {
    worker_connections 1024;
}

http {
    # 定义后端服务
    upstream auth_service {
        server 10.0.1.10:8000;
        server 10.0.1.11:8000 backup;
    }
    
    upstream user_service {
        server 10.0.2.10:8080;
    }
    
    upstream product_service {
        server 10.0.3.10:9000;
        server 10.0.3.11:9000;
    }

    # 主服务器配置
    server {
        listen 80;
        server_name gateway.example.com;
        
        # 认证服务
        location /auth/ {
            rewrite ^/auth/(.*)$ /$1 break;  # 去除/auth前缀
            proxy_pass http://auth_service;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
        
        # 用户服务
        location ~ ^/users/(.*)$ {
            proxy_pass http://user_service/$1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            
            # 超时设置
            proxy_connect_timeout 30s;
            proxy_send_timeout 60s;
            proxy_read_timeout 60s;
        }
        
        # 商品服务
        location /products/ {
            proxy_pass http://product_service/;
            
            # 缓存设置
            proxy_cache product_cache;
            proxy_cache_key $scheme$proxy_host$request_uri;
            proxy_cache_valid 200 302 10m;
        }
        
        # 静态文件
        location /static/ {
            root /var/www/static;
            expires 30d;
            add_header Cache-Control "public, immutable";
        }
        
        # 健康检查端点
        location /health {
            access_log off;
            return 200 "healthy\n";
            add_header Content-Type text/plain;
        }
        
        # 默认路由
        location / {
            proxy_pass http://default_backend;
            proxy_intercept_errors on;
            error_page 404 = @fallback;
        }
        
        location @fallback {
            proxy_pass http://fallback_service;
        }
        
        # 错误页面
        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
            root /usr/share/nginx/html;
        }
    }
    
    # 第二个虚拟主机
    server {
        listen 80;
        server_name api.example.com;
        
        location / {
            proxy_pass http://api_gateway;
            # API网关特定配置
            proxy_set_header X-API-Key $http_x_api_key;
            proxy_set_header X-API-Version "v1";
        }
    }
}
```

## 四、常用配置参数说明

```nginx
# 超时控制
proxy_connect_timeout 5s;    # 连接超时
proxy_send_timeout 60s;      # 发送超时
proxy_read_timeout 60s;      # 读取超时

# 缓冲区
proxy_buffering on;
proxy_buffer_size 4k;
proxy_buffers 8 4k;

# 重试机制
proxy_next_upstream error timeout http_502 http_503;

# 头部传递
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;

# 连接保持
proxy_http_version 1.1;
proxy_set_header Connection "";
```

## 五、调试技巧

1. **查看匹配顺序**：nginx按以下优先级匹配location
   - `=` 精确匹配
   - `^~` 前缀匹配
   - `~` 和 `~*` 正则匹配
   - `/` 通用前缀匹配

2. **调试命令**：

```bash
# 测试配置
nginx -t

# 重新加载配置
nginx -s reload

# 查看访问日志
tail -f /var/log/nginx/access.log

# 查看错误日志
tail -f /var/log/nginx/error.log
```

3. **日志记录代理信息**：

```nginx
location / {
    proxy_pass http://backend;
    access_log /var/log/nginx/proxy.log upstreamlog;
    
    # 自定义日志格式
    log_format upstreamlog '$remote_addr - $remote_user [$time_local] '
                           '"$request" $status $body_bytes_sent '
                           '"$http_referer" "$http_user_agent" '
                           'proxy: "$upstream_addr"';
}
```

根据实际需求选择合适的匹配方式，并注意location的匹配顺序，避免规则冲突。