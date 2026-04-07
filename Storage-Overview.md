
---

# Storage Architecture & File Structure Guide

> **Version:** 1.0  
> **Last Updated:** [2026-04-07]  
> **Purpose:** Documentation of storage pooling strategy, directory layout, and mount points for the Home Lab infrastructure.  

## 💾 Storage Pool Overview

The main server utilizes a hybrid redundancy solution combining **MergerFS** (for pooling) and **SnapRAID** (for parity protection). This setup maximizes usable capacity while protecting against single drive failure without the complexity of traditional RAID.

### Physical Drive Layout
| Slot | Mount Point | Function | Capacity | Role |
| :--- | :--- | :--- | :--- | :--- |
| **HDD 1** | `/mnt/data1` | Storage Pool Member | 8TB | Data |
| **HDD 2** | `/mnt/data2` | Storage Pool Member | 8TB | Data |
| **HDD 3** | `/mnt/data3` | Storage Pool Member | 8TB | Data |
| **HDD 4** | `/mnt/parity1` | Parity Volume | 8TB | Redundancy (Parity) |

*   **Logical Pool:** The data drives are pooled together via MergerFS, typically mounted at `/mnt/storage`.
*   **Redundancy Strategy:** `3 Data + 1 Parity`. If one drive fails, parity can be used to rebuild the lost data.
*   **Access Point:** Services access files via the merged path: `/media/` (mounted from the pool).

## 📂 Directory Structure & Workflow

All media and download directories reside under the main storage mount point. The structure is organized for easy maintenance by Sonarr/Radarr/Plex services.

```text
/media/array/
├── TV/           # Series content accessed via Plex/Sonarr
├── Movies/       # Movie content accessed via Plex/Radarr
├── Anime/        # Anime series content
└── Downloads/    # Incomplete downloads (qBittorrent output)

# Note: Sonarr/Radarr perform hardlinks from /media/array/* to the actual storage location.
```

### Download Workflow
1.  **Ingestion:** qBittorrent saves files initially to `/downloads`.
2.  **Processing:** ArrStack services (Sonarr, Radarr) move or scan these files.
3.  **Final Storage:** Files are hardlinked into the appropriate media folder (`/media/array/Movies`, etc.) to save space while keeping metadata consistent across devices.

## 🌐 Network Access (Samba)

The MiniPC and other LAN clients access storage via Samba over the local network. The shares correspond directly to the logical array folders.

| Share Name | Path on Server | Description |
| :--- | :--- | :--- |
| `array` | `/media/array/` | Media library root (TV, Movies, Anime) |
| `downloads` | `/downloads/` | Download queue access (optional for LAN users) |

**Mount Configuration:**  
Samba shares are configured in the MiniPC's `/etc/fstab`. All network mounts can be refreshed using:
```bash
sudo mount -a
```

## 🐳 Docker Application Data Strategy

To ensure consistency across reboots and container recreation, a centralized path for `appdata` is planned. This is currently **High Priority** but not yet fully implemented.

### Planned Directory Structure
All service configurations will be stored under `/opt/appdata/docker/{service}/config`.

```text
/opt/
└── appdata/
    └── docker/
        ├── openwebui/
        │   └── config/      # OpenWebUI data/volumes
        ├── qbittorrent/
        │   └── config/      # qBittorrent settings
        ├── sonarr/
        │   └── config/      # Sonarr database/settings
        └── ...             # Other services follow same pattern
```

**Note:** Existing containers should be migrated to this structure during the next maintenance window. Volume mounts in `docker-compose` files will point here instead of default paths.

## ⏰ Maintenance & Backup Schedule

| Task | Tool | Frequency | Time (Server Local) | Description |
| :--- | :--- | :--- | :--- | :--- |
| **Parity Sync** | SnapRAID | Daily | 02:00 AM | Updates parity data to reflect file changes. Runs via Cron Job. |
| **Storage Health** | Smartmontools | Weekly | Sunday Night | Checks drive health status and SMART attributes. |

### Cron Job Example (SnapRAID)
```bash
# Run daily at 2 AM for snapraid sync
0 2 * * * /usr/bin/snapraid -c /etc/snapraid.conf sync
```

---
