---

# Home Lab Hardware Inventory

> **Version:** 1.1  
> **Last Updated:** [2026-04-13]  
> **Status:** Public Repo Ready (No Serials or Sensitive Data)  

This document outlines the physical components comprising the home server infrastructure, split by node and networking peripherals.

## 🖥️ Main Server Node
**Role:** Primary Compute / Storage / AI Workloads  
**Build Type:** Custom Rackmount Tower

| Component | Model / Specification | Notes |
| :--- | :--- | :--- |
| **Chassis** | Rosewill 4U Server Chassis (Rackmount) | High airflow design suitable for server loads. |
| **Motherboard** | ASRock Z370 Killer SLI/AC | Supports high-speed NVMe and DDR4 overclocking headroom. |
| **CPU** | Intel Core i7-8700K | 6-Core / 12-Thread processor for general compute tasks. |
| **GPU** | ASUS ROG Strix RTX 3090 (O24G-Gaming) | Dedicated AI acceleration & rendering. <br>💡 *Maintenance: Repasted and repadded [Month/Year].* |
| **System SSD** | Samsung NVMe M.2, 1TB *(Model: Estimated 970 EVO Plus)* | OS + Application storage. Model confirmed via visual inspection only. |
| **Storage Cache** | WD Black SN850x NVMe SSD, 4TB | Hot layer cache for storage array. High-speed access tier. |
| **Storage (HDD)** | Seagate BarraCuda 8TB HDD x4 | Total Raw Capacity: 32TB. SATA III / 5400 RPM. |
| **RAM** | G.Skill Trident Z RGB DDR4, 32GB (4x 8GB) @ 3200MHz | High-speed memory for VM/Container overhead. |
| **Power Supply** | CORSAIR HX1200i Fully Modular ATX PSU | High-efficiency unit with sufficient headroom for GPU loads. |

## 💻 MiniPC Node (Beelink)
**Role:** Media Transcoding / Knowledge Base Hosting  
**Build Type:** Commercial SFF Unit

| Component | Model / Specification | Notes |
| :--- | :--- | :--- |
| **Unit Model** | Beelink EQ14 Mini PC | Compact form factor (Twin Lake). |
| **CPU** | Intel N150 (up to 4.6GHz) | 4-Core / 4-Thread efficiency chip for Plex transcoding. |
| **RAM** | 16GB DDR4 | Sufficient for Seerr + Obsidian/CouchDB services. |
| **Storage** | 500G SSD (Internal) | OS and App storage. |

## 🌐 Networking Hardware
**Role:** Local Connectivity & Internet Uplink

| Component | Model / Specification | Notes |
| :--- | :--- | :--- |
| **Switch** | Netgear Unmanaged Switch, 5-Port | *Model Unknown.* Direct connection between Main Server and MiniPC. |
| **Uplink** | Bell HomeHub 3000 | ISP Provided Gateway / Router. |

## 🛠️ Maintenance & Notes

*   **GPU Health:** The RTX 3090 received thermal maintenance (repaste/re-pad) last month to ensure optimal temperatures during LLM training and inference tasks.
*   **Storage Configuration:** All HDDs are configured as internal storage on the Main Server; MiniPC accesses this data via SMB/Samba over LAN. SMART monitoring via scrutiny.
*   **Cache Layer:** WD Black SN850x 4TB NVMe SSD deployed as hot tier cache for newly added storage array data.

---