---

# Home Lab: Future Roadmap & Upgrade Plan

> **Version:** 3  
> **Last Updated:** [2026-04-11]  
> **Status:** Planning Phase  

This document outlines future hardware expansions, software improvements, and strategic upgrades for the home lab infrastructure. Priorities are ranked based on **Risk Mitigation** (Need), followed by Cost and Implementation Ease.

---

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

## 📋 Phase 2: Service Consolidation & Security (Medium Priority)
*Focus: Centralizing authentication and adding personal cloud services.*

### 3. Authentik SSO Implementation 🔐
**Current State:** Individual service logins with no centralized authentication.  
**Proposed Solution:** Deploy Authentik as the identity provider for all internal services.
*   **Usage Scope:** 
    *   Single Sign-On (SSO) for all web applications.
    *   Centralized 2FA/MFA enforcement across services.
    *   User management and access control policies.
*   **Integration:** Works alongside Cloudflare Tunnels (Tailscale for network access, Authentik for application login).
*   **Why:** Improves security posture and simplifies user management without exposing additional ports.

### 4. Immich Personal Cloud 📸
**Current State:** Photos stored locally with no centralized backup or sharing.  
**Proposed Solution:** Deploy Immich for photo/video management and backup.
*   **Storage Target:** Direct write to HDD array (no SSD caching needed).
*   **Usage Scope:** 
    *   Mobile backup for personal devices.
    *   Photo organization and sharing with family.
    *   Long-term archival of media files.
*   **Why:** Provides a self-hosted Google Photos alternative with full data ownership.

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
| **Authentik SSO Implementation** | 🟡 High (Security) | 🟢 Low | 🟢 Moderate | **#3 Priority** |
| **Immich Personal Cloud** | 🟡 Medium (Convenience) | 🟢 Low | 🟢 Moderate | **#4 Priority** |
| **HDD Expansion (20TB CMR)** | 🟢 Low (Capacity) | 🔴 Very High | 🟠 Harder | **#5 Priority** |
| **Plex Transcoding Offload (Arc A310)** | 🟢 Low (Consolidation)| 🟡 Med/High | 🟠 Moderate | **#6 Priority** |

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
4.  **Planning:** Research Authentik deployment and Immich storage requirements.

---