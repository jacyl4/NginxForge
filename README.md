# Nginx 多架构编译系统

一键编译 Nginx，自动生成 10 个架构的优化二进制版本。

## 🚀 快速开始

### 编译所有架构
```bash
./nginx_build_all
```

### 编译单个架构
```bash
./nginx_build_arch --arch x86_64_v3
```

### 列出支持的架构
```bash
./nginx_build_arch --list-targets
```

---

## 📦 支持的架构

### x86_64 系列（4个）
| 架构 | 特性 | 兼容性 | 适用场景 |
|------|------|--------|----------|
| `x86_64_v1` | 基线 x86-64 | 100% | 最大兼容性 |
| `x86_64_v2` | SSE4.2 | 98% | 2008年后 CPU |
| `x86_64_v3` | AVX2 + FMA | 95% | **推荐**，2013年后 CPU，包含 mimalloc + LTO |
| `x86_64_v4` | AVX512 | 15% | 高端服务器 |

### ARM64 系列（6个）
| 架构 | 特性 | 兼容性 | 适用场景 |
|------|------|--------|----------|
| `armv8` | 基线 ARMv8 | 100% | 最大兼容性 |
| `armv8_crc` | ARMv8 + CRC32 | 95% | **推荐**，树莓派4，AWS Graviton |
| `armv8_crypto` | ARMv8 + Crypto | 85% | 加密优化 |
| `armv8.2` | ARMv8.2-A | 60% | AWS Graviton2 |
| `armv8.4` | ARMv8.4-A | 40% | 现代 ARM 服务器 |
| `armv9` | ARMv9-A | 20% | AWS Graviton3/4 |

---

## ✨ 编译特性

### 核心组件
- ✅ **Nginx** - 版本 1.29.3
- ✅ **BoringSSL** - Google 维护的 OpenSSL 分支，支持 HTTP/3 (QUIC) + TLS 1.3
- ✅ **Cloudflare zlib** - 优化的 gzip 压缩库

### 压缩模块
- ✅ **Brotli** - 高效的现代压缩算法
- ✅ **Zstandard** - Facebook 开发的高性能压缩

### 第三方模块
- ✅ **ngx_devel_kit** - Nginx 开发工具包

### 安全加固
- ✅ **Stack Protector** - 栈溢出保护
- ✅ **RELRO** - 只读重定位
- ✅ **FORTIFY_SOURCE** - 缓冲区溢出检测
- ✅ **TCP Fast Open** - TCP 快速打开支持

### 性能优化
- ⚡ **mimalloc** - 微软高性能内存分配器（仅 x86_64 本地编译）
- ⚡ **LTO** - 链接时优化（仅 x86_64 本地编译）
- ⚡ **并行构建** - BoringSSL 和 zlib 并行编译，节省约 10% 时间
- ⚡ **架构专用优化** - 每个架构使用对应的 `-march` 标志

---

## 📁 项目结构

```
nginx/
├── nginx_build_arch         # 单架构编译脚本（637行）
│                            # 支持：--arch <架构> [--skip-deps] [--list-targets]
│
├── nginx_build_all          # 批量编译脚本（179行）
│                            # 支持：[--arch <架构列表>] [--skip-deps] [--list-targets]
│
├── builds/                  # 编译产物目录
│   └── 1.29.3/
│       ├── nginx-x86_64_v1
│       ├── nginx-x86_64_v2
│       ├── nginx-x86_64_v3
│       ├── nginx-x86_64_v4
│       ├── nginx-armv8
│       ├── nginx-armv8_crc
│       ├── nginx-armv8_crypto
│       ├── nginx-armv8.2
│       ├── nginx-armv8.4
│       └── nginx-armv9
│
└── compile/                 # 编译工作目录（自动创建）
    ├── boringssl-src/       # BoringSSL 源码
    ├── zlib-src/            # zlib 源码
    ├── nginx-src/           # Nginx 源码
    ├── ngx_brotli/          # Brotli 模块
    ├── zstd-nginx-module/   # Zstandard 模块
    ├── ngx_devel_kit/       # NDK 模块
    ├── boringssl-<arch>/    # 各架构的 BoringSSL 构建目录
    ├── zlib-<arch>/         # 各架构的 zlib 构建目录
    └── nginx-build-<arch>/  # 各架构的 Nginx 构建目录
```

---

## 🛠️ 使用详解

### 编译所有架构
```bash
./nginx_build_all
```
自动编译 10 个架构，首次运行会安装依赖。

