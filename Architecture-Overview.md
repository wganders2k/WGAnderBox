
---

# Home Lab Architecture Overview

> **Version:** 1.0  
> **Last Updated:** [2026-04-07]  
> **Purpose:** High-level documentation of service topology, network flow, and storage architecture for the home server environment.

## 🏗️ System Topology

The infrastructure is split across two physical nodes to isolate compute-intensive tasks (LLM inference/fine-tuning) from media transcoding workloads. Both devices reside on a local area network (LAN) connected via an unmanaged Netgear switch.

### 1. Main Server Node
**Role:** Primary Compute, Storage, and Data Management  
**Orchestration:** Dockge / Docker Compose  

This node acts as the brain of the operation. It handles heavy AI workloads, media indexing/parsing, secure torrenting, and central storage.

*   **Core Services:**
    *   🧠 **LLM Inference & Orchestration** (`llama.cpp` + OpenWebUI): Handles local Large Language Model tasks.
    *   ⬇️ **Torrent Client** (qBittorrent via Gluetun + ProtonVPN): Secure download management with traffic encrypted through the VPN tunnel.
    *   📚 **Media Management Stack** (`arrstack`): Sonarr, Radarr, Prowlarr, FlareSolverr for automation and indexing.
    *   🗄️ **File Storage:** Samba (SMB) share hosting internal drives accessible by other LAN devices.
    *   📊 **Dashboard & Monitoring:** Homepage instance aggregating status checks across the lab.

### 2. MiniPC Node (Beelink N150)
**Role:** Media Transcoding, Consumption, and Personal Knowledge  
**Orchestration:** Docker Compose / Native  

Dedicated to media playback and transcoding duties to ensure CPU resources remain available for LLM tasks on the main server.

*   **Core Services:**
    *   🎬 **Media Server:** Plex (handles hardware-assisted video transcoding).
    *   💬 **Activity Tracking:** Seerr (activity tracking for Sonarr/Radarr libraries).
    *   ✍️ **Personal Knowledge Base:** Obsidian + CouchDB (Self-hosted LiveSync setup).  
        *📝 **Note:** Currently hosted here on the MiniPC as it is low-resource. Migration to Main Server is planned but not urgent.*

## 🌐 Network & Security Flow

### Local Access (LAN)
*   Both devices communicate over the internal subnet.
*   The MiniPC mounts remote storage from the Main Server via Samba protocol to access media files.
*   No firewall rules are required between nodes for standard service communication within the trusted LAN environment.

### Remote Access (External)
To expose services securely without opening traditional ports, **Cloudflare Tunnels** (`cloudflared`) are utilized:
*   **Exposed Services:** OpenWebUI, Homepage Dashboard, Seerr, Obsidian.
*   **Security Model:** End-to-end encrypted connection between the tunnel endpoint and Cloudflare's edge; no public IP exposure required for these services.

### Privacy & Traffic Isolation
*   **Torrenting:** All torrent traffic is routed through `gluetun` containers to ensure anonymity via ProtonVPN.
*   **AI vs. Media:** The separation of LLM tasks (Main Server) from Plex Transcoding (MiniPC) prevents resource contention, ensuring smooth media playback even during heavy AI inference or fine-tuning sessions on the main unit.

## 💾 Data Flow & Storage Architecture

| Component | Location | Access Method |
| :--- | :--- | :--- |
| **System Drives** | Main Server (Internal) | Local Mount / Docker Volumes |
| **Media Library** | Main Server (Internal) | Samba Share (LAN) → MiniPC |
| **Obsidian Data** | MiniPC (Local SSD/HDD) | CouchDB Sync (Self-hosted) |
| **Config/Data** | Main Server & MiniPC | Local to respective containers/vms |

1.  Media is stored physically on the **Main Server**.
2.  The **MiniPC** mounts this storage via SMB/Samba for Plex scanning and transcoding.
3.  ArrStack services (on Main) manage metadata but reference files across the network share when necessary.
4.  Obsidian data currently persists locally on the MiniPC until migration to the Main Server is scheduled.

---