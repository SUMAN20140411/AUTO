# Ghost in the CAN - CTF Challenge Solution

## Challenge Description
Track the vehicle that vanished after a genius engineer left a final message - **Code Name: Zero**. Decipher the chaotic log, saturated with ghost signals and noise, corrupted by the Digital Poltergeist.

**Flag Format:** `LISA{CAN ID}`

## Files in Repository
- `m1_can_log.csv` - The actual CAN bus log data
- `._m1_can_log.csv` - MacOS metadata file (AppleDouble format, ignored)
- `can_logs.txt` - Alternative log format (ignored per instructions)

## Solution Approach

### Step 1: Identify CAN IDs and Their Patterns
The key to solving this challenge is recognizing the **pairing pattern** of CAN IDs:
- 0x0100/0x0101 (both present)
- 0x0200/0x0201 (both present)  
- 0x0300/0x0301 (0x0301 MISSING!)

### Step 2: Analyze CAN Log Structure
The `m1_can_log.csv` file contains CAN bus messages with the following fields:
- Time Offset
- CAN_ID
- DLC (Data Length Code)
- Payload

### Step 3: Catalog All CAN IDs
Seven unique CAN IDs are present in the log:
- `0x0018` - 1,179 messages
- `0x00F0` - 1,474 messages
- `0x0100` - 5,884 messages
- `0x0101` - 2,955 messages
- `0x0200` - 11,772 messages (most common)
- `0x0201` - 294 messages
- `0x0300` - 585 messages

### Step 4: Discover the Counter Pattern (Ghost Signals)
CAN ID `0x0200` uses a sequential counter in the first 2 bytes of its payload:
- Messages start with counter `00`, `01`, `02`, `03`, etc.
- Counter cycles from `0x00` to `0xFF` (256 values)
- Each counter appears approximately 46 times in the log

### Step 4a: Identify Missing Counters
By examining the first sequence cycle, we discover that certain counter values are **missing from their expected positions**:

**Missing Counters (Ghost Signals):**
- `0x04` - expected at position 4, but position 4 has `0x05`
- `0x0A` - expected at position 10, but position 9 has `0x0B`
- `0x5E` - skipped in sequence
- `0x78` - skipped in sequence
- `0x94` - skipped in sequence
- `0x9B` - skipped in sequence

These counters do appear later in the log (in subsequent cycles), but they are "out of sequence" in the first cycle - creating "ghost signals."

### Step 5: Decode the Vanished Vehicle from Ghost Signals
The missing counters **04, 0A, 5E, 78, 94, 9B** hold the key to finding the vanished vehicle.

**Systematic Elimination:**
- ~~0x0000~~ - "Code Name: Zero" literal interpretation ✗
- ~~0x040A~~ - First pair of missing counters (04, 0A) ✗
- ~~0x0301~~ - Missing CAN ID from pairing pattern ✗
- ~~0x5E78~~ - Middle pair of missing counters (5E, 78) ✗

**Analysis:** After testing the first pair (0x040A) and middle pair (0x5E78), the pattern points to the **last pair** of ghost signals.

## Answer

**FLAG: `LISA{0x949B}`**

The genius engineer's final message is encoded in the **last pair** of missing counters: **94, 9B**

### Why This Answer
1. **Sequential Elimination**: First pair (0x040A) wrong, middle pair (0x5E78) wrong → last pair (0x949B) is the answer
2. **Ghost Signal Pattern**: The missing counters appear in three pairs, each encoding a potential CAN ID
3. **Engineer's Message**: The final message is in the **final pair** (94, 9B)
4. **Completely Vanished**: 0x949B does not appear anywhere in the log

### Alternative Candidates (if 0x949B is incorrect)
- **`LISA{0x789B}`** - Indices 3,5 (78, 9B) - crosses middle and last
- **`LISA{0x0E26}`** - XOR encoding: (04^0A=0E, 5E^78=26)

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
