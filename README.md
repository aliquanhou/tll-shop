# TLL Shop

**TLL OS 官方第一商业 Dogfooding 项目**

High Performance E-Commerce Platform built entirely with TLL High-Frame-Rate Language.

## 项目定位

TLL Shop 不是 Demo，而是 TLL OS 的真实商业产品施工现场。

- **TLL OS** 负责造工具：Language / Compiler / High-Frame Runtime / Stdlib / Crypto / Agent Foundation
- **TLL Shop** 负责用工具造产品：完整电商系统（注册→登录→商品→SKU→购物车→地址→优惠→Checkout→支付→库存→订单→发货→售后→管理后台→数据统计）

开发过程中遇到 TLL 能力缺口时，回 TLL OS 底座修复，只做针对性验证，不重复全量回归。

## 仓库边界

### 本仓库包含（Shop 业务）
- Shop 前台
- Admin 管理后台
- RESTful API
- Product / SKU / Inventory
- Cart / Checkout / Order
- Payment / After-sales
- Marketing / CMS
- User Center（15 个服务页面）
- Shop database schema
- Shop deployment configuration
- Shop-specific tests

### 不包含（TLL OS 核心）
- TLL Language 源码
- Compiler 源码
- Runtime 源码
- Stdlib 源码
- Crypto 源码
- Agent Foundation 源码

这些通过正式 TLL Toolchain Release 引用。

## TLL 依赖版本

当前 Shop 依赖的 TLL 工具链版本：

| 组件 | 版本 | Commit | SHA256 |
|------|------|--------|--------|
| TLL Compiler | tllc-v1.0.0-rc1 | 364b9c5 | 36A5A20EB1124D2F40ED0051357D06DD976E488489DF61D652140FEFC556A845 |
| TLL Runtime (Windows) | tllvm.exe | - | - |
| TLL Runtime (Linux) | tllvm | - | - |
| Bootstrap Compiler | compiler.tllbc | - | 38825F1487D713B70793049427809F5D376F3586870C38D6CE008D07AED23677 |

详细信息见 `TLL-TOOLCHAIN-VERSION.md`。

## 已封板能力（来自 TLL OS）

- 🔒 P1-01 Secure Random（CSPRNG，三平台）
- 🔒 P1-02 Password Hashing（bcrypt）
- 🔒 P1-03 HMAC-SHA256（RFC 4231 全量）
- 🔒 P0-15.19 Ed25519（RFC 8032）
- 🔒 P0-15.20 Identity
- 🔒 P0-15.21 Capability
- 🔒 P0-15.22 Authority
- 🔒 P0-15.23 Evidence
- 🔒 P0-15.24 Trust
- 🔒 P0-15.25 Agent Ecosystem
- 🔒 P4 Compiler Release / Rollback
- 🔒 P5 Truth / Evidence Consistency

## 当前状态

### 已实现
- ✅ 完整前台商城（首页/分类/商品详情/购物车/结算/订单）
- ✅ 用户认证（注册/登录/Session/RBAC）
- ✅ 用户中心 15 个服务页面全部贯通
- ✅ 管理后台 17 个模块
- ✅ RESTful API
- ✅ SQLite 数据库
- ✅ 营销模块（优惠券/积分/会员）
- ✅ 搜索功能
- ✅ ICP 备案/协议/发票
- ✅ 远程服务器部署（shop.tllos.com）

### 待开发（P1 安全加固 + 商业功能）
- ⏳ Session ID 替换为 CSPRNG
- ⏳ CSRF 使用 CSPRNG+HMAC 重新设计
- ⏳ SQL 注入防护（参数化查询）
- ⏳ 支付重新设计（订单金额绑定/nonce/signature/幂等/回调验证）
- ⏳ 完整商业流程验证
- ⏳ 库存管理
- ⏳ 物流跟踪
- ⏳ 售后系统
- ⏳ 数据统计

## 快速开始

### 编译
```bash
# 使用 TLL Compiler 编译
tllvm tools/TLLC/tllc.tllbc compile mall/main.tll -o mall/main.tllbc
```

### 运行
```bash
# Windows
tllvm mall/main.tllbc

# Linux
./tllvm mall/main.tllbc
```

### 访问
- 前台：http://localhost:8090/
- 管理后台：http://localhost:8090/admin（admin/admin123）

## 部署

远程服务器：1.117.221.61（Ubuntu 22.04）
- 部署目录：/opt/tllos-mall/
- 服务：tllos-mall.service（systemd）
- Nginx 反向代理 + Let's Encrypt SSL
- 访问地址：https://shop.tllos.com/

详细部署文档见 `deploy/` 目录。

## Git 分支策略

与 TLL OS 一致：
- `main`：稳定、可发布、可回滚
- `develop`：施工主分支
- `feature/*`：功能分支

```
feature → develop → CI → 验收 → main
```

main 不作为施工现场。

## 工程循环

### 能力循环
```
TLL Release → Shop 实战 → 暴露缺口 → TLL 补齐 → 重新封版 → Shop 继续
```

### 工程循环
```
Code → Test → CI → Evidence → Commit → Push → Remote Verify → Truth Sync
```

## 许可证

TLL Shop 是 TLL OS 的商业 Dogfooding 项目。

## 迁移记录

本仓库从 `aliquanhou/tllos` 的 `mall/` 目录分离而来。
- 迁移基线：tllos commit c3d9696（Mall-P0-1 用户中心表补全）
- 迁移时间：2026-09-06
- 迁移原因：TLL OS 负责造工具，TLL Shop 负责用工具造产品，分离后各自独立演进
