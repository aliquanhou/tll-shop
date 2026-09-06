# TLL Toolchain Version Manifest

TLL Shop 依赖的 TLL OS 工具链版本记录。

## 当前版本

### TLL Compiler
- **版本**: tllc-v1.0.0-rc1
- **源码 Commit**: 364b9c5（P1-04 HTTP Client，添加 builtin idx 200-209 映射）
- **Artifact**: tools/TLLC/tllc.tllbc
- **大小**: 665,681 bytes
- **SHA256**: 36A5A20EB1124D2F40ED0051357D06DD976E488489DF61D652140FEFC556A845
- **Functions**: 172
- **Constants**: 3911

### Bootstrap Compiler (Rescue)
- **Artifact**: compiler/compiler.tllbc
- **大小**: 592,514 bytes
- **SHA256**: 38825F1487D713B70793049427809F5D376F3586870C38D6CE008D07AED23677

### TLL Runtime
- **Windows**: host/c/tllvm.exe（MSVC 编译）
- **Linux**: host/c/tllvm（GCC 编译）
- **用法**: `tllvm <file.tllbc>`

### TLL OS 仓库
- **仓库**: https://github.com/aliquanhou/tllos
- **main 分支**: 6875401（P2 清理完成，稳定基线）
- **develop 分支**: 880115c（移除 Shop 业务代码，仓库边界清理）

## CI 固定 Toolchain（P0.1 Toolchain Dependency Pinning）

Shop CI 必须使用固定的 TLL OS Toolchain，禁止随 develop 自动漂移。

### CI 配置
- **CI 文件**: `.github/workflows/ci.yml`
- **TLLOS_REPO**: aliquanhou/tllos
- **TLLOS_REF**: 6875401（固定 main 稳定基线）
- **TLLOS_COMMIT**: 6875401
- **TLLC_PATH**: tools/TLLC/tllc.tllbc

### SHA256 验证
CI 构建时必须验证：
1. TLL OS commit == 6875401
2. Compiler SHA256 == 36A5A20EB1124D2F40ED0051357D06DD976E488489DF61D652140FEFC556A845
3. Runtime 构建成功
4. 工具链缺失时 CI FAIL（禁止静默跳过）

### 可重复构建保证
```
TLL Shop Commit
    ↓
固定 TLL OS Commit (6875401)
    ↓
固定 Compiler SHA256 (36A5A20E...)
    ↓
可重复构建
```

### 升级 Toolchain 流程
当需要升级 TLL Toolchain 时：
1. 在 TLL OS 仓库完成新能力开发和封板
2. 更新本文件的 TLLOS_COMMIT 和 COMPILER_SHA256
3. 更新 `.github/workflows/ci.yml` 中的 TLLOS_REF
4. 在 develop 分支测试 CI 通过
5. 验收后合并 main
6. 记录升级原因和影响

## 已封板能力

### 密码学
- 🔒 P1-01 Secure Random（CSPRNG，Windows BCryptGenRandom / Linux getrandom() / macOS /dev/urandom）
- 🔒 P1-02 Password Hashing（bcrypt，OpenBSD 公共领域代码）
- 🔒 P1-03 HMAC-SHA256（RFC 4231 全量 7 个测试用例 + TC5 full/truncated）
- 🔒 P0-15.19 Ed25519（RFC 8032 测试向量）

### Agent Foundation
- 🔒 P0-15.20 Identity（8/8）
- 🔒 P0-15.21 Capability（12/12）
- 🔒 P0-15.22 Authority（12/12）
- 🔒 P0-15.23 Evidence（12/12）
- 🔒 P0-15.24 Trust（14/14）
- 🔒 P0-15.25 Agent Ecosystem（18/18）

### 工程基础设施
- 🔒 P4 Compiler Release / Rollback（确定性 Bootstrap 2/2，Rescue Compiler）
- 🔒 P5 Truth / Evidence Consistency（45 项能力，11 SEALED）

### 语言能力
- 🔒 `\xNN` 十六进制字节转义（P1-03-R1 添加）
- 🔒 Map 字面量 `{}` 和 `.key = value` 语法
- 🔒 SQLite C 绑定（idx 150-159）
- 🔒 HTTP Client（idx 200-209，Linux/Windows 完整，macOS AVAILABLE/KNOWN ISSUE）

## 编译命令

### Windows
```cmd
host\c\tllvm.exe tools\TLLC\tllc.tllbc compile mall\main.tll -o mall\main.tllbc
```

### Linux/macOS
```bash
./host/c/tllvm tools/TLLC/tllc.tllbc compile mall/main.tll -o mall/main.tllbc
```

## 版本升级策略

当 TLL OS 发布新版本时：
1. 记录新版本的 Compiler SHA256 和 Runtime 版本
2. 在 tll-shop 中测试新版本兼容性
3. 更新本文件
4. 提交并推送
5. 如果新版本引入 Breaking Change，先在 develop 分支测试，验收后再合并 main

## 依赖方向

```
TLL OS (Language/Compiler/Runtime/Stdlib/Crypto/Agent)
    ↓ 正式 Release
TLL Shop (商业应用)
```

依赖方向保持单向：TLL Shop 依赖 TLL OS，TLL OS 不依赖 TLL Shop。

## 迁移记录

- **2026-09-06**: 从 aliquanhou/tllos 仓库分离，建立独立 tll-shop 仓库
- **迁移基线**: tllos develop commit c3d9696
- **迁移原因**: TLL OS 负责造工具，TLL Shop 负责用工具造产品
