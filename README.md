# Nginx 多架构编译系统

编译优化的 Nginx 二进制文件，支持 x86_64 和 ARM64 架构。

## 快速开始

```bash
# x86_64 架构
./nginx_build_x86_64 --arch x86_64_v3

# ARM64 架构（交叉编译）
./nginx_build_arm64 --arch armv8_crc

# 列出支持的目标
./nginx_build_x86_64 --list-targets
./nginx_build_arm64 --list-targets
```

## 支持的架构

### x86_64
| 架构 | 特性 | 适用场景 |
|------|------|----------|
| `x86_64_v1` | 基线 x86-64 | 最大兼容性 |
| `x86_64_v2` | SSE4.2 | 2008年后 CPU |
| `x86_64_v3` | AVX2 + FMA | **推荐**，2013年后 CPU |
| `x86_64_v4` | AVX512 | 高端服务器 |

### ARM64
| 架构 | 特性 | 适用场景 |
|------|------|----------|
| `armv8` | 基线 ARMv8 | 最大兼容性 |
| `armv8_crc` | ARMv8 + CRC32 | **推荐**，树莓派4，Graviton |
| `armv8_crypto` | ARMv8 + Crypto | 加密优化 |
| `armv8.2` | ARMv8.2-A | Graviton2，鲲鹏920 |
| `armv8.4` | ARMv8.4-A | 现代 ARM 服务器 |
| `armv9` | ARMv9-A | Graviton3/4 |

## 编译特性

- **Nginx 1.29.3** + BoringSSL（HTTP/3, TLS 1.3）
- **压缩**：Brotli, Zstandard, Cloudflare zlib
- **安全**：Stack Protector, RELRO, FORTIFY_SOURCE
- **性能**：mimalloc + LTO（仅 x86_64）

## 项目结构

```
nginx/
├── nginx_build_x86_64    # x86_64 编译脚本
├── nginx_build_arm64     # ARM64 交叉编译脚本
├── builds/               # 编译产物
│   └── 1.29.3/
│       ├── nginx-x86_64_v1
│       ├── nginx-x86_64_v3
│       ├── nginx-armv8_crc
│       └── ...
└── compile/              # 编译工作目录（自动创建）
```

## 使用选项

```bash
./nginx_build_x86_64 --arch <target> [options]
./nginx_build_arm64 --arch <target> [options]

Options:
  -a, --arch <target>     目标架构（必需）
      --skip-deps         跳过依赖安装
      --list-targets      列出支持的架构
  -h, --help              显示帮助
```

## 验证产物

```bash
# 查看版本和编译选项
./builds/1.29.3/nginx-x86_64_v3 -V

# 查看二进制信息
file ./builds/1.29.3/nginx-x86_64_v3

# 测试配置
sudo ./builds/1.29.3/nginx-x86_64_v3 -t
```

## 系统要求

- Debian/Ubuntu（推荐）
- 8GB+ 内存，20GB+ 磁盘空间
- ARM64 编译需要交叉编译工具链：`gcc-aarch64-linux-gnu`

## 许可

脚本：MIT | Nginx：BSD-2-Clause | BoringSSL：OpenSSL+ISC
