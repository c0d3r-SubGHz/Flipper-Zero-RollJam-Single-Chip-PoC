# Flipper Zero Single-Chip RollJam: Atomic Replay PoC

![Build Status](https://img.shields.io/badge/build-passing-brightgreen)
![Firmware](https://img.shields.io/badge/firmware-Unleashed%20%7C%20RogueMaster-blue)
![Platform](https://img.shields.io/badge/platform-Flipper%20Zero-orange)
![License](https://img.shields.io/badge/license-Proprietary%20Research-red)

> **⚠️ EDUCATIONAL & RESEARCH PURPOSE ONLY**
>
> This repository contains documentation for a Proof of Concept (PoC) demonstrating a timing vulnerability in Rolling Code implementations. This tool is intended for authorized security auditing and educational research into Sub-GHz protocols.

---

## 📡 Abstract

Traditionally, the **RollJam** attack (resynchronization attack) requires two simultaneous radio transceivers: one to jam the receiver and another to capture the valid code.

This project demonstrates that a **single external CC1101 transceiver** (controlled by the Flipper Zero), combined with the internal Sub-GHz radio for reception, can successfully perform this attack.

By leveraging continuous jamming on the external module and concurrent reception on the internal radio, along with a highly optimized replay phase, this tool achieves reliable capture and retransmission of rolling codes.

During replay, the implementation uses an **Atomic Async Bit-Banging** technique (bypassing standard buffered transmission) to achieve sub-microsecond timing precision via direct GPIO manipulation, effectively minimizing OS-induced jitter that can cause modern receivers (e.g. VAG, CAME) to reject signals.

---

## 📸 Attack Lifecycle (Step-by-Step)

Full demonstration of the **Atomic Replay** sequence, from target selection to final rolling code acquisition.

| **1. Configuration** | **2. Phase 1: Injection** |
|:---:|:---:|
| ![Config](screenshots/1_config.png) | ![Jamming 1](screenshots/2_phase1_jamming.png) |
| *Target Frequency, Modulation & Jamming Offset Setup* | *Continuous Jamming Active (Waiting for 1st press)* |

| **3. Interception A** | **4. Phase 2: Desync** |
|:---:|:---:|
| ![Captured A](screenshots/3_signal_a_captured.png) | ![Jamming 2](screenshots/4_phase2_jamming.png) |
| *First Rolling Code Cached in RAM* | *Continuous Jamming Active (Waiting for 2nd press)* |

| **5. Atomic Replay** | **6. Buffering** |
|:---:|:---:|
| ![Replay A](screenshots/5_phase3_replay.png) | ![Save B](screenshots/6_save_menu.png) |
| *Unlocking Target with Signal A (Low-Jitter Replay)* | *Preparing Signal B for storage* |

| **7. Success State** | **8. Verification** |
|:---:|:---:|
| ![Saved](screenshots/7_success_saved.png) | ![Native Replay](screenshots/8_native_replay.png) |
| *Valid .sub File Written to SD* | *Playback via native Sub-GHz App* |

---

## 🎥 Live PoC Demonstration

Watch the unedited Proof of Concept video demonstrating the Continuous Offset Jamming and DWT Atomic Replay in a real-world scenario.

👉 **[WATCH THE VIDEO DEMONSTRATION HERE](https://t.me/RollJam_Flipper/53)**

---

## 🛠️ Technical Features

### 1. Continuous Jamming Strategy
The application implements **continuous-wave (CW) jamming** through the external CC1101 module rather than using duty-cycled transmission.

An **adjustable frequency offset mechanism** (e.g. ±900 kHz relative to the target carrier) is available to mitigate local RF desensitization and self-interference while preserving effective jamming performance.

---

### 2. Atomic Replay Technology
The Flipper Zero standard FIFO transmission on FreeRTOS introduces unpredictable latency during GPIO toggling. This PoC forces the CC1101 into **Asynchronous Serial Mode** (`PKTCTRL0 = 0x32`) and drives the TX pin directly via CPU cycle-accurate timing (DWT cycle counter) during the signal replay phase.

*   **Result:** A highly precise signal reproduction with minimal jitter, suitable for strict rolling code receivers.

---

### 3. Hardware Integrity Check
The code implements a physical handshake (Write/Read `0x55`) with the external module before starting the attack, preventing BusFaults or Kernel Panics if the module is disconnected.

---

### 4. Wide-Spectrum Support
Optimized for 24 frequencies, including:
*   **433.92 MHz** (Standard EU)
*   **868.35 MHz** (Secure/Rolling)
*   **433.42 MHz** (Somfy RTS)
*   **315.00 MHz** (US/Legacy)

---

## ⚙️ Hardware Requirements & Modification (CRITICAL)

*   **Device:** Flipper Zero
*   **Module:** External CC1101 Module (E07-M1101D / Pingequa / Generic)
*   **Firmware:** Tested on Unleashed / RogueMaster (Recommended).

### 🔧 The "Bus-Hijack" Hardware Mod
To achieve **Continuous Offset Jamming** and bypass the Flipper's native FreeRTOS SPI bus locks, RollJam PRO relies on a custom **Bare-Metal Software SPI (Bit-Banging)** driver. 

Because the Flipper OS automatically locks the default Chip Select (CS) on Pin 4 when an external module is detected, **you must reroute the CS pin** to avoid `furi_check` crashes and BusFaults.

**Modification Steps:**
1. **Cut or remove** the header for **Pin 4** on your module so it no longer connects to the Flipper.
2. **Solder a jumper wire** connecting the CS pad on the module directly to **Pin 16 (PC0)** on the Flipper.

| Schematic Diagram | Real-World Implementation |
|:---:|:---:|
| ![Hardware Diagram](hw_setup/1_hw_diagram.jpg) | ![Real Board](hw_setup/2_hw_photo.jpg) |
| *Cut Pin 4 and bridge CS to Pin 16.* | *Example of the rerouted CS via jumper wire.* |

*(Note: If you are using a bare CC1101 module with Dupont jumper wires instead of a PCB shield, simply route the CS wire to Pin 16 instead of Pin 4).*

---

## 📦 Installation & Activation

Due to the sensitive nature of this tool, the binary is protected via Hardware ID (UID) locking to prevent unauthorized distribution.

### Step 1: Download
The binary file (`.fap`) is not hosted directly in this repo to comply with platform policies.

### Step 2: License Activation
Upon launching the app, you will be prompted for a license key.
This software is distributed under a research license. To obtain your activation file:

1.  Visit our **[Project Hub](https://rolljam-poc-flipper.carrd.co)**.
2.  Follow the instructions to request a license for your Device UID.
3.  Place the generated `.lic` file in `/ext/apps_data/rolljam_app/`.

👉 **[GET LICENSE / JOIN RESEARCH LAB](https://rolljam-poc-flipper.carrd.co)**

---

## ⚖️ Legal Disclaimer

**Read carefully before using.**

The author of this software takes no responsibility for any damage, loss, or legal consequences caused by the use or misuse of this tool. 
*   **Do not use this tool on vehicles or devices you do not own.**
*   **Do not use this tool to interfere with public radio services.**

This software is designed to highlight vulnerabilities in legacy Rolling Code implementations so that manufacturers can improve their security standards. By using this software, you agree to use it for educational and defensive purposes only.

---

**Developed by:** `c0d3r_SubGHz`
*Sub-GHz Security Research Lab*