### 编译指定架构
```bash
# 单个架构
./nginx_build_arch --arch x86_64_v3

# 多个架构
./nginx_build_all --arch x86_64_v3,armv8_crc

# 跳过依赖安装（加速重复构建）
./nginx_build_arch --arch x86_64_v3 --skip-deps
```

### 自定义版本
```bash
# 通过修改配置区（推荐）
# 编辑 nginx_build_arch 第 10 行：
readonly NGINX_VERSION="1.29.3"  # 改为你需要的版本

# 或通过环境变量（不推荐，需要修改脚本支持）
```

---

## 🎯 推荐配置

### 通用部署（最大兼容性）
```bash
./nginx_build_all --arch x86_64_v1,armv8
```
- `x86_64_v1`：所有 x86_64 CPU
- `armv8`：所有 ARM64 CPU

### 现代服务器（性能优先）
```bash
./nginx_build_all --arch x86_64_v3,armv8_crc
```
- `x86_64_v3`：95% 现代 x86_64 CPU，包含 mimalloc + LTO
- `armv8_crc`：95% 现代 ARM64 CPU

### 云平台推荐

**AWS**
```bash
./nginx_build_all --arch x86_64_v3,armv8_crc,armv9
```
- x86_64_v3：Intel/AMD 实例
- armv8_crc：Graviton 1
- armv9：Graviton 3/4

**阿里云**
```bash
./nginx_build_all --arch x86_64_v3,x86_64_v4
```
- x86_64_v3：通用型
- x86_64_v4：倚天710（ARM）需要单独编译

**华为云**
```bash
./nginx_build_all --arch x86_64_v3,armv8.2
```
- x86_64_v3：Intel 实例
- armv8.2：鲲鹏 920

---

## 🔍 验证编译产物

### 查看 Nginx 版本和编译选项
```bash
./builds/1.29.3/nginx-x86_64_v3 -V
```

### 查看二进制架构信息
```bash
file ./builds/1.29.3/nginx-x86_64_v3
```

### 查看所有产物大小
```bash
ls -lh ./builds/1.29.3/
```

### 测试运行（需要 root 权限）
```bash
sudo ./builds/1.29.3/nginx-x86_64_v3 -t
```

---

## ⏱️ 编译时间

| 场景 | 耗时 | 说明 |
|------|------|------|
| 单架构首次编译 | 15-20 分钟 | 下载源码 + 编译依赖 + 编译 Nginx |
| 单架构重复编译 | 8-12 分钟 | 跳过源码下载，使用缓存 |
| 10 架构全量编译 | 150-200 分钟 | 约 2.5-3.5 小时 |

**优化点：**
- BoringSSL 和 zlib 并行构建
- 源码共享（所有架构共用同一份源码）
- 依赖库缓存（已编译的依赖库不重复编译）

---

## 📋 系统要求

### 操作系统
- **推荐**：Debian 11/12 或 Ubuntu 20.04/22.04/24.04
- **支持**：其他 Linux 发行版（需手动安装依赖）

### 硬件要求
- **CPU**：4 核及以上（推荐 8 核，编译更快）
- **内存**：8GB 及以上（推荐 16GB）
- **磁盘**：20GB 可用空间（源码 + 编译产物）

### 软件依赖
脚本会自动安装以下依赖（需 root 权限）：
- `build-essential` - GCC、G++、Make
- `cmake` - 构建 BoringSSL
- `git` - 克隆源码
- `wget`, `curl` - 下载文件
- `libpcre2-dev` - PCRE 正则表达式库
- `zlib1g-dev` - zlib 开发文件
- `libatomic-ops-dev` - 原子操作库
- `libbrotli-dev` - Brotli 开发文件
- `libzstd-dev` - Zstandard 开发文件
- `libmimalloc-dev` - mimalloc 开发文件
- `gcc-aarch64-linux-gnu` - ARM64 交叉编译工具链
- `g++-aarch64-linux-gnu` - ARM64 C++ 交叉编译

**跳过依赖安装：**
```bash
./nginx_build_arch --arch x86_64_v3 --skip-deps
```

---

## 🔧 配置管理

### 集中配置区
所有可配置项集中在 `nginx_build_arch` 脚本的顶部（第 10-95 行）：

