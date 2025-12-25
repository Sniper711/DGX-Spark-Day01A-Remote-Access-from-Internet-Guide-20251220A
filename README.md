# DGX Spark (Day01A) Remote Access from Internet Guide 20251220A 🟩 [English](https://github.com/Sniper711/DGX-Spark-Day01A-Remote-Access-from-Internet-Guide-20251220A/blob/main/DGX%20Spark%20(Day01A)%20Remote%20Access%20Guide%2020251220A.md)
# DGX Spark (第01天A) 外網遠端操控 指南 20251220A 🟩 [中文版](https://github.com/Sniper711/DGX-Spark-Day01A-Remote-Access-from-Internet-Guide-20251220A/blob/main/DGX%20Spark%20(%E7%AC%AC01%E5%A4%A9A)%20%E5%A4%96%E7%B6%B2%E9%81%A0%E7%AB%AF%E6%93%8D%E6%8E%A7%20%E6%8C%87%E5%8D%97%2020251220A.md)


## Scenarios & Advantages

From an external network on Mac/PC → via WireGuard VPN → connect back home to access DGX Spark

    Use WireGuard VPN
        Use DGX Spark as VPN Server. (Mac/PC = Client)
        The VPN penetration rate is extremely high, and using a mobile hotspot is rarely blocked by carriers.
        Configuring WireGuard with UDP port 51820 along with keepalive is the right move.
        90% of mobile networks in Taiwan allow WireGuard, but OpenVPN does not.
    Do not use Tunnelblick / OpenVPN, no need for an expensive router's built-in VPN.
        Reason 1 for VPN connection failure when using mobile hotspots: carrier intentionally block mobile network UDP/TCP 1194 Ports VPN traffic.
            Using TCP port 443 can enhance VPN penetration rates, which is relatively stable but slower, and may cause TCP-over-TCP head-of-line blocking leading to meltdown issues on mobile networks.
            Some expensive routers cannot change TCP port# which is also a problem (only TCP port 1194 is available).
        Reason 2 for VPN connection failure when using mobile hotspots: mobile network NAT causes UDP packet loss.
            Although Tunnelblick and OpenVPN use UDP, their internal implementation is similar to TCP and SSL/TLS, with many steps, making it prone to disconnections on mobile networks.
        Therefore, there is no need for Tunnelblick and OpenVPN
    Use a low-end Router
        Router: must have a fixed Public IP (x.x.x.x) and support Port Forwarding.
        Since Tunnelblick and OpenVPN are not used, the Router does not need advanced VPN features (if available, disable them), just a cheap Router is sufficient.
.....

.....

Congratulations — your Mac/PC can now reach your DGX Spark from anywhere.
        

## 適用情境 與 優點

人在外網用 Mac/PC → 透過 WireGuard VPN → 連回家 存取 DGX Spark

    全面改用 WireGuard VPN
        以 DGX Spark 為 VPN Server. (Mac/PC = Client)
        VPN 穿透率極高，行動網路開熱點上網幾乎不被行動網路阻擋.
        WireGuard 設定 UDP 51820 Port 搭配 keepalive 是正解.
        90% 在台灣行動網路，WireGuard 能過，OpenVPN 過不了.
    不用 Tunnelblick 與 OpenVPN，不用昂貴 Router 的內建 VPN Server
        行動網路開熱點連VPN失敗主因之一：電信商刻意阻擋行動網路的 UDP/TCP 1194 Ports VPN 流量.
            改用 TCP 443 Port 能增強 VPN 穿透率，相對穩定但速度慢，在行動網路可能有TCP-over-TCP隊頭阻塞(HOL Blocking)導致熔斷(Meltdown)問題.
            某些昂貴的 Router 無法改 TCP Port# 也是問題 (僅有 TCP 1194 Port).
        行動網路開熱點連VPN失敗主因之二：行動網路NAT導致UDP丟包.
            Tunnelblick 與 OpenVPN 雖然使用UDP，但內部實作卻類似TCP與SSL/TLS，步驟多，在行動網路容易中途斷掉.
        有鑑於此，不用 Tunnelblick 與 OpenVPN
    用低階的 Router
        Router：需有 固定 Public IP (x.x.x.x), 需支援 Port Forward.
        因為不使用 Tunnerblick 與 OpenVPN，所以 Router 並不需要 VPN 高階功能 (有則關閉之)，只需便宜的 Router.
.....

.....

恭喜你！從此你能從任何地方連回你心愛的 DGX Spark 了！

---

## 喜歡這個專案嗎？ 如果對您有幫助，請給一個 ★ Star 吧！這對我非常重要！

## If you find this project helpful, please give it a Star ★! Your support means a lot to me!


---
Davis Lin (Sniper711) . 
Unauthorized article copying, distribution, or modification is prohibited.
