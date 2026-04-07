
---

# Home Lab: Future Roadmap & Upgrade Plan

> **Version:** 2.2  
> **Last Updated:** [2026-04-07]  
> **Status:** Planning Phase  

This document outlines future hardware expansions, software improvements, and strategic upgrades for the home lab infrastructure. Priorities are ranked based on **Risk Mitigation** (Need), followed by Cost and Implementation Ease.

## 🎯 Strategic Objectives
1.  **Stabilize Critical Data:** Eliminate single points of failure on OS/AppData storage immediately.
2.  **Enhance Security & Access:** Secure remote connections for SSH and backend services without exposing ports publicly.
3.  **Improve Hardware Longevity:** Reduce HDD wear caused by torrent seeding via SSD caching (Hot/Cold tiered architecture).
4.  **Expand Capacity:** Increase storage capacity with high-capacity CMR drives when market prices normalize.

---

## 📋 Phase 1: Critical Stability & Security (High Priority)
*Focus: Protecting data and securing access immediately.*

### 1. OS Drive Backup Strategy 🔐 **⭐ TOP PRIORITY**
**Current Risk:** High (OS NVMe failure = Service Loss).  
**Proposed Solution:** Purchase an identical 1TB NVMe drive for redundancy or backup storage.
*   **Implementation Plan:** 
    *   **Option A (Preferred):** Daily automated script (`rsync`/`restic`) copying `/opt/appdata` and critical configs to the new NVMe. This allows versioning and easier restoration without needing RAID1 hardware support immediately.
    *   **Option B (TBD):** Configure RAID1 mirror if performance overhead is negligible compared to backup flexibility.
*   **Why:** Provides immediate protection for Docker configurations, LLM weights, and system state against drive failure or corruption.

### 2. Remote Access Hardening 🌐
**Current State:** Cloudflare Tunnels used for public web apps (OpenWebUI, Homepage).  
**Proposed Solution:** Implement **Tailscale VPN**.
*   **Usage Scope:** 
    *   SSH access to servers.
    *   Backend connectivity for Sonarr/Radarr when traveling or on mobile networks.
    *   Private network overlay for internal tools (Seerr/Plex) if needed locally without exposing ports.
*   **Note:** Cloudflare Tunnels remain the primary method for public web UI exposure; Tailscale supplements this for private/internal needs only.

---

## 📋 Phase 2: Storage Performance & Longevity (Medium Priority)
*Focus: Protecting HDDs, improving media performance via Tiered Architecture, and expanding capacity.*

### 3. Hot/Cold SSD Caching Implementation 💾 **⭐ HIGH TECHNICAL FOCUS**
**Current Pain Point:** High I/O load from torrents/QBittorrent wears out HDDs quickly; Plex latency on new content.  
**Proposed Solution:** Integrate a high-capacity (4TB) SSD as the "Hot" tier, pooled with existing HDDs ("Cold" tier).

#### **3a. MergerFS Configuration Strategy**
*   **Main Server Mount Point:** `/mnt/storage/array/` (Logical View of Storage Pool on Main Server).
*   **Policy:** `category.create=ff`. The SSD will be listed first in the mount string, ensuring all new downloads land on it by default.
*   **Config Snippet (`/etc/fstab`):**
    ```bash
    # /mnt/storage/array = [SSD Cache] + [HDD Array]
    /mnt/ssd_cache:/mnt/hdd1:/mnt/hdd2...  /mnt/storage/array  mergerfs  defaults,category.create=ff,cache.files=partial,dropcacheonclose=true,allow_other,minfreespace=100G 0 0
    ```

#### **3b. Directory Structure & Hardlinks**
To ensure space efficiency and atomic moves:
*   **Single Root:** Mount `/mnt/storage/array` in Docker (instead of separate `downloads`/`movies`).
*   **Subfolders on Main Server:** 
    *   `/mnt/storage/array/downloads`: Active seeding/downloads.
    *   `/mnt/storage/array/movies`, `/tv`, `/anime`: Library for Plex/Sonarr/Radarr.
*   **MiniPC Access (Samba):** The MiniPC mounts this via SMB to access files as: `/media/array/`.
*   **Hardlink Logic:** When Radarr moves a file, it remains on the SSD (same Inode) until archived to HDD, ensuring zero data duplication during the transition.

