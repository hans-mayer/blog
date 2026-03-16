---
layout: post
title:  Comparison Precise Point Positioning (PPP) Methods
date:   2026-03-16 14:02:00 CET
categories: 
---

# Comprehensive Comparison: Precise Point Positioning (PPP) Methods

This report documents the evolution of GNSS measurement techniques from basic statistical averaging to professional-grade post-processing and real-time RTK solutions. All experiments were conducted using a fixed roof-mounted antenna and u-blox ZED-F9P/ZED-X20P receivers.

---

## 1. Overview of Evaluated Methods

| Method | Title | Core Technique | Data Source |
| :--- | :--- | :--- | :--- |
| **1** | **Statistical Averaging** | Long-term mean calculation (24h) | Autonomous GNSS |
| **2** | **CSRS-PPP Service** | Cloud-based post-processing | Global Ephemerides |
| **3** | **RTKLIB (Local)** | Manual post-processing | Local Base (APOS) |
| **4** | **NTRIP (Hardware)** | Internal RTK Engine | Real-time NTRIP Stream |
| **5** | **RTKNAV (Software)** | External RTK processing | Real-time NTRIP Stream |

---

## 2. Detailed Method Analysis

### Method 1: Precise Point Positioning with Averaging
This "brute force" approach relies on the law of large numbers. By averaging data over 24 hours, local ionospheric fluctuations are partially smoothed out.
* **Performance:** Achieved a precision within a **20 cm radius**.
* **Key Insight:** Changing the `dynModel` to "stationary" did not significantly improve the result. The absolute offset compared to corrected methods remained over 1 meter.

### Method 2: CSRS-PPP & ECTT Transformation
Utilizes the Canadian Spatial Reference System (CSRS) for professional post-processing.
* **Workflow:** Collect RINEX data -> Upload to CSRS -> Wait for "Final" orbit products -> Transform coordinates.
* **The Transformation Factor:** Results are delivered in **ITRF**. For European accuracy, the **ECTT tool** must be used to convert to **ETRF**, accounting for tectonic plate drift (approx. 2.5 cm/year).

### Method 3: RTKLIB with Local Correction (APOS)
The most precise "offline" method using local reference stations from the Austrian BEV (APOS service).
* **Technique:** Manual calculation using `rnx2rtkp` and `.obs` files from both the rover and a nearby base station (e.g., `WIEN00AUT`).
* **Results:** Extremely tight clustering with a maximum deviation of only **26 mm**.
* **Comparison:** After transformation, it aligned within **11 cm** of the CSRS-PPP result.

### Method 4: NTRIP over gpsd (Hardware RTK)
A real-time solution where the u-blox ZED-F9P processes RTCM3 correction data internally.
* **Workflow:** Direct NTRIP stream via `gpsd` to the receiver.
* **Stability:** High percentage of **Status: FIXED** (Q=1). 
* **Precision:** The average value was only **3.9 cm** away from the high-precision results of Method 3.

### Method 5: Software-Based RTK (rtknavi_qt)
Utilizes the RTKlib software suite to perform the heavy lifting of RTK calculations on a host PC rather than the chip.
* **Hardware:** Conducted with the **u-blox ZED-X20P**.
* **Observations:** It required approximately **30 minutes** to achieve a "FIX."
* **Control:** Offers the most granular control over navigation systems (GPS, Galileo, BDS) and satellite selection.

---

## 3. Comparison Table

| Feature | Method 1 | Method 2 | Method 3 | Method 4 | Method 5 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Real-Time** | No | No | No | **Yes** | **Yes** |
| **Complexity** | Low | Medium | High | Medium | High |
| **Accuracy (Relative)** | ~20-40 cm | < 5 cm | **< 3 cm** | ~5 cm | ~5-10 cm |
| **Ref. System** | WGS84 | ITRF (Global) | ETRF (Local) | ETRF | ETRF |
| **Data Effort** | Zero | RINEX Upload | RINEX + Base | NTRIP Login | NTRIP + Config |

---

## 4. Final Conclusion & Recommendations

1.  **For Static Surveying:** Method 3 (Post-processing with local APOS data) is the gold standard, providing the highest repeatability and millimeter-level precision.
2.  **For Daily Use:** Method 4 (NTRIP into F9P) is the most efficient. It provides professional "Fixed" solutions in real-time with minimal software overhead.
3.  **Critical Factor:** When comparing results over time (e.g., 2023 vs 2026), always perform a **coordinate transformation** (ITRF to ETRF). Without it, continental drift will be misinterpreted as measurement inaccuracy.

## 5. Links to this 5 methods 

(1) [PPP - Precise Point Positioning with averaging](/2023/06/03/PPP-Precise-Point-Positioning.html){:target="_blank"} <br>
(2) [PPP with gpsrinex, CSRS-PPP and ECTT](/2026/01/21/PPP-with-gpsrinex.html){:target="_blank"} <br>
(3) [PPP with RTKlib and local correction](/2026/02/21/PPP-with-RTKLIB.html){:target="_blank"} <br>
(4) [PPP with NTRIP source for u-blox GNSS receiver over gpsd](/2026/02/28/PPP-with-NTRIP-source.html){:target="_blank"} <br>
(5) [PPP with NTRIP source and rtknavi_qt](/2026/03/15/PPP-with-NTRIP-source-and-rtknavi_qt.html){:target="_blank"} <br>


