# Blue Knight Gate: 

A VLESS / VMess / Trojan / Shadowsocks / XHTTP panel that runs on **Free Hosts** (Pterodactyl) using the **one port** your server is given.

## Follow Blue Knight
- 📢 Telegram: [@BlueKnight_Net](https://t.me/BlueKnight_Net)
- ▶️ YouTube: [@BlueKnight-Net](https://www.youtube.com/@BlueKnight-Net)


## Setup on Free Hosts
1. Create a **Node.js** server (Node 18 or newer, 20 recommended).
2. In **File Manager**, upload `index.js`, `package.json`, `package-lock.json` and the `assets/` folder. Don't upload `node_modules`; it installs itself.
3. Startup file: `index.js` (the start command is `node index.js`).
4. Start the server. The panel reads your port from `SERVER_PORT` automatically; you don't need to set it.
5. On first start, the panel downloads **sing-box** and **Xray-core** from GitHub, so give it a minute.
6. Open `http://<server-ip>:<port>/knight` and create your panel password.
7. Copy the configs or the subscription link into your client.

**Keep the `bk-data/` folder.** It holds your password, UUID and the TLS certificate. If you delete it, every config and TLS pin changes and you'll have to re-import everything.

## Configs (all on the same single port)
| Config | Works in |
|---|---|
| VLESS-WS / VLESS-WS-TLS | all clients |
| VMess-WS / VMess-WS-TLS | all clients |
| Trojan-WS / Trojan-WS-TLS | all clients (Clash: use only Trojan-WS-TLS) |
| VLESS / VMess / Trojan HTTPUpgrade | Xray, sing-box (Clash: VLESS and VMess only) |
| VLESS-gRPC / VLESS-gRPC-TLS | Xray, sing-box, Clash Meta |
| Shadowsocks-WS | Xray, sing-box, Clash Meta (v2ray-plugin websocket) |
| VLESS-XHTTP | **Xray clients only** (v2rayN, v2rayNG, Hiddify with the Xray core) |

Every config is named `💞t.me/BlueKnight_Net - <type>` and is included in:
- the normal base64 subscription
- the "full" subscription
- the Clash file
- the sing-box file

## TLS notes
- TLS configs use a self-signed certificate.
- Links carry `allowInsecure=1` for older clients and `pcs=<sha256>` (a certificate pin) for Xray 26.2+.
- The certificate renews itself before it expires. When that happens, re-import the subscription.

## Settings (Startup → Variables, or `.env`)
All are optional.

| Variable | Default | Meaning |
|---|---|---|
| `UUID` | random, saved in `bk-data` | Fixed UUID for all configs |
| `PANEL_PASSWORD` | – | Fixed panel password (8+ characters) |
| `SUB_TOKEN` | random | Fixed subscription token |
| `SETUP_KEY` | – | Extra key required the first time you create the password |
| `DOMAIN` | server IP | Domain or host used in links |
| `ENABLE_TROJAN_WS_TLS` | `true` | Trojan-WS-TLS |
| `ENABLE_VMESS_HTTPUPGRADE` / `ENABLE_TROJAN_HTTPUPGRADE` | `true` | HTTPUpgrade configs |
| `ENABLE_VLESS_GRPC` / `ENABLE_VLESS_GRPC_TLS` | `true` | gRPC configs |
| `ENABLE_SS_WS` | `true` | Shadowsocks-WS |
| `ENABLE_XHTTP` | `true` | VLESS-XHTTP |
| `SS_METHOD` / `SS_PASSWORD` | `aes-256-gcm` / random | Shadowsocks cipher and password. For 2022 ciphers, `SS_PASSWORD` must be a base64 key. |
| `LINK_ALLOW_INSECURE` | `true` | `false` removes `allowInsecure=1` from links |

Turn a config off by setting its variable to `false`, then restart the server.

## Troubleshooting
- **XHTTP or Shadowsocks links are missing:** Xray failed to download. Check the console, then restart.
- **TLS config won't connect in a new Xray app:** re-import the subscription so the link has the current `pcs=` pin.
- **Configs changed after a restart:** `bk-data/` was deleted or reset. Set `UUID`, `PANEL_PASSWORD` and `SUB_TOKEN` as variables so they stay the same.
- **Testing:** use **Real delay** or the URL test, not TCP ping.
- **Keep the subscription URL private.** It contains your token.

---
Made by **Blue Knight** · [Telegram](https://t.me/BlueKnight_Net) · [YouTube](https://www.youtube.com/@BlueKnight-Net)

## Users (quota / expiry / device limit) - added
Open `http://<server-ip>:<port>/knight/panel` (PasarGuard-style dashboard: stats, users, settings) or tap the **Users** button on the panel. The simpler list is still at `/knight/users`.

| Field | Meaning |
|---|---|
| Days | Subscription length. `0` = unlimited. Optional: start counting at the user's first connection. |
| Volume (GB) | Upload + download. When it is used up the user's connections are cut and new ones are refused. `0` = unlimited. |
| Devices | Max number of different client IPs active at the same time (an IP counts while it moved traffic in the last 2 minutes). `0` = unlimited. |

Each user gets a private subscription link: `/knight/u/<token>` (base64), `/raw`, `/clash`, `/sb`, and a status page `/info`.
The subscription header carries the real used / total / expiry so v2rayNG, Hiddify, etc. show them.

**How it works:** every user config has a secret token inside its path (`.../ws/<token>`). The panel reads the token, counts the bytes
of that user, applies the limits, and removes the token before handing the connection to sing-box / Xray. Users live in `bk-data/users.json`.

**Strict mode** (switch on the Users page, or `BK_STRICT_USERS=true`): only connections that carry a valid user token are accepted.
Turn it ON when you sell/share configs, otherwise anyone can delete the token from the path and use the old shared link without limits.
In strict mode gRPC and other token-less protocols are closed.
User links contain only WS / WS-TLS / HTTPUpgrade / XHTTP / Shadowsocks-WS configs (gRPC cannot carry the token).

## PasarGuard-style panel (`/knight/panel`)
Pages: **Dashboard**, **Users**, **Hosts**, **Settings**.

**Hosts** work like PasarGuard hosts: every host is one config in the user's subscription and its **Remark is the config name**.
Pick protocol (VLESS / VMess / Trojan / Shadowsocks), transport (WS / HTTPUpgrade / XHTTP), security (default / none / TLS),
address, port, SNI, Host header, fingerprint, ALPN, allow-insecure, priority and which user statuses can see it.
Address / SNI / Host accept a comma list (one is picked at random per request) and `*` (random salt).
Remarks and addresses accept PasarGuard variables: `{USERNAME} {PROTOCOL} {TRANSPORT} {DATA_USAGE} {DATA_LIMIT} {DATA_LEFT}
{DAYS_LEFT} {EXPIRE_DATE} {JALALI_EXPIRE_DATE} {TIME_LEFT} {STATUS_EMOJI} {USAGE_PERCENTAGE} {SERVER_IP}`.
With no hosts, users get the default configs; once you add a host, subscriptions contain only your hosts.
`Port` only changes the port written into the link (the server still listens on the single port your hosting gives you).
