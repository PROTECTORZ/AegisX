# RE:Aegis Bans v2.0 & Proxy Check Pro

![AMX Mod X](https://img.shields.io/badge/AMX_Mod_X-1.9+-green)
![ReAPI](https://img.shields.io/badge/ReAPI-Required-blue)
![EasyHTTP](https://img.shields.io/badge/EasyHTTP-Required-orange)
![License](https://img.shields.io/badge/License-Free-lightgrey)

## Advanced Ban Management and Network Security Suite for AMX Mod X

RE:Aegis Bans v2.0 & Proxy Check Pro is an advanced anti-evasion and network security solution for Counter-Strike 1.6 servers running AMX Mod X and ReAPI.

The project combines intelligent ban tracking, fingerprint identification, subnet analysis, VPN/Proxy detection, flood protection, and automated ban-evasion detection into a single package.

---

# Features

## RE:Aegis Bans

- Smart Ban System
- SteamID Detection
- IP Address Detection
- Client Fingerprint Tracking
- Subnet Matching (/24 and /16)
- Automatic Ban Evasion Detection
- Screenshot System
- Advanced Unban Search
- SQL Storage Support
- nVault Storage Support
- Public API for Third-Party Plugins

## Proxy Check Pro

- ProxyCheck.io Integration
- VPN Detection
- Proxy Detection
- Hosting Provider Detection
- Local Cache System
- Flood Protection
- Strict Flood Mode
- Automatic Subnet Analysis
- Cached Network Reputation Checks

---

# Requirements

## Required

- AMX Mod X 1.9+
- ReAPI
- ReHLDS
- EasyHTTP Module

## Optional

- SQLx
- MySQL
- nVault

---

# Installation

## 1. Install the Plugins

Copy the compiled plugins into:

```text
addons/amxmodx/plugins/
```

Add them to:

```text
addons/amxmodx/configs/plugins.ini
```

Example:

```ini
re_aegis_bans.amxx
proxy_check_pro.amxx
```

---

## 2. Install EasyHTTP

Copy the EasyHTTP module to:

```text
addons/amxmodx/modules/
```

Enable it inside:

```text
addons/amxmodx/configs/modules.ini
```

```ini
easy_http
```

---

## 3. Configure ProxyCheck API

Set your ProxyCheck.io API key:

```pawn
new const g_szKey[] = "YOUR_PROXYCHECK_API_KEY";
```

---

# Smart Ban Score System

When a player connects, RE:Aegis calculates a match score based on known identifiers.

| Identifier | Score |
|------------|--------|
| SteamID | 3 |
| IP Address | 3 |
| Fingerprint | 2 |
| Subnet /24 | 1 |
| Subnet /16 | 1 |

## Default Threshold

```cfg
ab_score_autoban "3"
```

### Examples

**SteamID Match**

```text
Score = 3
```

Result:

```text
Automatic Ban
```

**Fingerprint + Subnet Match**

```text
2 + 1 = 3
```

Result:

```text
Automatic Ban
```

---

# Fingerprint System

The fingerprint is generated using stable client cvars:

```text
rate
cl_updaterate
cl_cmdrate
cl_dlmax
cl_lc
cl_lw
cl_dlfile
_vgui_menus
lefthand
```

The collected values are hashed using MD5 to create a persistent client fingerprint.

This allows the system to identify players even after changing:

- SteamID
- IP Address
- VPN Endpoint

---

# Configuration

## RE:Aegis Bans

```cfg
ab_storage_type "0"

ab_fp_enabled "1"

ab_score_auth "3"
ab_score_ip "3"
ab_score_fp "2"

ab_score_subnet_mini "1"
ab_score_subnet_full "1"

ab_score_autoban "3"
ab_score_logonly "1"

ab_fp_ban_minutes "0"
ab_subnet_ban_minutes "10080"

ab_pwn_enabled "1"
```

## Proxy Check Pro

```cfg
pcp_enabled "1"

pcp_cache_days "30"

pcp_flood_threshold "15"

pcp_subnet_limit "5"

pcp_strict_flood "1"
```

---

# Administrative Commands

## Ban Management

```text
amx_ban <target> <minutes> <reason>
amx_banip <target> <minutes> <reason>
amx_banid <target> <minutes> <reason>

amx_addban <target> <minutes> <reason>

amx_unban <search>

amx_pwn <target> <minutes>

amx_ss <target>

amx_checkfp <target>
```

## Proxy Check Pro

```text
amx_pcp_status

amx_pcp_check <ip>

amx_pcp_flush
```

---

# SQL Installation

If SQL storage is enabled:

```cfg
ab_storage_type "1"
```

Import the provided `install.sql` file before starting the server.

---

# Public API

## Natives

```pawn
native re_aegis_ban_player(...);
native re_aegis_unban_player(...);
native re_aegis_screenshot(...);
native re_aegis_get_fingerprint(...);
native re_aegis_get_match_score(...);
```

## Forward

```pawn
forward re_aegis_suspicious_match(id, score, autoBanned);
```

Triggered whenever a suspicious match is detected.

---

# License

Free for personal and commercial server use.

## Credits

Dynamic Defense Project

RE:Aegis Bans v2.0 & Proxy Check Pro
