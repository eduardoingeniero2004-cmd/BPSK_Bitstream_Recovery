**Project 02 - Telecommunications and SDR**
**Author:** Eduardo Alberto Bañales Ramirez
**Date:** March 2026

## 1. Executive summary
This project demonstrates the implementation of a **BPSK (Binary phase Shift Keying)** transceiver using **GNU Radio**. The core objective was to achieve a reliable date link by solving critical issues, including **bit slip** and **8-bit bus alignment**

## 2. Phase Recovery and Channel Synchronization
Before implementing the data receiver, the project focused on stabilizing the physical link (PHY) under non-ideal conditions. This phase validated the modulation and corrected channel-induced phase offsets.

### 2.1. Validating Ideal Modulation
The first milestone was achieving a stable BPSK modulation. The constellation diagram confirmed the presence of the two symbols ('0' and '1') separated by 180 degrees. The recovered data was clean under these ideal conditions.

![1st Diagram](./Attachments/1st%20diagram%20perfect%20channel.png)
*Figure 1: Initial BPSK block diagram setup.*

![Correct Phase No Costas Loop 1](./Attachments/Correct%20phase%20no%20costas%20loop%201.jpg)
*Figure 2: Stable BPSK constellation diagram under ideal channel conditions.*

![Bits Recovered No Costas Loop](./Attachments/Bits%20recovered%20no%20costas%20loop.png)
*Figure 3: Initial recovered data sequence with no channel impairments.*

### 2.2. Simulating a Non-Ideal Channel
A **Phase Offset** was introduced into the simulated channel to emulate real-world propagation issues. This caused the entire constellation to rotate continuously, making symbol detection impossible.

![[Wrong phase no costas loop.png]]
*Figure 4: Rotating constellation due to channel phase offset.*

### 2.3. The Costas Loop Implementation
A **Costas Loop** was integrated into the receiver chain to track the suppressed carrier and stabilize the phase. The loop successfully locked, correcting the rotation.


![[Correct phase after costas loop.jpg]]
*Figure 5: Corrected constellation diagram showing the phase lock achieved by the Costas Loop.*

> [!INFO] Technical Note
> Although the initial flowchart for this specific stage was not captured, the constellation diagrams in Figures 4 and 5 clearly demonstrate the transition from a rotating, corrupted signal to a stabilized, detectable one.

### 2.4. Results: Recovered Bits
After the Costas Loop stabilized the phase, we were able to recover a bitstream. However, due to the Costas Loop's inherent 180-degree ambiguity, two distinct cases were observed in the recovered data.

* **Case 1 (Normal Phase):** The bit sequence matches the transmitted data.
* **Case 2 (Inverted Phase):** The entire bit sequence is logically inverted (all '0's become '1's, and vice-versa) because the Costas Loop locked to the reverse phase.

![[Bits recovery 1st case.png]]
*Figure 6: Recovered bitstream (Case 1: Normal Phase).*

![[Bits recovery 2nd case.png]]
*Figure 7: Recovered bitstream (Case 2: Inverted Phase).*

## 3. System Evolution and Receiver Design
After validating the basic modulation, the project transitioned from a simple noise-based simulation to a structured data transmission system. This required a more complex architecture to handle byte-to-bit conversion and signal recovery.

## 3. System Evolution and Receiver Design
After validating the basic modulation, the project transitioned to a structured data transmission system. This required a precise configuration of the physical layer (PHY) parameters.

### 3.1. Structured Data and Signal Shaping
The first improvement was replacing the **Random Source** with a **Vector Source**. To process this data, I implemented the **Unpack K Bits** block to serialize the 8-bit bytes into individual bits for the BPSK Modulator.

**Key Parameters for Transmission:**
* **Samples per Symbol (SpS):** I configured the system to use **8 SpS**. This high resolution ensures that the waveform is clearly defined, which is essential for accurate timing and phase recovery at the receiver.
* **Bandwidth Optimization:** The signal's bandwidth ($BW$) is directly related to the symbol rate ($R_s$). By adjusting these parameters, I ensured that the main lobe of the BPSK signal was fully utilized.

> [!MATH] Bandwidth Formula
> $$BW = 2 \cdot R_s$$

![[2nd diagram.png]]
*Figure 8: Updated flowgraph with Vector Source and optimized SpS configuration.*

### 3.2. Implementing the Full Receiver Logic
With a high-resolution signal (8 SpS), the receiver architecture became significantly more robust. I integrated the following components to achieve full bitstream recovery:

1. **Phase Synchronization:** The **Costas Loop** established a stable phase reference.
2. **Signal Conversion:** A **Binary Slicer** converted the complex symbols back into a binary format.
3. **Data Re-assembly:** The **Repack Bits** block grouped the bits back into their original 8-bit format.

