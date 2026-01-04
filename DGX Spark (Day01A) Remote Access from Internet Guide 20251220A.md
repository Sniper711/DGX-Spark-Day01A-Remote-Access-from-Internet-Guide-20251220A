# DGX Spark (Day01A) Remote Access from Internet Guide 20251220A
## 🟩 English
> ## Scenarios & Advantages
> **From an external network on Mac/PC → via WireGuard VPN → SSH login to DGX Spark at home**
> - **Use WireGuard VPN**
>   - Use DGX Spark as VPN Server. (Mac/PC = Client)
>   - The VPN penetration rate is extremely high, and using a mobile hotspot is rarely blocked by carriers.
>   - Configuring WireGuard with UDP port 51820 along with keepalive is the right move.
>   - 90% of mobile networks in Taiwan allow WireGuard, but OpenVPN does not.
> - **Do not use Tunnelblick / OpenVPN, no need for an expensive router's built-in VPN.**
>   - Reason 1 for VPN connection failure when using mobile hotspots: carrier intentionally block mobile network UDP/TCP 1194 Ports VPN traffic.
>     - Using TCP port 443 can enhance VPN penetration rates, which is relatively stable but slower, and may cause TCP-over-TCP head-of-line blocking leading to meltdown issues on mobile networks.
>     - Some expensive routers cannot change TCP port# which is also a problem (only TCP port 1194 is available).
>   - Reason 2 for VPN connection failure when using mobile hotspots: mobile network NAT causes UDP packet loss.
>     - Although Tunnelblick and OpenVPN use UDP, their internal implementation is similar to TCP and SSL/TLS, with many steps, making it prone to disconnections on mobile networks.
>   - Therefore, there is no need for Tunnelblick and OpenVPN
> - **Use a low-end Router**
>   - Router: must have a fixed Public IP (x.x.x.x) and support Port Forwarding.
>   - Since Tunnelblick and OpenVPN are not used, the Router does not need advanced VPN features (if available, disable them), just a cheap Router is sufficient.
> - Simple one-line **SSH** command **login to DGX Spark**
>   - After rebooting, simply have the Mac/PC Client run this SHH command in step 9.1 - it's super easy.


<img width="617" height="508" alt="Day01A" src="https://github.com/user-attachments/assets/a9631407-221e-4316-9cea-fa9c1415cf4e" />


---


