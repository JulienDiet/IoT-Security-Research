# IoT Security Research

This repository collects research work and practical labs focused on offensive IoT security. It documents hardware vulnerability analyses, firmware extraction, and reverse engineering on real-world devices.

## ⚠️ Disclaimer

This work was conducted within an educational and ethical framework. The techniques presented here must only be used on hardware you own or for which you have explicit authorization.

## Projects

### 1. Security Audit and LoRaWAN Cloning (ESP32)

This project explores the physical security of a LoRaWAN node based on a Heltec ESP32 WiFi LoRa 32 (V3). The goal was to demonstrate how temporary physical access allows for the compromise of the entire chain of trust.

**Objectives**
* Firmware extraction (Dumping) via the serial interface.
* Reverse engineering to exfiltrate cryptographic keys (DevEUI, AppKey).
* Device cloning and identity spoofing on the LoRaWAN network.

**Tools and Techniques**
* Hardware: Heltec ESP32 (V3), CP210x drivers.
* Software: esptool (extraction), Ghidra (analysis), Arduino IDE.
* Vulnerability: Unsecured bootloader and cleartext key storage.

**Methodology**
1. Dumping: Extraction of flash memory (range 0x10000 to 0xF0000) via the USB-UART port using esptool.
2. Differential Analysis: Comparison of a known firmware with the target firmware in Ghidra to locate the data structure (.rodata).
3. Exfiltration: Recovery of the target's DevEUI and AppKey.
4. Spoofing: Reprogramming a second device with the exfiltrated secrets to send valid data to the LoRaWAN gateway.

### 2. Hardware Analysis and Firmware Extraction (TP-Link Tapo C100)

This project documents the hardware analysis of a TP-Link Tapo C100 camera. The objective is to obtain privileged access to the system, extract the complete firmware, and analyze the file system for sensitive artifacts.

**Method 1: Root Access via UART**
This approach exploits the camera's serial interface to obtain a root shell on the embedded Linux system (BusyBox).

* Hardware Reconnaissance: Disassembly and identification of UART test points (TX, RX, GND) on the PCB.
* Connection: Use of an FTDI USB-TTL adapter and PCB probes (baud rate: 115200).
* Exploitation: Interception of boot logs and authentication using known credentials (`slpingenic`).
* Result: Full access to the file system and identification of the firmware partition (`/dev/mtd10`).

**Method 2: Flash Extraction & Forensic Analysis**
This method bypasses software protections by interacting directly with the SPI Flash chip to perform a full memory dump and offline analysis.

* **Hardware:** CH341A Programmer (Black Edition) and SOIC8/SOP8 test clip.
* **Software:** IMSProg (Linux) for extraction, Unblob & SquashFS-tools for analysis.
* **Process:**
    1. **Dump:** In-circuit reading of the XMC 25QH64 chip via SPI protocol to obtain a raw binary image.
    2. **Extraction:** Automated analysis using `unblob` to identify and extract the SquashFS root filesystem.
    3. **Forensics:** Exploration of the extracted `rootfs`.
* **Findings:** Discovery of unencrypted RSA private keys and certificates in `/etc` (e.g., `uhttpd.key`, `uhttpd.crt`) and `/etc/data/dss_certificates`, confirming the exposure of sensitive cryptographic material at rest.
