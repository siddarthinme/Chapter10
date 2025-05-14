Here is a **detailed explanation of the Packet Header** as per **Section 10.6.1.1** of the IRIG 106-15 (Chapter 10) standard from your PDF:

---

### 📦 **Packet Header Overview**

* **Length**: Fixed at **24 bytes (192 bits)**
* **Purpose**: Describes and identifies the contents of the packet
* **Structure**: Consists of 10 fields laid out in a specific order

---

### 🧱 **Packet Header Fields and Their Meaning**

| **Field**                          | **Size** | **Description**                                                                          |
| ---------------------------------- | -------- | ---------------------------------------------------------------------------------------- |
| **1. Sync Pattern**                | 2 bytes  | Always `0xEB25`, used to detect start of a packet                                        |
| **2. Channel ID**                  | 2 bytes  | Identifies source of data. Each channel must be unique. Channel ID `0x0000` is reserved. |
| **3. Packet Length**               | 4 bytes  | Total packet length (header + data + trailer), in bytes. Always multiple of 4.           |
| **4. Data Length**                 | 4 bytes  | Length of **valid data only**, excluding trailer/filler                                  |
| **5. Data Type Version**           | 1 byte   | Indicates version of data type (e.g., `0x07` = RCC 106-15)                               |
| **6. Sequence Number**             | 1 byte   | Increments with each packet for a channel. Rolls over after 255                          |
| **7. Packet Flags**                | 1 byte   | Bitmask controlling time source, checksum, and secondary header usage (see below)        |
| **8. Data Type**                   | 1 byte   | Indicates type of data in packet (e.g., MIL-STD-1553, video, PCM)                        |
| **9. Relative Time Counter (RTC)** | 4 bytes  | 48-bit time value split across this and optional secondary header                        |
| **10. Header Checksum**            | 2 bytes  | 16-bit sum of all 16-bit words in header (excluding checksum field)                      |

---

### 🕹️ **Breakdown of Packet Flags Byte**

This byte has bitwise flags:

| **Bit(s)** | **Meaning**                                                                                     |
| ---------- | ----------------------------------------------------------------------------------------------- |
| Bit 7      | Secondary Header Present (1 = Yes, 0 = No)                                                      |
| Bit 6      | IPTS uses secondary header time (1 = Yes)                                                       |
| Bit 5      | RTC Sync Error (1 = Error)                                                                      |
| Bit 4      | Data Overflow (1 = Overflow occurred)                                                           |
| Bits 3-2   | Time Format: <br>`00` = 48-bit RTC (Chapter 4 format)<br>`01` = IEEE-1588<br>`10/11` = Reserved |
| Bits 1-0   | Checksum: <br>`00` = None<br>`01` = 8-bit<br>`10` = 16-bit<br>`11` = 32-bit                     |

---

### 🧠 Example Packet Header (Hex Layout)

```
EB25         → Sync Pattern
0100         → Channel ID = 0x0001
8C000000     → Packet Length = 140 bytes
78000000     → Data Length = 120 bytes
07           → Data Type Version (RCC 106-15)
0F           → Sequence Number
C1           → Packet Flags (11000001b → SecHeader, IEEE-1588, 8-bit checksum)
19           → Data Type (e.g., MIL-STD-1553)
12345678     → Relative Time Counter
9A5B         → Header Checksum
```

---

### 📘 IRIG 106 Chapter 10 — **Packet Secondary Header (Optional)**

The **Packet Secondary Header** is an optional 12-byte (96-bit) block that follows the main 24-byte Packet Header, **if enabled** by the Packet Flags (specifically, Bit 7 of the flags byte).

---

### ✅ **Purpose**

The secondary header provides **high-precision absolute time** to better time-align packet data, especially useful when using formats like IEEE 1588 or extended RTC.

---

### 📦 **Structure of the Packet Secondary Header (12 bytes total)**

It consists of the following three fields, in strict sequence:

