
---

# 🏠 Home Lab Documentation

> **Welcome to my personal home server infrastructure documentation!**  
> This repo serves as a single source of truth for hardware, software architecture, storage strategies, and future upgrades. It is intended for personal reference but shared publicly here in case it helps others (just keeps things organized!). 

## 📚 Documentation Index
Navigate through the technical documentation below to understand how this lab is built:

| Document | Description | Link |
| :--- | :--- | :--- |
| **Architecture Overview** | High-level topology, service flow, and network security. | [`Architecture-Overview.md`](./Architecture-Overview.md) 🏗️ |
| **Hardware Inventory** | Detailed list of physical components (Server & MiniPC). | [`Hardware-Inventory.md`](./Hardware-Inventory.md) 💾 |
| **Storage Structure** | File system layout, MergerFS/SnapRAID config. | [`Storage-Overview.md`](./Storage-Overview.md) 📂 |
| **Backup & DR Plan** | Current redundancy status and recovery procedures. | [`Recovery-Backup.md`](./Recovery-Backup.md) 🛡️ |
| **Future Roadmap** | Planned upgrades, priorities, and research notes. | [`Roadmap.md`](./Roadmap.md) 🗺️ |

## 🔍 Quick Glossary
Since home lab setups often use specific terminology, here are the key terms used throughout these docs:

*   **Dockge:** A graphical management interface for Docker Compose projects (used on both servers).
*   **MergerFS:** A FUSE program that combines multiple directories into a single virtual directory. Used to pool HDDs and SSDs together.
*   **SnapRAID:** A parity backup solution designed specifically for mass storage systems, not meant for real-time redundancy like RAID 1/5 (used here for weekly/daily backups).
*   **qBittorrent + Gluetun:** The torrent client paired with a Dockerized VPN container to encrypt traffic.
*   **ArrStack:** A collection of automation tools (`Sonarr`, `Radarr`) used to manage media libraries.
*   **LSI 9305-16i:** An expansion card used to add more SATA ports if the motherboard runs out (planned upgrade).
*   **Tailscale:** A Zero Trust network overlay protocol for secure remote SSH and backend access without opening public ports.

## ⚠️ Important Notes
*   **Public Repo Safety:** This repository contains **NO** sensitive credentials, API keys, or private IP addresses. All secrets are stored locally only (e.g., `.env` files). 
*   **Hardware Risks:** Some components listed in the Hardware Inventory may be repurposed from previous builds and have varying warranty statuses.

---