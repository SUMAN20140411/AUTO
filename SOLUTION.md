# Ghost in the CAN - CTF Challenge Solution

## Challenge Description
Track the vehicle that vanished after a genius engineer left a final message - **Code Name: Zero**. Decipher the chaotic log, saturated with ghost signals and noise, corrupted by the Digital Poltergeist.

**Flag Format:** `LISA{CAN ID}`

## Files in Repository
- `m1_can_log.csv` - The actual CAN bus log data
- `._m1_can_log.csv` - MacOS metadata file (AppleDouble format, ignored)
- `can_logs.txt` - Alternative log format (ignored per instructions)

## Solution Approach

### Step 1: Analyze CAN Log Structure
The `m1_can_log.csv` file contains CAN bus messages with the following fields:
- Time Offset
- CAN_ID
- DLC (Data Length Code)
- Payload

### Step 2: Identify CAN IDs
Seven unique CAN IDs are present in the log:
- `0x0018` - 1,179 messages
- `0x00F0` - 1,474 messages
- `0x0100` - 5,884 messages
- `0x0101` - 2,955 messages
- `0x0200` - 11,772 messages (most common)
- `0x0201` - 294 messages
- `0x0300` - 585 messages

### Step 3: Discover the Counter Pattern
CAN ID `0x0200` uses a sequential counter in the first 2 bytes of its payload:
- Messages start with counter `00`, `01`, `02`, `03`, etc.
- Counter cycles from `0x00` to `0xFF` (256 values)
- Each counter appears approximately 46 times in the log

### Step 4: Find the Ghost Signals
By examining the first sequence cycle, we discover that certain counter values are **missing from their expected positions**:

**Missing Counters (Ghost Signals):**
- `0x04` - expected at position 4, but position 4 has `0x05`
- `0x0A` - expected at position 10, but position 9 has `0x0B`
- `0x5E` - skipped in sequence
- `0x78` - skipped in sequence
- `0x94` - skipped in sequence
- `0x9B` - skipped in sequence

These counters do appear later in the log (in subsequent cycles), but they are "out of sequence" in the first cycle - creating "ghost signals."

### Step 5: Decode the Vanished Vehicle
The hint **"Code Name: Zero"** is the key to solving this challenge.

The missing/ghost counters can be interpreted in several ways:
- **0x040A** (first pair of ghost counters)
- **0x5E78** (middle pair of ghost counters)
- **0x949B** (last pair of ghost counters)

However, the hint "Code Name: Zero" strongly suggests the answer is **CAN ID 0x0000**, which:
1. Does NOT appear anywhere in the m1_can_log.csv file
2. Represents "Zero" (matching the code name)
3. Has completely "vanished" from the log

## Answer

**FLAG: `LISA{0x0000}`**

### Alternative Answers (if 0x0000 is incorrect)
- `LISA{0x040A}` - formed from first two ghost counters (04, 0A)
- `LISA{0x5E78}` - formed from middle two ghost counters (5E, 78)
- `LISA{0x949B}` - formed from last two ghost counters (94, 9B)

## Running the Solution
```bash
python3 solution.py
```

This will analyze the CAN log and display the complete solution process.

## Key Insights
1. **Ghost Signals**: Data corruption manifests as missing sequence numbers in the counter
2. **Code Name Zero**: Direct hint pointing to CAN ID 0x0000
3. **Vanished Vehicle**: A CAN ID that should exist but doesn't appear in the log
4. **Digital Poltergeist**: The force that caused certain counters to skip/disappear

## Technical Notes
- CAN (Controller Area Network) is a vehicle bus standard
- CAN IDs typically range from 0x000 to 0x7FF (11-bit identifier)
- Sequential counters are commonly used for message tracking and synchronization
- Missing messages in CAN logs can indicate network issues, tampering, or deliberate omission