| **Field**                        | **Size** | **Description**                                                    |
| -------------------------------- | -------- | ------------------------------------------------------------------ |
| **1. Time**                      | 8 bytes  | 64-bit timestamp. The format depends on Packet Flags (bits 2–3).   |
| **2. Reserved**                  | 2 bytes  | Must be set to zero (`0x0000`).                                    |
| **3. Secondary Header Checksum** | 2 bytes  | 16-bit checksum of all previous 10 bytes in this secondary header. |

---

### ⏱️ **Time Field Format (8 bytes)**

The actual format of this field is determined by **bits 3-2** of the Packet Flags byte in the main Packet Header:

| **Bits 3-2** | **Time Format**                       | **Structure**                                                            |
| ------------ | ------------------------------------- | ------------------------------------------------------------------------ |
| `00`         | IRIG 106 Chapter 4 Binary Time        | `High Order`, `Low Order`, `Microseconds`, `Reserved` (See Figure 10-12) |
| `01`         | IEEE 1588 Time                        | `Seconds (MSLW)` and `Nanoseconds (LSLW)` (See Figure 10-13)             |
| `10`         | Extended Relative Time Counter (ERTC) | `MSLW`, `LSLW` (1 GHz high-resolution timer) (See Figure 10-14)          |
| `11`         | Reserved                              | Not valid/used                                                           |

**Figures in the PDF:**

* **Figure 10-12:** Chapter 4 Time Format
* **Figure 10-13:** IEEE 1588 Time Format
* **Figure 10-14:** ERTC Format

---

### 🧪 **Checksum Calculation**

* The final 2 bytes in the Secondary Header contain a **16-bit checksum**.
* It's the **arithmetic sum** of the previous 10 bytes (Time + Reserved) as 16-bit words.

---

### 🔎 **Usage Notes**

* All packets on channels that use a secondary header **must use the same time format**.
* The **Time field** corresponds to the **first bit of valid data** in the packet body unless otherwise specified in the data type.

---

### 📦 IRIG 106 Chapter 10 — **Packet Body (Payload)**

The **Packet Body** is the core of a Chapter 10 packet — it contains the **actual telemetry data**. It follows immediately after the **Packet Header** (24 bytes) and **optional Secondary Header** (12 bytes if present).

---

### 🧱 **General Structure of the Packet Body**

While the exact structure varies depending on the **Data Type**, most packet bodies follow this high-level pattern:

| **Component**                         | **Presence**             | **Description**                                          |
| ------------------------------------- | ------------------------ | -------------------------------------------------------- |
| **CSDW (Channel-Specific Data Word)** | Always present (4 bytes) | Identifies how to interpret the rest of the packet data. |
| **IPTS (Intra-Packet Time Stamp)**    | Optional (varies)        | Time tag for the first data word or block.               |
| **IPDH (Intra-Packet Data Header)**   | Optional (varies)        | Metadata for sub-packet data.                            |
| **Data Words**                        | Always present           | The actual sensor/bus/video/bit stream/etc. data.        |
| **Filler**                            | Optional                 | Padding to maintain 4-byte alignment.                    |

These elements are defined specifically for each **data type** (e.g., MIL-STD-1553, PCM, Video, ARINC-429, etc.) in subsections like **10.6.2–10.6.18**.

---

### ✅ **1. Channel-Specific Data Word (CSDW)**

* **Size**: 4 bytes (32 bits)
* **Purpose**: Provides context for interpreting the packet (e.g., word count, flags, formats)
* **Always the first word in the packet body**

> 📝 Example (MIL-STD-1553): CSDW includes number of message words, gap time presence, status flags, etc.

---

### ✅ **2. Intra-Packet Time Stamp (IPTS)** *(Optional)*

* Gives precise timing for a subset of data
* Used for high-speed or asynchronous signals
* **Format**: Typically 4 or 8 bytes, depending on configuration
* Presence and format defined in the CSDW or packet header flags

---

### ✅ **3. Intra-Packet Data Header (IPDH)** *(Optional)*

