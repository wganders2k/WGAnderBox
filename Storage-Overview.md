
---

# Storage Architecture & File Structure Guide

> **Version:** 2.0  
> **Last Updated:** [2026-04-11]  
> **Purpose:** Documentation of the hybrid storage pooling strategy (MergerFS + SnapRAID + SSD Cache), directory layout, and monitoring infrastructure for the Home Lab.  

## 💾 Storage Pool Overview

The server utilizes a **3-tier storage architecture** combining **MergerFS** for pooling, **SnapRAID** for parity protection, and an **NVMe SSD** as a high-speed cache layer. This setup maximizes performance for active data while maintaining cost-effective capacity and redundancy for cold data.

### Physical Drive Layout
| Slot | Mount Point | Function | Capacity | Role |
| :--- | :--- | :--- | :--- | :--- |
| **NVMe 1** | `/mnt/ssd_cache` | Cache Layer (Hot) | 4TB | High-Speed I/O (No Parity) |
| **HDD 1** | `/mnt/data1` | Storage Pool Member | 8TB | Data |
| **HDD 2** | `/mnt/data2` | Storage Pool Member | 8TB | Data |
| **HDD 3** | `/mnt/data3` | Storage Pool Member | 8TB | Data |
| **HDD 4** | `/mnt/parity1` | Parity Volume | 8TB | Redundancy (Parity) |

*   **Logical Pool (Hot + Cold):** The SSD and HDDs are pooled together via MergerFS, mounted at `/mnt/storage`.
*   **Logical Pool (Cold Only):** The HDDs alone are accessible at `/mnt/storage_cold` for direct parity operations or specific cold workloads.
*   **Redundancy Strategy:** `3 Data (HDD) + 1 Parity`. The NVMe cache is **not** protected by SnapRAID; it serves purely as a performance buffer.
*   **Access Point:** Services access files via the merged path: `/mnt/storage`.

## 📂 Directory Structure & Workflow

The directory structure distinguishes between the **Cache (Hot)** and **Archive (Cold)** layers. The `mergerfs-cache-mover` script manages the transition of files between these layers.

```text
/mnt/storage/ (MergerFS Pool: SSD + HDDs)
├── Downloads/          # ⚠️ SSD ONLY: Active downloads (qBittorrent output)
├── TV/                 # SSD or HDD: Transferred to HDDs when idle
├── Movies/             # SSD or HDD: Transferred to HDDs when idle
└── Anime/              # SSD or HDD: Transferred to HDDs when idle

/mnt/storage_cold/      # Direct access to HDDs only (No SSD)
└── (Mirror of TV, Movies, Anime after migration)
```

### Cache Migration Workflow
1.  **Ingestion:** qBittorrent saves files initially to `/mnt/storage/Downloads/` (NVMe SSD).
2.  **Processing:** ArrStack services (Sonarr, Radarr) move or scan these files.
3.  **Caching:** New media is written to the SSD for fast access.
4.  **Migration:** The `mergerfs-cache-mover` script runs daily at **03:00 AM**.
    *   It identifies files in `/mnt/storage/TV`, `/mnt/storage/Movies`, and `/mnt/storage/Anime` that have not been accessed recently.
    *   It moves these files from the SSD to the HDDs.
    *   **Note:** The `Downloads` directory remains exclusively on the SSD and is **not** migrated.

## 🌐 Network Access (Samba)

LAN clients access the storage via Samba. The shares point to the MergerFS pool, allowing seamless access to both hot (SSD) and cold (HDD) data.

| Share Name | Path on Server | Description |
| :--- | :--- | :--- |
| `data` | `/mnt/storage/array` | Media library root (TV, Movies, Anime) |
| `downloads` | `/mnt/storage/array/Downloads` | Active download queue (SSD only) |

**Mount Configuration:**  
Samba shares are configured in the MiniPC's `/etc/fstab`. All network mounts can be refreshed using:
```bash
sudo mount -a
```

## 🐳 Docker Application Data Strategy

Service configurations are stored in a centralized path to ensure consistency across reboots.

### Directory Structure
All service configurations are stored under `/opt/appdata/docker/{service}/config`.

```text
/opt/
└── appdata/
    └── docker/
        ├── openwebui/
        │   └── config/
        ├── qbittorrent/
        │   └── config/
        ├── sonarr/
        │   └── config/
        └── scrutiny/
            └── config/
```

**Note:** Volume mounts in `docker-compose` files point here. Existing containers should be migrated to this structure during maintenance.

## 🛡️ Monitoring & Maintenance

### Drive Health Monitoring (Scrutiny)
**Scrutiny** is deployed as a Docker container to monitor SMART data for all drives (HDDs and NVMe).

*   **Container Name:** `scrutiny`
*   **Mount Configuration:**
    ```yaml
    # Example mount for Scrutiny
    - /dev/sda:/dev/sda
    - /dev/sdb:/dev/sdb
    - /dev/sdc:/dev/sdc
    - /dev/sdd:/dev/sdd
    - /dev/nvme0n1:/dev/nvme0n1
    - /dev/nvme1n1:/dev/nvme1n1
    ```
*   **Access:** Web UI available at `http://<server-ip>:8080`

### Maintenance Schedule

| Task | Tool | Frequency | Time (Server Local) | Description |
| :--- | :--- | :--- | :--- | :--- |
| **Parity Sync** | SnapRAID | Daily | 02:00 AM | Updates parity data for HDDs. |
| **Cache Migration** | mergerfs-cache-mover | Daily | 03:00 AM | Moves inactive files from SSD to HDDs. |
| **Storage Health** | Scrutiny | Continuous | N/A | Monitors SMART attributes for all drives. |

### Cron Job Example (SnapRAID)
```bash
# Run daily at 2 AM for snapraid sync
nsenter -t 1 -m -u -n -i bash -c "snapraid sync && snapraid scrub -p 8 -o 10"
```

---