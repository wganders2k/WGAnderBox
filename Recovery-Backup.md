
---

# Backup & Disaster Recovery (DR) Plan

> **Version:** 1.0  
> **Last Updated:** [2026-04-07]  
> **Status:** ⚠️ **Partial Protection Active** | Critical Configs Unbacked Up  

This document outlines the current strategy for protecting data and services within the Home Lab infrastructure. It distinguishes between hardware redundancy (SnapRAID) and actual file backups, highlighting where single points of failure currently exist.

## 🎯 Recovery Objectives
Since this lab serves personal/family use rather than commercial operations, recovery speed is prioritized over extreme urgency. However, **data loss risks are elevated for non-media files** stored on the OS drive.

*   **RTO (Recovery Time Objective):** 24–48 Hours for full system rebuild acceptable.
*   **Priority Data:**
    1.  🔐 **Docker Configs & Settings** (`/opt/appdata/docker`) - *Highest Risk.* Stored on OS NVMe, no parity protection.
    2.  🧠 **LLM Weights & Training Data** - High value, difficult to regenerate quickly. Stored on OS NVMe, no parity protection.
    3.  🎬 **Media Files** (Movies/TV) - Protected by parity but vulnerable to corruption/deletion without versioning.

## 💾 Storage Architecture & Protection Status

### ✅ What IS Protected: Media Array (HDDs)
The primary defense against hardware failure relies on **Parity**. This protects physical files stored on the 4x8TB drives if a drive fails physically.

*   **Configuration:** `3 Data Drives` + `1 Parity Drive`.
*   **Recovery Mechanism:** If a data drive dies:
    1.  Replace failed HDD with new drive of equal/greater size.
    2.  Run SnapRAID sync to rebuild parity and restore access via MergerFS.
*   **Sync Schedule:** Daily at 02:00 AM (Cron Job).

### ⚠️ What IS NOT Protected: OS Drive & Critical Data
**ALERT: Current Shortfall Detected.**  
Currently, Docker configurations (`/opt/appdata/docker`), database files, and LLM model weights reside on the **OS NVMe SSD**. While this provides fast access, it is a **Single Point of Failure (SPOF)**. SnapRAID parity does *not* protect this drive.

*   **Risk:** If the OS NVMe fails or becomes corrupted:
    1.  Docker services cannot start without re-creating containers and restoring configs manually.
    2.  LLM weights/training data are lost unless backed up externally.
    3.  Media files remain safe on the HDD array, but access to them is temporarily disrupted until rebuild.

## 🔐 Security & Secrets Management

**⚠️ Public Repository Warning:**  
This documentation repository is public. The following data must **NEVER** be committed to this repo:

*   `docker-compose.yml` secrets (API keys, DB passwords).
*   Private SSH Keys.
*   Cloudflare Tunnels tokens (if applicable for external access).
*   ProtonVPN credentials.

### Handling Secrets Locally
1.  **Environment Variables:** All sensitive data should be passed via `.env` files in the container's root directory, never hardcoded.
2.  **Local Storage:** Sensitive configuration backups must remain on physical media (USB drive) or encrypted local storage and not synced to cloud services without encryption keys.

## 🛡️ Drive Health Monitoring

Instead of manual SMART checks, we utilize a dedicated monitoring solution:

*   **Tool:** `Scrutiny` Container
*   **Function:** Automated collection and visualization of S.M.A.R.T data for all drives (HDDs & NVMe).
*   **Access:** Monitored via the local Dashboard (`/media/array/Homepage`).
*   **Alerting:** Scrutiny alerts are configured to notify via discord if drive health degrades below safe thresholds.

## 🚨 Disaster Recovery Scenarios

### Scenario A: Single HDD Failure ✅
*   **Symptom:** Service alerts indicate a missing disk in MergerFS/SnapRAID pool.
*   **Action:** 
    1.  Swap physical drive.
    2.  Run `snapraid sync`.
    3.  Verify parity rebuild completion before removing old drives from the array.

### Scenario B: OS NVMe Failure 🛑
*   **Symptom:** Server will not boot, or Docker containers fail to start due to missing config volumes.
*   **Action:** 
    1.  Replace/Repair OS drive and reinstall OS/Docker environment.
    2.  Mount HDD array to recover media files (safe).
    3.  ⚠️ **CRITICAL RISK:** Recreating Docker services requires manual reconfiguration of all containers, as `appdata` backups are not currently stored off-site or on a separate device.

### Scenario C: Data Corruption / Accidental Deletion 🛑
*   **Symptom:** Files missing or corrupted within the array (e.g., config DBs).
*   **Action:** 
    *   SnapRAID parity cannot recover deleted files effectively if they are small configs or databases.
    3.  Restore specific folders from local backup copies of `/opt/appdata` if available (**Currently not implemented**).

## 📝 Maintenance Checklist

| Task | Frequency | Command / Action | Status | Risk Level |
| :--- | :--- | :--- | :--- | :--- |
| **Scrutiny Monitoring** | Continuous | Automated SMART checks via Container UI | ✅ Active (Container) | 🟢 Low (Drive Failures) |
| **SnapRAID Sync** | Daily (02:00 AM) | `snapraid -c /etc/snapraid.conf sync` | ✅ Active via Cron | 🟡 Medium (Parity Issues) |
| **Config Backup Copy** | Weekly/Monthly | Manually copy `/opt/appdata` to external USB drive | 🔨 Not Implemented Yet | 🔴 High (Data Loss Risk) |
---