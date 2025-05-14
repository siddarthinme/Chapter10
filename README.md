📦 Packet Header Overview
Length: Fixed at 24 bytes (192 bits)

Purpose: Describes and identifies the contents of the packet

Structure: Consists of 10 fields laid out in a specific order

🧱 Packet Header Fields and Their Meaning
Field	Size	Description
1. Sync Pattern	2 bytes	Always 0xEB25, used to detect start of a packet
2. Channel ID	2 bytes	Identifies source of data. Each channel must be unique. Channel ID 0x0000 is reserved.
3. Packet Length	4 bytes	Total packet length (header + data + trailer), in bytes. Always multiple of 4.
4. Data Length	4 bytes	Length of valid data only, excluding trailer/filler
5. Data Type Version	1 byte	Indicates version of data type (e.g., 0x07 = RCC 106-15)
6. Sequence Number	1 byte	Increments with each packet for a channel. Rolls over after 255
7. Packet Flags	1 byte	Bitmask controlling time source, checksum, and secondary header usage (see below)
8. Data Type	1 byte	Indicates type of data in packet (e.g., MIL-STD-1553, video, PCM)
9. Relative Time Counter (RTC)	4 bytes	48-bit time value split across this and optional secondary header
10. Header Checksum	2 bytes	16-bit sum of all 16-bit words in header (excluding checksum field)

🕹️ Breakdown of Packet Flags Byte
This byte has bitwise flags:

Bit(s)	Meaning
Bit 7	Secondary Header Present (1 = Yes, 0 = No)
Bit 6	IPTS uses secondary header time (1 = Yes)
Bit 5	RTC Sync Error (1 = Error)
Bit 4	Data Overflow (1 = Overflow occurred)
Bits 3-2	Time Format:
00 = 48-bit RTC (Chapter 4 format)
01 = IEEE-1588
10/11 = Reserved
Bits 1-0	Checksum:
00 = None
01 = 8-bit
10 = 16-bit
11 = 32-bit

🧠 Example Packet Header (Hex Layout)
pgsql
Copy
Edit
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


📘 IRIG 106 Chapter 10 — Packet Secondary Header (Optional)
The Packet Secondary Header is an optional 12-byte (96-bit) block that follows the main 24-byte Packet Header, if enabled by the Packet Flags (specifically, Bit 7 of the flags byte).

✅ Purpose
The secondary header provides high-precision absolute time to better time-align packet data, especially useful when using formats like IEEE 1588 or extended RTC.

📦 Structure of the Packet Secondary Header (12 bytes total)
It consists of the following three fields, in strict sequence:

Field	Size	Description
1. Time	8 bytes	64-bit timestamp. The format depends on Packet Flags (bits 2–3).
2. Reserved	2 bytes	Must be set to zero (0x0000).
3. Secondary Header Checksum	2 bytes	16-bit checksum of all previous 10 bytes in this secondary header.

⏱️ Time Field Format (8 bytes)
The actual format of this field is determined by bits 3-2 of the Packet Flags byte in the main Packet Header:

Bits 3-2	Time Format	Structure
00	IRIG 106 Chapter 4 Binary Time	High Order, Low Order, Microseconds, Reserved (See Figure 10-12)
01	IEEE 1588 Time	Seconds (MSLW) and Nanoseconds (LSLW) (See Figure 10-13)
10	Extended Relative Time Counter (ERTC)	MSLW, LSLW (1 GHz high-resolution timer) (See Figure 10-14)
11	Reserved	Not valid/used

Figures in the PDF:

Figure 10-12: Chapter 4 Time Format

Figure 10-13: IEEE 1588 Time Format

Figure 10-14: ERTC Format

🧪 Checksum Calculation
The final 2 bytes in the Secondary Header contain a 16-bit checksum.

It's the arithmetic sum of the previous 10 bytes (Time + Reserved) as 16-bit words.

🔎 Usage Notes
All packets on channels that use a secondary header must use the same time format.

The Time field corresponds to the first bit of valid data in the packet body unless otherwise specified in the data type.

