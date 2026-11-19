# qx-resource-parser (Hysteria2 补丁版)

基于 **KOP-XIAO/QuantumultX** 的 `Scripts/resource-parser.js`（上游 ©𝐒𝐡𝐚𝐰𝐧，最新基线 commit `cce2996b`，2026-11-03），
**完整复刻**上游逻辑，仅追加 **Hysteria 2** 支持（Quantumult X 1.9.0 build 965+）。

## 改了什么（只加不改）

| 位置 | 改动 |
|---|---|
| 新增 `HY2_Build() / HY2_OK()` | 从 `$environment.version` 解析 build 号，`>=965` 或解析不到时放行 hy2 |
| 新增 `CHY2_2QX()` | Clash / Clash.Meta 的 `type: hysteria2` 节点 → QX 行 |
| 新增 `HY2_URI2QX()` | `hysteria2://` / `hy2://` URI → QX 行 |
| `Subs2QX()` URI 分支 | 新增 hy2 分支（排在"不支持"分支之前） |
| `Clash2QX()` | 新增 `typecc == "hysteria2" / "hy2"` 分支 |
| `isQuanX()` / 两处 `QuanXK` | 类型白名单补 `hysteria2`，否则转换结果会在收尾阶段被丢掉 |
| `XUDP()` / `QX_TLS()` | 类型正则补 `hysteria2`，`udp=1` 时不会误写成 `udp-relay=false` |
| `TagCheck_QX()` typefix | 补 `hysteria2` 条目（空 tag 兜底） |

其余类型（ss/ssr/vmess/trojan/vless/anytls/http/socks5…）的转换结果与上游**逐字节一致**（已用 Node 打桩对比验证）。

## 字段映射

Clash → QX：

- `server:port` → `hysteria2=host:port`
- `password` → `password=`
- `ports` → `port-hopping=`
- `obfs: salamander` + `obfs-password` → `salamander-password=`
- `sni` → `tls-host=`
- `skip-cert-verify: true` → `tls-verification=false`
- 固定 `udp-relay=true`；`name` → `tag=`

URI 参数：`mport`/`ports` → `port-hopping`，`obfs=salamander&obfs-password=` → `salamander-password`，
`sni`/`peer` → `tls-host`，`insecure=1` → `tls-verification=false`，`#fragment` → `tag`。

## 在 QX 里使用

资源解析器地址填：

```
https://raw.githubusercontent.com/Cornelius798/qx-resource-parser/main/resource-parser.js
```

参数用法与上游完全相同（`#emoji=1&udp=1&in=...` 等）。

## 已知（上游遗留，未改）

- 上游 `version` 变量是 `Number($environment.version.split("build")[1])`，QX 实际返回形如 `1.9.0 (build 965)` 时结果是 `NaN`，
  导致 `Clash2QX()` 里 `anytls && version > 913` 永远不成立 → **Clash 源里的 anytls 节点被丢弃**。本仓库保持原样（复刻要求）。
- `tuic` / `wireguard` 仍然走"不支持"提示。