```bash
# 版本配置
readonly NGINX_VERSION="1.29.3"
readonly BORINGSSL_REPO="https://github.com/google/boringssl"
readonly ZLIB_REPO="https://github.com/cloudflare/zlib"
readonly BROTLI_REPO="https://github.com/google/ngx_brotli"
readonly ZSTD_REPO="https://github.com/tokers/zstd-nginx-module"
readonly NDK_REPO="https://github.com/vision5/ngx_devel_kit"

# 架构配置（格式: "交叉编译前缀:CMake架构:DPKG架构:march标志"）
declare -Ar ARCH_CONFIG=(
    ["x86_64_v1"]="x86_64-linux-gnu:x86_64:amd64:-march=x86-64"
    ...
)

# Nginx 编译参数（数组形式，便于维护）
readonly NGINX_CONFIGURE_ARGS=(
    --prefix=/etc/nginx
    --sbin-path=/usr/sbin/nginx
    ...
)
```

### 修改配置
1. 编辑 `nginx_build_arch` 脚本
2. 找到配置区（第 10-95 行）
3. 修改对应的配置项
4. 重新运行编译脚本

---

## 🐛 故障排查

### 编译失败

**问题：缺少工具链**
```
错误: 缺少交叉编译工具: aarch64-linux-gnu-gcc
```
**解决：**
```bash
sudo apt install -y gcc-aarch64-linux-gnu g++-aarch64-linux-gnu
```

**问题：依赖库编译失败**
```
错误: 依赖库构建失败
```
**解决：** 查看编译日志，通常是缺少依赖或网络问题
```bash
# 清理后重试
rm -rf compile/
./nginx_build_arch --arch x86_64_v3
```

**问题：磁盘空间不足**
```
错误: No space left on device
```
**解决：** 清理不需要的文件，保留至少 20GB 空间
```bash
# 清理编译目录
rm -rf compile/

# 只保留需要的架构
rm builds/1.29.3/nginx-armv8*
```

### 二进制无法运行

**问题：CPU 不支持**
```
Illegal instruction
```
**解决：** 使用更低级别的架构
```bash
# 如果 x86_64_v3 失败，尝试 x86_64_v2 或 x86_64_v1
./nginx_build_arch --arch x86_64_v1
```

**问题：缺少共享库**
```
error while loading shared libraries: libmimalloc.so.2
```
**解决：** 安装 mimalloc 或使用 ARM 架构
```bash
sudo apt install -y libmimalloc2.0
```

---

## 📊 性能对比

### 架构性能差异（相对 x86_64_v1）

| 架构 | 相对性能 | 说明 |
|------|---------|------|
| x86_64_v1 | 100% | 基线 |
| x86_64_v2 | 105-110% | SSE4.2 优化 |
| x86_64_v3 | 115-125% | AVX2 + mimalloc + LTO |
| x86_64_v4 | 120-130% | AVX512（部分场景） |
| armv8 | 90-95% | 基线 ARM |
| armv8_crc | 95-100% | CRC32 加速 |
| armv8_crypto | 100-110% | 加密场景 |

*注：性能提升取决于具体负载和 CPU 型号*

---

## 📝 更新日志

### v1.29.3（当前版本）
- ✅ 优化代码结构，集中配置管理
- ✅ 添加并行构建支持（BoringSSL + zlib）
- ✅ 统一错误处理机制
- ✅ 改进日志输出，添加彩色提示
- ✅ 函数重命名，提高代码可读性

### 特性
- 支持 10 个架构
- 自动化程度高，一键编译
- 集成最新补丁（dynamic TLS records、OpenSSL MD5/SHA1）

---

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

### 添加新架构
1. 编辑 `nginx_build_arch` 配置区
2. 在 `TARGETS` 数组添加架构名
3. 在 `ARCH_CONFIG` 添加架构配置
4. 测试编译

### 添加新模块
1. 编辑配置区，添加模块仓库地址
2. 在 `clone_third_party_modules()` 函数添加克隆逻辑
3. 在 `NGINX_CONFIGURE_ARGS` 添加 `--add-module` 参数

---

## 📄 许可

本项目脚本采用 MIT 许可。

各依赖组件许可：
- Nginx：BSD-2-Clause
- BoringSSL：OpenSSL + ISC
- zlib：zlib License
- Brotli：MIT
- Zstandard：BSD + GPLv2

---

## 🔗 相关链接

- [Nginx 官网](https://nginx.org/)
- [BoringSSL](https://github.com/google/boringssl)
- [Cloudflare zlib](https://github.com/cloudflare/zlib)
- [ngx_brotli](https://github.com/google/ngx_brotli)
- [zstd-nginx-module](https://github.com/tokers/zstd-nginx-module)

---

**Nginx 版本：** 1.29.3  
**支持架构：** 10 个（4 x x86_64 + 6 x ARM64）  
**编译时间：** 单架构 15-20 分钟，全架构 2.5-3.5 小时  
**一键编译，全架构覆盖** 🚀