#### **3c. Automated Mover (`mergerfs-cache-mover`)**
A nightly Cron job manages the migration from Hot (SSD) to Cold (HDD).
*   **Seeding Policy:** `MIN_AGE=21d`. Ensures files stay on SSD for at least 3 weeks (exceeds private tracker minimums of ~14 days).
*   **Capacity Trigger:** `THRESHOLD=70%` (~2.8TB used). Script only activates when space is tight.
*   **SnapRAID Integration:** The SSD is **EXCLUDED** from the SnapRAID config to prevent constant parity syncs and HDD spin-ups during downloads. Loss of recent data (SSD failure) can be mitigated by re-downloading via Sonarr/Radarr "Missing" filter.

#### **3d. Implementation Checklist**
*   [ ] Update `/etc/fstab` with new SSD mount point first in the list (`/mnt/storage/array`).
*   [ ] Configure `mergerfs-cache-mover` script (Path: `/opt/scripts/mover.sh`).
*   [ ] Ensure mover has access to physical disk paths (`/mnt/hdd1`, etc.) to bypass mergerfs for hardlink maintenance.
*   [ ] Set up Cron schedule: Mover at 3:00 AM, SnapRAID sync at 6:00 AM.

### 4. Storage Expansion (20TB CMR Drives) 💾 **🔴 HIGH COST / LOW PRIORITY**
**Current Constraint:** Current array is full; HDD prices are currently inflated.  
**Proposed Solution:** Acquire high-capacity 20TB CMR drives to expand the "Cold" tier capacity significantly.
*   **Prerequisites:** 
    *   Requires LSI 9305-16i expansion card (to add SATA ports).
    *   May require PSU upgrade (e.g., 1200W) depending on total drive count and power draw.
*   **Status:** Monitor prices closely; purchase only when CMR pricing drops to reasonable levels compared to current market inflation.

---

## 📋 Phase 3: Consolidation & Efficiency (Low Priority)
*Focus: Power savings and simplifying hardware.*

### 5. Plex Transcoding Offload (Intel Arc A310) ⚡ **🟢 VERY LOW PRIORITY**
**Current State:** MiniPC handles transcoding; Main Server runs LLM tasks.  
**Proposed Solution:** *Theoretical Upgrade Only.* Install Intel Arc A310 to handle QSV transcodes, allowing decommissioning of the Beelink MiniPC.
*   **Status:** Low Priority / Theoretical. 
    *   Current setup works well with separate devices (MiniPC is low power).
    *   Requires physical space verification around RTX 3090 (LSI card takes priority if expansion happens first).
    *   Requires CPU check: i7-8700K lacks integrated graphics, so discrete GPU is mandatory for QSV support on this board anyway.

### 6. Out-of-Band Management (JetKVM) 🛠️ **🟢 FUTURE PROOFING**
**Current State:** None.  
**Proposed Solution:** Research JetKVM for physical access management if the server is moved to an off-site location or rack in future. 
*   **Status:** Future Proofing only, not active planning.

---

## ⚖️ Decision Matrix: Prioritization Logic

| Task | Need (Risk) | Cost ($$) | Ease of Implementation | Final Rank |
| :--- | :--- | :--- | :--- | :--- |
| **OS Backup Script + NVMe** | 🔴 Critical | 🟢 Low | 🟢 Easy | **#1 Priority** |
| **Tailscale Setup (SSH/Backend)** | 🟡 Medium | 🟢 Free | 🟢 Easy | **#2 Priority** |
| **SSD Cache / Hot-Cold Tiering** | 🟡 High (Longevity) | 🔴 Med | 🟠 Moderate | **#3 Priority** |
| **HDD Expansion (20TB CMR)** | 🟢 Low (Capacity) | 🔴 Very High | 🟠 Harder | **#4 Priority** |
| **Plex Transcoding Offload (Arc A310)** | 🟢 Low (Consolidation)| 🟡 Med/High | 🟠 Moderate | **#5 Priority** |

---

## ⚠️ Technical Notes & Caveats
*   **Physical Fit:** The RTX 3090 is physically large. Ensure there are at least two empty PCIe slots and adequate airflow before planning an Arc A310 installation or LSI card install.
*   **CPU Limitations:** The Intel i7-8700K does not have integrated graphics (UHD Graphics). Any hardware transcoding on the Main Server requires a discrete GPU with media engines (like the planned Arc A310) to function correctly with Plex/Jellyfin.
*   **SnapRAID Exclusion:** Remember that SSD data is *not* parity-protected. Recent downloads are vulnerable until moved to HDDs and synced by SnapRAID.

---

### 💡 Immediate Next Steps
1.  **Action Item:** Purchase and install identical NVMe drive for OS backup testing.
2.  **Scripting:** Draft daily `rsync` script to mirror `/opt/appdata`.
3.  **Networking:** Install Tailscale client containers on Main Server/MiniPC.

---