**Constellation Analysis:**
Under simulated ideal conditions, the constellation shows high density and clear separation between symbols. This "perfect" recovery is a direct result of the 8 SpS setting and the Costas Loop performance.

![[New constellation.png]]
*Figure 9: High-fidelity BPSK constellation recovered under ideal conditions.*

> [!IMPORTANT] Design Observation
> Although the receiver could "see" the bits and the constellation was perfect, the data was still not synchronized with the byte boundaries. This required a manual timing adjustment (The Delay) to align the 8-bit bus.
## 4. Addressing Bit Alignment and Synchronization
Once the phase was stable, the main challenge was to reconstruct the original 8-bit bus. Even though the bits were correct, they were not aligned in their original byte positions.

### 4.1. The Bit-Slip Phenomenon
In digital communications, a **bit-slip** occurs when the receiver starts grouping bits at the wrong position. For example, if the transmitted byte is `10110010`, the receiver might start at the second bit, resulting in a completely different value.

### 4.2. Implementation of the Delay Block
To solve this, I implemented a **Delay block** before the "Repack Bits" stage. By adjusting the delay value, I manually shifted the bitstream until the output matched the original source.

* **Observed Issue:** Data was shifted by several positions, causing corrupted characters.
* **Solution:** A manual delay of **2 units** was applied. This shifted the bitstream back into alignment with the 8-bit framing.
* **Result:** The system achieved a consistent and error-free data recovery.

> [!SUCCESS] Achievement
> After applying the correct delay and ensuring the Costas Loop was locked, the "Message Recovered" sink displayed the exact sequence from the Vector Source. This confirmed the full integrity of the communication link.

## 5. Results and Data Validation
The implementation was validated by transmitting a consistent 8-bit value and verifying its correct recovery at the receiver. This section presents the experimental evidence demonstrating full system functionality after phase (Costas Loop) and bit-slip (Delay of 2) correction.

### 5.1. Transmitted Test Signal
To establish a clear baseline, I configured the Vector Source to send a constant integer value of **65**.

* **Observation:** The "Message Sent" plot displays a single, horizontal line, representing a continuous byte value.
* **Interpretation:** This serves as a reference signal with zero amplitude variance, allowing for simple visual verification at the receiver.

![[Message Sent.png]]
*Figure 10: Constant test signal (Integer 65) transmitted by the Vector Source.*

### 5.2. Synchronized Bitstreams (Source vs. Recovered)
The next step was to verify the temporal synchronization after the Delay block was applied. I monitored both the transmitted bits and the recovered bits in real-time.

* **Analysis:** The plot shows that both bit sequences are perfectly aligned on the timeline.
* **Key Finding:** There are no observable shifts or inversions. This confirms that the **Costas Loop** and the **Delay block** have correctly synchronized both the phase and the framing of the 8-bit bus.

![[Bits synchronized.png]]
*Figure 11: Real-time comparison showing high correlation between transmitted and recovered bitstreams.*

### 5.3. Final Data Recovery
The final evidence of success is the reconstruction of the original byte value at the receiver's end.

* **Result:** The "Message Recovered" plot is identical to Figure 10. It shows a single, stable horizontal line at the precise value of **65**.
* **Conclusion:** The perfect correspondence between the sent and received values proves the integrity of the complete communication link, including the full modulation and decoding chain.

![[Message Received.png]]
*Figure 12: Successfully decoded constant signal (Integer 65) at the receiver.*

> [!SUCCESS] Evidence of Robustness
> As shown in Figure 12, the system achieved error-free data transmission under ideal channel conditions. This confirms that the 8 SpS setting and the full receiver architecture are optimized for reliable bitstream recovery.
## 6. System Optimization and Future Objectives
While the current transceiver is fully functional, there are specific areas where the design can be optimized to improve efficiency and expand its capabilities.

### 6.1. Data Type Optimization
A critical observation in the current flowchart is the redundant conversion between data types. To process the bitstream, the system currently performs several **Byte-to-Float** and **Float-to-Byte** conversions.

* **Design Flaw:** These intermediate conversions increase computational overhead and latency.
* **Proposed Solution:** In future iterations, I aim to implement a more direct processing chain using fixed-point or native integer blocks to reduce CPU usage and streamline the signal path.

### 6.2. From Bitstreams to File Transfer
The current success in recovering a 8-bit bus from a Vector Source is the foundation for a more ambitious goal. The next stage of this project is to implement a **complete File Transfer system**.

* **Objective:** Transmit and receive entire files (images or text documents) over the BPSK link.
* **Requirement:** This will require adding a more robust **Framing** protocol and potentially **Forward Error Correction (FEC)** to handle packet loss.

> [!TIP] Personal Milestone
> This project serves as a technical bridge toward my long-term goal: obtaining my **Amateur Radio License**. Mastering these SDR concepts in a simulated environment is essential before moving to hardware implementation and real-air transmissions.
