
---

# Home Lab: Future Roadmap & Upgrade Plan

> **Version:** 2.0  
> **Last Updated:** [Date]  
> **Status:** Planning Phase  

This document outlines future hardware expansions, software improvements, and strategic upgrades for the home lab infrastructure. Priorities are ranked based on **Risk Mitigation** (Need), followed by Cost and Implementation Ease.

## 🎯 Strategic Objectives
1.  **Stabilize Critical Data:** Eliminate single points of failure on OS/AppData storage immediately.
2.  **Enhance Security & Access:** Secure remote connections for SSH and backend services without exposing ports publicly.
3.  **Improve Hardware Longevity:** Reduce HDD wear caused by torrent seeding via SSD caching (future phase).
4.  **Optimize Efficiency:** Keep the MiniPC for transcoding unless consolidation becomes absolutely necessary.

---

## 📋 Phase 1: Critical Stability & Security (High Priority)
*Focus: Protecting data and securing access immediately.*

### 1. OS Drive Backup Strategy 🔐 **⭐ TOP PRIORITY**
**Current Risk:** High (OS NVMe failure = Service Loss).  
**Proposed Solution:** Purchase an identical 1TB NVMe drive for redundancy or backup storage.
*   **Implementation Plan:** 
    *   **Option A (Preferred):** Daily automated script (`rsync`/`restic`) copying `/opt/appdata` and critical configs to the new NVMe. This allows versioning and easier restoration without needing RAID1 hardware support.
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
*Focus: Protecting HDDs and improving media performance.*

### 3. SSD Caching with `mergerfs-cache-mover` 💾
**Current Pain Point:** High I/O load from torrents/QBittorrent wears out HDDs quickly; Plex latency on new content.  
**Proposed Solution:** Add a high-capacity (4TB+) NVMe/SATA SSD as a cache pool.
*   **Workflow:** 
    1.  New downloads write to the SSD Cache Pool first.
    2.  `mergerfs-cache-mover` script moves files older than 3-4 weeks to HDD Storage Pool automatically.
*   **Benefits:** 
    *   Reduces noise (HDDs spin down more often).
    *   Extends drive lifespan by reducing write cycles on spinning rust.
    *   Improves Plex performance for "hot" content (new releases stay on SSD).

### 4. Storage Expansion & Connectivity 🛠️
**Current Constraint:** Limited SATA ports on Z370 Mobo; PSU headroom may be tight with expansion.  
**Proposed Solution:** 
*   **LSI Card Upgrade:** Install an LSI 9211/9305-16i (IT Mode) to add more SATA ports for future drive bays or cache drives.
    *   *Note:* This card takes priority over GPU expansion if physical space is tight.
*   **PSU Upgrade:** Replace CORSAIR RM850x with a higher wattage unit (e.g., **1200W Gold**) if adding multiple high-power GPUs or expanding HDD count significantly later.
*   **Drive Acquisition:** Monitor market for 20TB CMR drives to expand the array when prices normalize below current inflated rates.

---

## 📋 Phase 3: Consolidation & Efficiency (Very Low Priority)
*Focus: Power savings and simplifying hardware.*

### 5. Plex Transcoding Offload (Intel Arc A310) ⚡
**Current State:** MiniPC handles transcoding; Main Server runs LLM tasks.  
**Proposed Solution:** *Theoretical Upgrade Only.* Install Intel Arc A310 to handle QSV transcodes, allowing decommissioning of the Beelink MiniPC.
*   **Status:** Low Priority / Theoretical. 
    *   Current setup works well with separate devices.
    *   Requires physical space verification around RTX 3090 (LSI card takes priority if expansion happens).
    *   Requires CPU check: i7-8700K lacks integrated graphics, so discrete GPU is mandatory for QSV support on this board anyway.

### 6. Out-of-Band Management (JetKVM) 🛠️
**Current State:** None.  
**Proposed Solution:** Research JetKVM for physical access management if the server is moved to an off-site location or rack in future. 
*   **Status:** Future Proofing only.

---

## ⚖️ Decision Matrix: Prioritization Logic

| Task | Need (Risk) | Cost ($$) | Ease of Implementation | Final Rank |
| :--- | :--- | :--- | :--- | :--- |
| **OS Backup Script + NVMe** | 🔴 Critical | 🟢 Low | 🟢 Easy | **#1 Priority** |
| **Tailscale Setup (SSH/Backend)** | 🟡 Medium | 🟢 Free | 🟢 Easy | **#2 Priority** |
| **SSD Cache / LSI Card** | 🟡 Medium | 🔴 High | 🟠 Moderate | **#3 Priority** |
| **Plex Transcoding Offload (Arc A310)** | 🟢 Low | 🟡 Med | 🟠 Harder | **#4 Priority** |

---

## ⚠️ Technical Notes & Caveats
*   **Physical Fit:** The RTX 3090 is physically large. Ensure there are at least two empty PCIe slots and adequate airflow before planning an Arc A310 installation or LSI card install.
*   **CPU Limitations:** The Intel i7-8700K does not have integrated graphics (UHD Graphics). Any hardware transcoding on the Main Server requires a discrete GPU with media engines (like the planned Arc A310) to function correctly with Plex/Jellyfin.

---

### 💡 Immediate Next Steps
1.  **Action Item:** Purchase and install identical NVMe drive for OS backup testing.
2.  **Scripting:** Draft daily `rsync` script to mirror `/opt/appdata`.
3.  **Networking:** Install Tailscale client containers on Main Server/MiniPC.

---
