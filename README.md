# Nginx 多架构一键编译

一键编译 Nginx，自动生成 10 个架构的优化版本。

## 🚀 使用

```bash
./nginx_build_all
```

## 📦 输出 (10个架构)

编译产物位于 `./builds/` 目录：

### x86_64 系列 (4个)
- `nginx-x86_64_v1` - 基线 (100%兼容)
- `nginx-x86_64_v2` - SSE4.2 (2008+)
- `nginx-x86_64_v3` - AVX2 (2013+, **推荐**, 含 mimalloc+LTO)
- `nginx-x86_64_v4` - AVX512 (2017+)

### ARM64 系列 (6个)
- `nginx-armv8` - 基线
- `nginx-armv8_crc` - **推荐** (树莓派4, AWS Graviton)
- `nginx-armv8_crypto` - 加密优化
- `nginx-armv8.2` - Graviton2
- `nginx-armv8.4` - 现代ARM
- `nginx-armv9` - Graviton3

## ✨ 特性

- ✅ **BoringSSL** - HTTP/3 (QUIC) + TLS 1.3
- ✅ **Brotli** + **Zstandard** 压缩
- ✅ **Cloudflare zlib** - 优化的 gzip
- ✅ **安全加固** - Stack Protector, RELRO, FORTIFY_SOURCE
- ✅ **架构优化** - 每个架构专门优化
- ⚡ **mimalloc** - 高性能内存分配器 (x86_64本地)
- ⚡ **LTO** - 链接时优化

## ⏱️ 编译时间

- **预计**: 30-60 分钟
- **并行**: 依赖库并行编译
- **串行**: Nginx 逐个架构编译

## 🎯 推荐架构

### 通用部署
- `x86_64_v1` + `armv8_crc` - 最大兼容

### 性能优先
- `x86_64_v3` + `armv8.2` - 现代服务器

### 云平台
- **AWS**: `x86_64_v3` + `armv8_crc` + `armv9`
- **阿里云**: `x86_64_v3` + `x86_64_v4`
- **华为云**: `x86_64_v3` + `armv8.2`

## 📊 架构覆盖率

| 架构 | 覆盖率 | 适用范围 |
|------|--------|----------|
| x86_64_v1 | 100% | 所有 x86_64 CPU |
| x86_64_v2 | 98% | 2008年后 |
| x86_64_v3 | 95% | 2013年后 |
| x86_64_v4 | 15% | 高端服务器 |
| armv8 | 100% | 所有 ARM64 |
| armv8_crc | 95% | 主流 ARM64 |
| armv8.2+ | 60% | 现代 ARM64 |

## 🔍 验证

```bash
# 查看版本
./builds/nginx-x86_64_v3 -V

# 查看架构
file ./builds/nginx-x86_64_v3

# 查看大小
ls -lh ./builds/
```

## 🛠️ 系统要求

- Debian/Ubuntu Linux
- 8GB+ RAM
- 20GB+ 磁盘空间
- 自动安装所有依赖

---

**Nginx 版本**: 1.29.3  
**一键编译，十架构齐全** 🚀
