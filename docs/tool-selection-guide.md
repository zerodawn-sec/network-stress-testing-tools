---
title: 压力测试工具选型对比
description: 主流压力测试方法在功率、协议覆盖、场景适配上的横向对比
---

# 压力测试工具选型对比

选择测试方法不是选"最强的"，而是选"最匹配目标架构的"。以下从三个维度做横向对比。

## 维度一：功率效率

功率效率 = 目标资源消耗 / 测试端资源消耗。比值越高，方法越"划算"。

| 方法类型 | 典型功率比 | 瓶颈在 |
|---------|-----------|--------|
| SYN Flood | 1:10（目标消耗10倍） | 测试端上行带宽 |
| UDP Amplification | 1:100+ | 反射器可用性 |
| HTTP GET Flood | 1:3 | 测试端连接数 |
| HTTPS Flood | 1:5 | 测试端TLS握手CPU |
| Slowloris | 1:50 | 测试端连接保持数 |

## 维度二：协议覆盖

| 目标类型 | 有效方法 | 无效方法 |
|---------|---------|---------|
| 纯TCP服务 | SYN, ATCP, PTCP | HTTP类全部 |
| Web服务器 | HTTP, HTTPS, HTTPMix | UDP类 |
| CDN防护站 | HTTPS, SlowHTTP | SYN, UDP（被边缘吸收） |
| 游戏服务器 | UDP Rand, ATCP | HTTP类 |
| API端点 | HTTPMix, POST Flood | SYN（被LB吸收） |

## 维度三：场景适配

### 高防IP目标
高防IP（如OVH、淘宝高防）的清洗能力很强。纯泛洪方法几乎无效，应该用：
- 建立真实连接的方法（ATCP/PTCP）消耗连接表
- L7方法绕过L4清洗直接到源站
- 混合方法让清洗规则难以匹配

### CDN防护目标
CDN（如Cloudflare）的核心是边缘缓存+验证。策略：
- 先确认源站IP（CDN只是代理，源站可能裸露）
- 如果源站隐藏，测试L7方法消耗CDN的TLS卸载CPU
- Slow HTTP测试CDN的连接超时

### 裸服务器目标
没有CDN/高防的裸服务器最容易测试：
- SYN Flood直接打TCP栈
- UDP Flood打带宽
- 任何L7方法直接到应用

## 选择决策树

```
目标有CDN吗？
├─ 是 → 能找到源站IP吗？
│   ├─ 是 → 直连源站测试（L4/L7均可）
│   └─ 否 → L7方法消耗CDN边缘
└─ 否 → 目标有高防IP吗？
    ├─ 是 → 真实连接方法 + L7混合
    └─ 否 → 任意方法（裸服务器）
```

---

*相关阅读: [压力测试方法速查表](https://zerodawnsec.com/ddos-stress-test-guide.html)*
