# qx-resource-parser (Hysteria2 + Mieru 补丁版)

基于 **KOP-XIAO/QuantumultX** 的 `Scripts/resource-parser.js`（上游 ©𝐒𝐡𝐚𝐰𝐧，最新基线 commit `cce2996b`，2026-11-03），
**完整复刻**上游逻辑，仅追加 **Hysteria 2**（Quantumult X 1.9.0 build 965+）与 **Mieru**（Quantumult X 1.9.0 build 968+）支持。

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
| HY2 三字段补齐（2026-11-22） | `port-hopping-interval` / `download-bandwidth` / `tls-cert-sha256`：Clash 的 `hop-interval` / `down` / `fingerprint` 与 URI 对应参数 → QX 行（上游字段移植，门槛保持自家正则；非法值只丢字段、不跳节点） |
| 新增 `MIERU_Build() / MIERU_OK()` | 从 `$environment.version` 解析 build 号，`>=968` 或解析不到时放行 mieru |
| 新增 `CMIERU_2QX()` | Clash / Clash.Meta 的 `type: mieru` 节点 → QX 行 |
| 新增 `MIERU_URI2QX()` | `mierus://` 明文分享链接 → QX 行（多组 `port`/`protocol` 只取第一组） |
| `Subs2QX()` URI 分支 | 新增 mierus 分支（排在"不支持"分支之前）；`mieru://` base64 形式走"不支持"提示 |
| `Clash2QX()` | 新增 `typecc == "mieru"` 分支 |
| `isQuanX()` / 两处 `QuanXK` | 类型白名单补 `mieru`，否则转换结果会在收尾阶段被丢掉 |
| `XUDP()` / `QX_TLS()` | 类型正则补 `mieru`，`udp=1` 时不会误写成 `udp-relay=false` |
| `TagCheck_QX()` typefix | 补 `mieru` 条目（空 tag 兜底） |

其余类型（ss/ssr/vmess/trojan/vless/anytls/http/socks5…）的转换结果与上游**逐字节一致**（已用 Node 打桩对比验证）。

## 字段映射

Clash → QX：

- `server:port` → `hysteria2=host:port`
- `password` → `password=`
- `ports` → `port-hopping=`
- `obfs: salamander` + `obfs-password` → `salamander-password=`
- `sni` → `tls-host=`
- `skip-cert-verify: true` → `tls-verification=false`
- `hop-interval` → `port-hopping-interval=`（仅当 `ports` 存在时写出；不校验 5–3600 范围）
- `down` → `download-bandwidth=`（Mbps 数字：`100 Mbps`→100，`1 Gbps`→1000，`10MBps`→80；解析不出丢弃）
- `fingerprint` → `tls-cert-sha256=`（64 位 hex，冒号分隔的也认，非法丢弃；`skip-cert-verify: true` 时丢弃）
- 固定 `udp-relay=true`；`name` → `tag=`

URI 参数：`port-hopping`/`mport`/`ports` → `port-hopping`，`obfs=salamander&obfs-password=` → `salamander-password`，
`sni`/`peer` → `tls-host`，`insecure=1` → `tls-verification=false`，`#fragment` → `tag`。
`port-hopping-interval`/`hop-interval` → `port-hopping-interval=`（同上），`download-bandwidth`/`down` → `download-bandwidth=`，
`pinSHA256` → `tls-cert-sha256=`（`insecure=1` 时丢弃）。

Mieru，Clash → QX：

- `server:port` → `mieru=host:port`；`port-range` 取首端口（如 `2090-2099` → `2090`）
- `username` → `username=`，`password` → `password=`
- `transport: UDP` 或 `udp: true` → 追加 `udp-relay=true, udp-over-tcp=true`；TCP（默认）不写 udp 字段
- `multiplexing` / `handshake-mode` / `traffic-pattern` / `mtu` / `dialer-proxy` 无 QX 对应字段，直接丢弃
- `name` → `tag=`

Mieru，`mierus://` 链接参数：`port`（可重复，取第一个；范围取首端口）与 `protocol`（可重复，取第一个；`UDP` → 追加两条 udp 字段），
`profile` / `mtu` / `multiplexing` / `handshake-mode` / `traffic-pattern` 丢弃，`#fragment` → `tag`。
`mieru://` 是 base64 整包形式，无法解析，暂不支持。

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
- Mieru 的 TLS / REALITY 暂不由转换生成：Clash 与 `mierus://` 均无对应字段；QX 侧按惯例用 `tls-host` /
  `tls-verification` / `reality-base64-pubkey` / `reality-hex-shortid` 追加，字段名以 App 内配置样本为准。
- `mierus://` 一条链接可带多组 `port`/`protocol`（对应多个服务端点），本补丁只取第一组，其余丢弃。