## Table of Contents
- [1. Router setup (required)](#1-router-setup-required)
  - [1.1 Confirm the Network Topology](#11-confirm-the-network-topology)
  - [1.2 Confirm that Router VPN is disabled](#12-confirm-that-router-vpn-is-disabled)
  - [1.3 Configure Port Forwarding](#13-configure-port-forwarding)

- [2. DGX Spark Server: confirm NIC name and install WireGuard](#2-dgx-spark-server-confirm-nic-name-and-install-wireguard)
  - [2.1 Confirm the physical NIC name (critical)](#21-confirm-the-physical-nic-name-critical)
  - [2.2 Install WireGuard VPN](#22-install-wireguard-vpn)

- [3. DGX Spark Server: create WireGuard directory and keys](#3-dgx-spark-server-create-wireguard-directory-and-keys)
  - [3.0 Enter a root shell (Very Important)](#30-enter-a-root-shell-very-important)
  - [3.1 Create directory](#31-create-directory)
  - [3.2 Generate keys](#32-generate-keys)
  - [3.3 Verify keys](#33-verify-keys)

- [4. DGX Spark Server: create WireGuard server config](#4-dgx-spark-server-create-wireguard-server-config)
  - [4.0 Notes on paths and names](#40-notes-on-paths-and-names)
  - [4.1 Create `wg-dgx-spark.conf`](#41-create-wg-dgx-sparkconf)
  - [4.2 Edit `wg-dgx-spark.conf` (copy/paste)](#42-edit-wg-dgx-sparkconf-copypaste)
  - [4.3 Key mapping quick reference](#43-key-mapping-quick-reference)

- [5. DGX Spark Server: enable IPv4 forwarding (required, once)](#5-dgx-spark-server-enable-ipv4-forwarding-required-once)
  - [5.1 Take effect immediately](#51-take-effect-immediately)
  - [5.2 Make it permanent](#52-make-it-permanent)
  - [5.3 Verify it](#53-verify-it)

- [6. DGX Spark Server: start WireGuard](#6-dgx-spark-server-start-wireguard)
  - [6.1 Check systemd (system daemon) status](#61-check-systemd-system-daemon-status)
  - [6.2 Check WireGuard status](#62-check-wireguard-status)

- [7. Mac/PC Client: create WireGuard client config](#7-macpc-client-create-wireguard-client-config)
  - [7.0 Notes on paths and names](#70-notes-on-paths-and-names)
  - [7.1 Create `wg-mac-pc.conf`](#71-create-wg-mac-pcconf)
  - [7.2 Edit `wg-mac-pc.conf` (copy/paste)](#72-edit-wg-mac-pcconf-copypaste)
  - [7.3 Key mapping quick reference](#73-key-mapping-quick-reference)

- [8. Mac/PC Client: VPN back home (DGX Spark Server) from an external network](#8-macpc-client-vpn-back-home-dgx-spark-server-from-an-external-network)
  - [8.1 Install WireGuard VPN](#81-install-wireguard-vpn)
  - [8.2 Import WireGuard client configuration file, and activate VPN (must use external network)](#82-import-wireguard-client-configuration-file-and-activate-vpn-must-use-external-network)
  - [8.3 Three ways to validate the `VPN tunnel` (must use external network)](#83-three-ways-to-validate-the-vpn-tunnel-must-use-external-network)

- [9. Login and Command the DGX Spark Server](#9-login-and-command-the-dgx-spark-server)
  - [9.1 SHH Login to the DGX Spark Server from your Mac/PC Client](#91-shh-login-to-the-dgx-spark-server-from-your-macpc-client)
  - [9.2 Two Example Methods to Command the DGX Spark Server from your Mac/PC Client](#92-two-example-methods-to-command-the-dgx-spark-server-from-your-macpc-client)

---

## 1. Router setup (required)

### 1.1 Confirm the Network Topology

- Verify that **exactly one router** is deployed in front of DGX Spark:
  - This router connects directly to the upstream **modem**.
  - This router logs in to the ISP using **PPPoE** and must have a **fixed Public IP (x.x.x.x)**.
- There should be **no other routers** in front of DGX Spark (if there are, set them to **AP Mode** and use them as **switches**).
  - This way, no matter where DGX Spark is connected, it is effectively behind the only Router.
- Networks at the same layer as DGX Spark and further downstream may have other routers.

### 1.2 Confirm that Router VPN is disabled
- Log in to the router.
- **Disable** VPN Server and VPN Client (all off).
- Do not retain any accounts or configurations.

### 1.3 Configure Port Forwarding

**Find DGX Spark’s LAN IP**
- Log in to the router settings page, locate the intranet IP address of device spark-xxxx (192.168.x.x), and make a note of it.

**PPPoE login ISP to obtain the fixed Public IP**  
(Usually under router settings page: `Main Page` → `WAN` → `WAN Setting`)
- WAN Connection Type: **PPPoE**
- PPPoE Setting:
  - Address Mode = **Dynamic IP**
  - User Name = ISP-provided login for the fixed Public IP
  - Password = ISP-provided password for the fixed Public IP
  - Operation Mode = **Keep Alive**
- Save

**Configure Port Forwarding**  
(Usually under router settings `Main Page` → `WAN` → `Port Forwarding`)
| Field | Value |
|---|---|
| Rule Name | wireguard |
| Protocal | UDP |
| Public Port | 51820 |
| Private Port | 51820 |
| Private IP | DGX Spark LAN IP (enter the `192.168.x.x` value) |
| Inbound Filter | Allow All |
| Enabled | Yes |

**How it works**
- The router forwards external `UDP:51820`
- To DGX Spark `UDP:51820`
- Which must match `ListenPort = 51820`

**Quick self-check**
- Port Forwarding is enabled
- No other VPN service is occupying that port
- Public/Private ports match

---

## 2. DGX Spark Server: confirm NIC name and install WireGuard

### 2.1 Confirm the physical NIC name (critical)
NIC stands for Network Interface Card. Use this command
```
ip link
```

to check the DGX Spark physical NIC name (consist of alphanumber characters), and note it. 
```
exxxxx (for example)
```

- The commands below use this `exxxxx` example when building the WireGuard Server config.
  - If you change hardware, you must update the commands accordingly.
  - Example: buying a second DGX Spark may yield a different NIC name; you must confirm again.

### 2.2 Install WireGuard VPN
```
sudo apt update
sudo apt install -y wireguard
```

Verify version (required)
```
wg --version
```

---

## 3. DGX Spark Server: create WireGuard directory and keys

### 3.0 Enter a root shell (Very Important)
```
sudo -i
```

You should see a prompt similar to:
```
root@spark-xxxx:~#
```

**Why you must use `sudo -i` first**
- `/etc` is a root-only writable directory; `cd` does not inherit sudo
- `umask` is a shell built-in; you cannot run `sudo umask`
- Avoid two common errors:
  - `cd: Permission denied`
  - `sudo: umask: command not found`

### 3.1 Create directory
Notes
- Use the path:
  - `/etc/wireguard`
  - The commands below assume this exact path. If you change it, update the commands accordingly.
```
mkdir -p /etc/wireguard
cd /etc/wireguard
umask 077
```

### 3.2 Generate keys  
(still inside the root shell with a prompt like `root@spark-xxxx:~#`)
```
wg genkey | tee server_private.key | wg pubkey > server_public.key
wg genkey | tee client_private.key | wg pubkey > client_public.key
```

Why this is the “correct and only recommended” way
- `tee private.key`: securely saves the private key without exposing it in your shell history (terminal).
- `> public.key`: avoids misusing `tee` and causing overwrite errors.

### 3.3 Verify keys
```
ls -l /etc/wireguard/
```

You should see
```
server_private.key
server_public.key
client_private.key
client_public.key
```

---

## 4. DGX Spark Server: create WireGuard server config

### 4.0 Notes on paths and names
- Path:
  - `/etc/wireguard`
  - The commands below assume this exact path. If you change it, update the commands accordingly.
- File name:
  - `wg-dgx-spark.conf`
  - The commands below assume this exact file name. If you change it, update the commands accordingly.

### 4.1 Create `wg-dgx-spark.conf`
- Under `/etc/wireguard` on the DGX Spark Server, create file:
```
nano /etc/wireguard/wg-dgx-spark.conf
```

### 4.2 Edit `wg-dgx-spark.conf` (copy/paste)
```
[Interface]
# WireGuard Server internal IP (VPN subnet)
Address = 10.100.0.1/24

# WireGuard listening port (must match the router's Port Forward setting)
ListenPort = 51820

# Server PrivateKey
# How to query (all Keys can only be queried using commands on DGX Spark):
#   sudo cat /etc/wireguard/server_private.key
# Remember: Server PrivateKey must be stored on DGX Spark (Server)
#
# Remove `<server_private_key>`, and replace it with the Server PrivateKey value
PrivateKey = <server_private_key>

# Enable NAT + Forwarding
# Allow VPN Client to access DGX Spark internal network and external network
#
# wg-dgx-spark = WireGuard interface name (matches the file name)
#
# <exxxxx>       = DGX Spark physical NIC name (must be confirmed on the actual machine)
# Remove `<exxxxx>`, and replace it with the DGX Spark physical NIC name value (must be confirmed on the actual machine; step 1.1 provides the command to find it)
PostUp   = iptables -A FORWARD -i wg-dgx-spark -j ACCEPT; iptables -t nat -A POSTROUTING -o <exxxxx> -j MASQUERADE
PostDown = iptables -D FORWARD -i wg-dgx-spark -j ACCEPT; iptables -t nat -D POSTROUTING -o <exxxxx> -j MASQUERADE

[Peer]
# Client PublicKey
# How to query (all Keys can only be queried by running commands on DGX Spark):
#   sudo wg show wg-dgx-spark
#   (*the output prints text; the string after "peer" is the public key)
# Or:
#   sudo cat /etc/wireguard/client_public.key
#
# Remember: Client PublicKey must be stored on DGX Spark (Server)
#
# Remove `<client_public_key>`, and replace it with the Client PublicKey value
PublicKey = <client_public_key>

# WireGuard Client fixed IP inside the VPN
AllowedIPs = 10.100.0.2/32
```

When finished:
- Press `Ctrl+O` to save
- Press `Enter` to confirm overwriting the file
- Press `Ctrl+X` to exit nano

### 4.3 Key mapping quick reference
| Whose key | (On DGX Spark Server) What command to query key | Where to paste the key |
|---|---|---|
| Server PublicKey | `cat server_public.key` or `wg show wg-dgx-spark public-key` | Client `wg-mac-pc.conf` file → paste under `[Peer] PublicKey` |
| Server PrivateKey | `cat server_private.key` | Server `wg-dgx-spark.conf` file → paste under `[Interface] PrivateLeyKey` |
| Client PublicKey | `cat client_public.key` or `wg show wg-dgx-spark` → look after `peer:` | Server `wg-dgx-spark.conf` file → paste under `[Peer] PublicKey` |
| Client PrivateKey | `cat client_private.key` | Client `wg-mac-pc.conf` file → paste under `[Interface] PrivateKey` |

- **Only on the DGX Spark Server** can you query all Server/Client keys.
- On DGX Spark Server `wg-dgx-spark.conf`, paste **Server PrivateKey** and **Client PublicKey** (crossed; if you think it through, it makes sense).
- On Mac/PC Client `wg-mac-pc.conf`, paste **Client PrivateKey** and **Server PublicKey** (crossed; if you think it through, it makes sense).

---

## 5. DGX Spark Server: enable IPv4 forwarding (required, once)
Without this step, the VPN will connect, but traffic will not be forwarded.

### 5.1 Take effect immediately
```
sysctl -w net.ipv4.ip_forward=1
```

### 5.2 Make it permanent
```
echo "net.ipv4.ip_forward=1" > /etc/sysctl.d/99-wireguard.conf
sysctl --system
```

### 5.3 Verify it
```
sysctl net.ipv4.ip_forward
```

Expected output:
```
net.ipv4.ip_forward = 1
```

---

## 6. DGX Spark Server: start WireGuard
```
systemctl enable wg-quick@wg-dgx-spark
systemctl start wg-quick@wg-dgx-spark
```

### 6.1 Check systemd (system daemon) status
```
systemctl status wg-quick@wg-dgx-spark
```

You should see:
- A long string output including text like `Active: active`

Exit using `q` (do not press Ctrl+C).

### 6.2 Check WireGuard status
```
wg
```

You should see:
- `interface: wg-dgx-spark`
- `listening port: 51820`
- `peer:` followed by a value (this is the Client Public Key)
- `AllowedIPs = 10.100.0.2/32`

---

## 7. Mac/PC Client: create WireGuard client config

Server side done! ✅

Time to set up the **client on your Mac/PC**.

### 7.0 Notes on paths and names
- Relative path:
  - `~/vpn/wireguard` (we use the relative path when running commands)
    - On macOS, the absolute path will be `User/<Your PC Name>/vpn/wireguard`; the absolute path varies by Mac's “PC Name”.
  - The commands below assume this exact relative path. If you change it, update the commands accordingly.
- File name:
  - `wg-mac-pc.conf`
  - The commands below assume this exact file name. If you change it, update the commands accordingly.

### 7.1 Create `wg-mac-pc.conf`
- Under `~/vpn/wireguard` on the Mac/PC Client, create file:
```
nano ~/vpn/wireguard/wg-mac-pc.conf
```

### 7.2 Edit `wg-mac-pc.conf` (copy/paste)
```
[Interface]
# Client PrivateKey
# How to query (all Keys can only be queried using commands on DGX Spark):
#   sudo cat /etc/wireguard/client_private.key
# Remember: Client PrivateKey must be stored on Mac-PC (Client)
#
# Remove `<client_private_key>`, and replace it with the Client PrivateKey value
PrivateKey = <client_private_key>

# WireGuard Client fixed IP inside the VPN
# Must align with AllowedIPs in DGX Spark (Server) wg-dgx-spark.conf
Address = 10.100.0.2/32

# DNS used after the VPN is enabled
# You can use Chunghwa Telecom and Google DNS, or switch to internal DNS later
# You can replace it with your favorate DNS
# It is recommended to **use your preferred DNS servers instead**.
DNS = 168.95.192.1, 8.8.8.8

[Peer]
# Server PublicKey
# How to query (all Keys can only be queried using commands on DGX Spark):
#   sudo wg show wg-dgx-spark
# Remember: Server PublicKey must be stored on Mac-PC (Client)
#
# Remove `<server_public_key>`, and replace it with the Server Public value
PublicKey = <server_public_key>

# WireGuard Server public endpoint address
# Use DGX Spark fixed Public IP on the external network
# The port must match:
#   - ListenPort in DGX Spark wg-dgx-spark.conf
#   - Router WAN Port Forwarding settings
#
# Remove `<DGX_SPARK_PUBLIC_IP>`, and replace it with DGX Spark fixed Public IP (x.x.x.x)
Endpoint = <DGX_SPARK_PUBLIC_IP>:51820

# Full Tunnel: all traffic goes through the VPN
AllowedIPs = 0.0.0.0/0

# NAT traversal keepalive (required when the Client is behind NAT)
PersistentKeepalive = 25
```

When finished:
- Press `Ctrl+O` to save
- Press `Enter` to confirm overwriting the file
- Press `Ctrl+X` to exit nano

### 7.3 Key mapping quick reference
| Whose key | (On DGX Spark Server) What command to query key | Where to paste the key |
|---|---|---|
| Server PublicKey | `cat server_public.key` or `wg show wg-dgx-spark public-key` | Client `wg-mac-pc.conf` file → paste under `[Peer] PublicKey` |
| Server PrivateKey | `cat server_private.key` | Server `wg-dgx-spark.conf` file → paste under `[Interface] PrivateLeyKey` |
| Client PublicKey | `cat client_public.key` or `wg show wg-dgx-spark` → look after `peer:` | Server `wg-dgx-spark.conf` file → paste under `[Peer] PublicKey` |
| Client PrivateKey | `cat client_private.key` | Client `wg-mac-pc.conf` file → paste under `[Interface] PrivateKey` |

- **Only on the DGX Spark Server** can you query all Server/Client keys.
- In DGX Spark Server `wg-dgx-spark.conf`, paste **Server PrivateKey** and **Client PublicKey** (crossed; if you think it through, it makes sense).
- In Mac/PC Client `wg-mac-pc.conf`, paste **Client PrivateKey** and **Server PublicKey** (crossed; if you think it through, it makes sense).

---

## 8. Mac/PC Client: VPN back home (DGX Spark Server) from an external network  

**Tip**：Enable your phone's mobile hotspot and connect your Mac/PC to it (to simulate an external network).

**On Mac/PC Client：** (must use external network)

Confirm that the Mac/PC is not connected to the home network, at this point:
- Unplug any wired ethernet
- Connect to mobile hotspot

### 8.1 Install WireGuard VPN
- macOS: install via **App Store** or **wireguard.com** (two options)
- Windows / Linux: install via **wireguard.com** (one option)

### 8.2 Import WireGuard client configuration file, and activate VPN (must use external network)

**Important：** Ensure your Mac/PC is on an external network before proceeding

(e.g., connect to your phone's mobile hotspot).

- Open WireGuard
- Click the bottom-left `+` and choose `Import Tunnel(s) from File`
- From `~/vpn/wireguard`, select `wg-mac-pc.conf`, then click `Import`
- If the displayed configuration looks correct, click `Activate`

### 8.3 Three ways to validate the `VPN tunnel` (must use external network)

#### Method 1 — Ping the default gateway of most home routers
If the VPN tunnel is working, you should be able to:
```
ping 10.100.0.1
```
or
```
ping 10.10.0.1
```

#### Method 2 — Ping DGX Spark’s home LAN IP
If the VPN tunnel is working, you should be able to:
```
# Remove `<192.168.x.x>`, and replace it with DGX Spark intranet IP address (192.168.x.x)
ping <192.168.x.x>
```

#### Method 3 — Query your fixed Public IP from “inside home”
If the VPN tunnel is working, you should see your fixed Public IP `x.x.x.x`:
```
curl ifconfig.me
```

You should see the three checks all pass.  

---

## 9. Login and Command the DGX Spark Server
### 9.1 SHH Login to the DGX Spark Server from your Mac/PC Client
Use the following simple one-line **SSH** to login DGX Spark Server. (you'll need to enter the DGX Spark boot password)

<sub><sup>＊After rebooting, simply have the Mac/PC Client run this SHH command in step 9.1 - it's super easy.</sup></sub>
```
# Remove `<DGX Spark username>`, and replace it with the username used to log in after DGX Spark boots
# Remove `<192.168.x.x>`, and replace it with DGX Spark intranet IP address (192.168.x.x)
ssh <DGX Spark username>@<192.168.x.x>
```

### 9.2 Two Example Methods to Command the DGX Spark Server from your Mac/PC Client
**On your Mac/PC Client**

Method 1: Command the DGX Spark server to report GPU temperature (GPU Temp column) and GPU utilization (GPU-Util column) once per second.
```
watch -n 1 nvidia-smi
```

Method 2: Command the DGX Spark server to report total system memory (total column) and current usage (used column) once per second.
```
watch -n 1 free -h
```
You should see both methods working correctly.  

---

# Congratulations — your Mac/PC can now reach your DGX Spark from anywhere.
<sub><sup>＊After rebooting, simply have the Mac/PC Client run this SHH command in step 9.1 - it's super easy.</sup></sub>

---
