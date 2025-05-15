# Nginx 编译脚本

这个脚本用于从源代码编译Nginx，并包含对Lua模块和HTTP/2支持，可用于代理gRPC流量。脚本经过精简和优化，移除了不必要的依赖项，采用模块化设计，便于维护和扩展。

## 支持的功能

- **HTTP/2 和 HTTP/3**: 支持最新的HTTP协议
- **TLS**: 使用OpenSSL支持HTTPS和安全连接
- **gRPC代理**: 通过HTTP/2和Stream模块代理gRPC流量
- **Lua**: 内置Lua模块，可以使用Lua脚本直接在Nginx配置中编写业务逻辑
- **Brotli**: Google开发的压缩算法，比GZIP有更好的压缩率
- **ZSTD**: 提供更快的压缩和解压速度
- **性能优化**:
  - 动态TLS记录大小调整
  - mimalloc内存分配器 (显著提升内存管理性能)
  - PCRE JIT编译器
  - 多种性能优化编译参数

## 版本信息

- Nginx: 1.27.5
- OpenSSL: 最新版 (来自quictls/openssl)
- PCRE: 8.45
- ZLIB: 1.3.1
- LuaJIT: 2.1-20231117

## 设计原则

脚本遵循以下软件工程设计原则:

- **单一职责原则**: 每个函数只负责一项特定任务
- **高内聚低耦合**: 相关功能集中在一起，减少函数间依赖
- **模块化设计**: 将安装步骤拆分为独立模块，便于维护和扩展
- **错误处理**: 包含基本的错误检查和日志记录

## 使用方法

### 编译

1. 赋予脚本执行权限
   ```bash
   chmod +x nginx_compile
   ```

2. 运行脚本 (需要root权限)
   ```bash
   sudo ./nginx_compile
   ```

3. 安装Nginx
   ```bash
   cd nginx-1.27.5
   sudo make install
   ```

### 验证安装

```bash
sudo nginx -t
```

### 启动Nginx

```bash
sudo systemctl start nginx
# 或
sudo nginx
```

## gRPC代理配置示例

由于Nginx 1.27.5不直接支持`--with-http_grpc_module`选项，我们可以使用HTTP/2和Stream模块来代理gRPC流量。以下是两种常用的配置方法：

### 方法1：使用HTTP/2代理gRPC

```nginx
http {
    server {
        listen 443 ssl http2;
        server_name grpc.example.com;

        ssl_certificate /path/to/cert.pem;
        ssl_certificate_key /path/to/key.pem;

        # 代理到后端gRPC服务
        location / {
            # 将请求发送到后端gRPC服务器
            proxy_pass https://backend_grpc_server;
            proxy_http_version 1.1;
            
            # 这些设置对gRPC很重要
            proxy_set_header Connection "";
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            
            # gRPC需要较长的超时时间
            proxy_read_timeout 300s;
            proxy_send_timeout 300s;
            
            # 对于大型gRPC消息
            proxy_buffer_size 128k;
            proxy_buffers 4 256k;
            proxy_busy_buffers_size 256k;
        }
    }
}

# 定义后端服务器
upstream backend_grpc_server {
    server grpc.backend.service:50051;
}
```

### 方法2：使用Stream模块代理gRPC (TCP层代理)

```nginx
# Stream模块配置
stream {
    upstream grpc_backend {
        server backend.grpc.service:50051;
    }

    server {
        listen 50051;
        
        # 直接转发TCP流量到后端
        proxy_pass grpc_backend;
        
        # 对TLS流量进行SSL预读取，便于路由
        proxy_ssl_preread on;
    }
}
```

### 使用ngx_http_proxy_connect_module代理gRPC CONNECT请求

本编译包含了`ngx_http_proxy_connect_module`，它支持HTTP CONNECT方法，这对某些gRPC客户端很有用：

```nginx
server {
    listen 443;
    server_name proxy.example.com;

    # 对CONNECT请求启用代理
    proxy_connect;
    proxy_connect_allow 443;
    proxy_connect_connect_timeout 10s;
    proxy_connect_read_timeout 10s;
    proxy_connect_send_timeout 10s;

    location / {
        return 404;
    }
}
```

## Lua配置示例

在Nginx配置文件中使用Lua：

```nginx
http {
    # 设置Lua模块路径
    lua_package_path "/path/to/lua/?.lua;;";

    server {
        listen 80;
        server_name example.com;

        location /lua_content {
            # 直接在Nginx中使用Lua代码
            content_by_lua_block {
                ngx.say("<p>Hello, Lua!</p>")
            }
        }

        location /lua_file {
            # 使用外部Lua文件
            content_by_lua_file /path/to/lua/script.lua;
        }
    }
}
```

## 优化说明

与原始脚本相比，此优化版本:

1. 移除了不必要的依赖项
2. 使用系统包管理器安装基本依赖
3. 采用函数模块化设计，提高可维护性
4. 添加了错误处理机制
5. 保留了mimalloc内存分配器 (提供实质性性能提升)
6. 保留了PCRE和ZLIB的手动编译 (可获得更好的性能优化)

## 注意事项

- 该脚本需要在root权限下运行
- 编译过程可能需要较长时间，取决于系统性能
- 脚本会自动优化编译参数以提高性能
- Nginx 1.27.5不直接支持`--with-http_grpc_module`，但可通过HTTP/2和Stream模块代理gRPC流量

## 故障排除

如果遇到编译错误，可以尝试：

1. 确保所有依赖项已正确安装
2. 检查是否有足够的磁盘空间
3. 查看具体错误信息，通常在编译输出中会有详细说明

对于gRPC或Lua相关的问题：

- 确保环境变量LUAJIT_LIB和LUAJIT_INC设置正确
- 检查gRPC服务器是否正确配置并可访问 