* Used when data within the packet includes multiple data segments (e.g., multiple messages in one packet)
* Describes each segment's size, ID, or status
* Format varies with data type (e.g., in ARINC-429, Ethernet, or MIL-STD-1553 multi-message packets)

---

### ✅ **4. Data Words**

* These are the actual measurements, messages, or frames
* Format depends on data type:

  * **PCM**: 16-bit or 32-bit words of digitized sensor values
  * **MIL-STD-1553**: Command, Status, and Data words (20 bits each, usually stored as 32 bits)
  * **Video**: MPEG or H.264 frames
  * **Analog/Discrete**: Sampled voltages or logic signals

---

### ✅ **5. Filler Words** *(Optional)*

* Adds padding to make total packet size a multiple of 4 bytes
* Common in cases where data word count doesn’t align naturally

---

### 🧠 Key Point

Each **data type** defines its own packet body format (see sections 10.6.2 to 10.6.18 in the PDF). So to fully interpret a packet, you must:

1. Check its **Data Type** in the Packet Header
2. Use that section (e.g., 10.6.4 for MIL-STD-1553) to parse the CSDW, IPTS, IPDH, and data words

---

Based on your data type (`0x19` = **MIL-STD-1553B Data**) and data type version `0x06`, the packet body follows the **Format 1 MIL-STD-1553B message structure** as defined in section **10.6.4.2** of the IRIG 106-15 Chapter 10 PDF.

---

### 🧱 **MIL-STD-1553 Packet Body Layout (Format 1)**

Here’s the complete breakdown of what the packet body contains and how to interpret it:

---

### 1. **Channel-Specific Data Word (CSDW) — 4 bytes**

Located immediately after the Packet Header.

**Bit breakdown (from MSB to LSB)**:

* **Bits 31–30: TTB (Time Tag Bits)**
  Indicates which part of the message is time-tagged:

  * `00`: Last bit of last word
  * `01`: First bit of first word
  * `10`: Last bit of first (command) word
  * `11`: Reserved

* **Bits 29–24: Reserved**

* **Bits 23–0: MSGCOUNT**
  Total number of 1553 messages in this packet (each one is a transaction)

---

### 2. **For Each 1553 Message (repeated MSGCOUNT times)**:

Each message consists of:

#### a. **Intra-Packet Time Stamp (IPTS) — 8 bytes**

* 64-bit time value (usually 48-bit RTC + 16 bits zero-filled).
* Correlates to the bit specified by TTB.
* May represent absolute time if enabled by packet flags.

#### b. **Intra-Packet Data Header (IPDH) — 6 bytes**

Split into 3 words (each 2 bytes):

* **Block Status Word (BSW)**: Error and status flags (e.g., bus ID, sync error, word error).
* **Gap Times Word**: Bus timing gap between words (e.g., GAP1 and GAP2).
* **Length Word**: Total byte length of the upcoming message block (command, data, and status words).

#### c. **Message Data Block**

This contains the actual MIL-STD-1553 words:

* **Command Word** (always first)
* **Optional Second Command Word** (for RT-to-RT)
* **Data Words** (0 to 32)
* **Status Words** (1 or 2 depending on transaction type)

Each word is typically **32 bits** in the file, where the lower 20 bits are valid (corresponding to the actual 1553 word), and the rest may be zero or used for metadata.

---

### 3. **Packet Trailer (at the end)**

Includes:

* Optional **checksum** (8, 16, or 32-bit depending on packet flags)
* Optional **filler** (to make packet a multiple of 4 bytes)

---

### 📌 Summary Diagram (Simplified)

```
[ CSDW (4 bytes) ]
[ IPTS 1 (8 bytes) ]
[ IPDH 1 (6 bytes) ]
[ Msg 1: Cmd, Data, Status (n bytes) ]

[ IPTS 2 ]
[ IPDH 2 ]
[ Msg 2 ]

...

[ IPTS N ]
[ IPDH N ]
[ Msg N ]

[ Packet Trailer ]
```

---

