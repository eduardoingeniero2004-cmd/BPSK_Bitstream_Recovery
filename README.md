# BPSK Digital Transceiver with Bit-Stream Recovery
**Author:** Eduardo Alberto Bañales Ramirez  
**Field:** Telecommunications & Electronics Engineering  
**Tools:** GNU Radio, SDR, Linux (Lubuntu)

## Project Overview
This repository contains a full **BPSK (Binary Phase Shift Keying)** transceiver implemented in GNU Radio. The project moves beyond basic modulation by addressing real-world synchronization challenges, such as phase recovery and bit-alignment (bit-slip) in an 8-bit bus architecture.

### Key Technical Features:
* **Phase Synchronization:** Integrated **Costas Loop** for carrier tracking and phase lock.
* **Signal Shaping:** Optimized at **8 Samples per Symbol (SpS)** for high-fidelity waveform definition.
* **Data Integrity:** Implementation of a manual **Delay block** to resolve bit-slip and ensure 8-bit framing alignment.
* **Architecture:** Transitioned from random noise sources to structured **Vector Source** data transmission.

---

## Flowgraph Architecture
The final design features a complete End-to-End communication link.

![](Attachments/2nd%20diagram.png)
*Figure: Complete BPSK Transceiver with receiver synchronization logic.*

---

## Experimental Results
The following results demonstrate the robustness of the synchronization:

| Metric | Observation | Result |
| :--- | :--- | :--- |
| **Constellation** | Stable 180° separation | **Phase Locked** |
| **Bit Correlation** | 1:1 match between Source & Sink | **Synchronized** |
| **Data Recovery** | Constant 8-bit output (Value: 65) | **Error-Free** |

![Bits Synchronized](./Attachments/Bits%20synchronized.png)
*Figure: Real-time bitstream correlation after Delay adjustment.*

---

## Future Objectives
* **File Transfer:** Implement a framing protocol to transmit entire documents/images.
* **Optimization:** Reduce computational overhead by streamlining Byte-to-Float conversions.
* **SDR Hardware:** Transition from simulation to real-air transmission using **ADALM-PLUTO**.

---
*This project is part of my technical preparation for the **Amateur Radio License**.*
