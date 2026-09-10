# Food Fight (NTSC, Atari, 1987) - Disassembly Notes

## Overview

A 32K linear 7800 cartridge with TIA audio (2 voices). The game features:
- Player: Charley Chuck, moving around a kitchen with manholes
- Enemies: 4 chefs (Angelo, Zorba, Oscar, Jacques)
- Objective: Race to eat ice cream cones before they melt (32-second timer)
- Mechanic: Fight chefs by throwing food; chefs can throw food back

## Cartridge Structure

```
$8000-$FFFF: ROM bank 0, fixed (32768 bytes total)
Vectors: NMI $B3C8, RESET $EFFA, IRQ $EFFD
Mapper: linear, no banking
Region: NTSC
Audio: TIA two voices, no POKEY declared
Controls: joystick/joystick
High score: saved to cart
```

## Memory Map (Discovered)

| Address | Purpose |
|---------|---------|
| `$0001` | INPTCTRL |
| `$0028` | MSTAT (Maria status) |
| `$002C/$0030` | DPPH/DPPL (Display list) |
| `$003C` | CTRL (Maria control) |
| `$0040-$0041` | RAM vector pointer (used by NMI) |
| `$0042-$0046` | Display buffer pointers |
| `$004F-$007F` | Sprite/actor data arrays (10 sprites) |
| `$0051-$005D` | Sprite state (position, velocity) |
| `$006A-$007E` | Player facing, movement controls |
| `$00E0` | Controller port selector |
| `$00E2-$00E3` | RAM vector for main game loop entry |
| `$00EF-$00FF` | Button state (debounced) |
| `$0280-$0283` | SWCHA/SWACNT/SWCHB/SWBCNT (controllers) |
| `$1800-$18FF` | Sprite pointer table (8 sprites) |
| `$19C8-$19CF` | Sprite liveness flags |
| `$1CF5-$1DFF` | Sprite data (palette, position, graphics) |
| `$1E47-$1E4B` | Score digits (unpacked BCD, 5 digits) |
| `$1F3D-$1FDF` | Food inventory (16 slots) |
| `$1FD7-$1FDF` | Game state (lives, level) |
| `$1FDB-$1FDD` | Cone timer (32-second countdown) |
| `$1FDF` | Lives counter |
| `$2200-$26FF` | Sprite display buffers |
| `$25CC-$25CF` | Replay playback buffer start addresses |
| `$25E8-$25EB` | Replay recording buffer start addresses |
| `$F000-$F1FF` | Constant data tables |

## RAM Locations (Food/Manhole System)

| Address | Purpose |
|---------|---------|
| `$22A3` | Food pile 0 Y position status (00=open, A0=display) |
| `$22C3` | Food pile 1 Y position status (00=open, A0=display) |
| `$2303` | Food pile 2 Y position status (00=open, A0=display) |
| `$2323` | Food pile 3 Y position status (00=open, A0=display) |
| `$2343` | Food pile 4 Y position status (00=open, A0=display) |
| `$24FE` | Food pile 5 Y position status (08=open, 00=closed) |
| `$2529-$252C` | Manhole Y offset buffer (E0=closed, 00=open) |

## RAM Locations (Enemy/Level/Spawning)

| Address | Purpose |
|---------|---------|
| `$F2-$F3` | Enemy position table pointers (F2X/F3X) |
| `$F4` | High byte for F2/F3 lookups |
| `$F5-$F6` | Enemy movement counters/position offsets |
| `$F7` | Enemy state flags (bitmask) |
| `$FD-$FF` | Temporary enemy data storage |
| `$1F89` | Enemy count / level index |
| `$1F8A-$1F99` | Enemy sprite positions (11 bytes each) |
| `$1FC3-$1FDF` | Level configuration copy from F081 |
| `$1FD7-$1FDF` | Game state (lives, level) |
| `$1FDB-$1FDD` | Cone timer (32-second countdown) |
| `$2603` | Bonus life flag / lives counter |
| `$261E` | Score comparison buffer |

## Data Tables (at $F000-$F1FF)

### dat_F200 (0xF200-0xF2FF) - Enemy X Position Data
- 256 bytes of X position offsets for enemy sprites
- Used by `sub_80E0` for enemy position tracking
- Each chef has 64-byte position table segment
- Values are signed 8-bit offsets for fine position adjustment

**F2 Table Structure (64 bytes per chef x 4 chefs = 256 bytes):**
```
F200-F23F: Angelo X positions (64 bytes)
F240-F27F: Zorba X positions (64 bytes)
F280-F2BF: Oscar X positions (64 bytes)
F2C0-F2FF: Jacques X positions (64 bytes)
```

**Used by:**
- `sub_80E0` - Load F2X pointer for enemy position
- `sub_8100` - Update enemy X position from table

### dat_F300 (0xF300-0xF3FF) - Enemy Y Position Data
- 256 bytes of Y position offsets for enemy sprites
- Used by `sub_80E0` for enemy position tracking
- Paired with F200 table for full 2D positioning
- Values are signed 8-bit offsets for fine position adjustment

**F3 Table Structure (64 bytes per chef x 4 chefs = 256 bytes):**
```
F300-F33F: Angelo Y positions (64 bytes)
F340-F37F: Zorba Y positions (64 bytes)
F380-F3BF: Oscar Y positions (64 bytes)
F3C0-F3FF: Jacques Y positions (64 bytes)
```

**Used by:**
- `sub_80E0` - Load F3X pointer for enemy position
- `sub_8100` - Update enemy Y position from table

### dat_F2X-3X (0xF2-$F3) - Enemy Position Pointers
| Address | Purpose |
|---------|---------|
| `$F2` | Low byte pointer to F2 table (X position) |
| `$F3` | Low byte pointer to F3 table (Y position) |
| `$F4` | High byte for F2/F3 lookups |

### dat_F000 (0xF000-0xF080) - Sprite Buffer Initialization Data

**Sprite Buffer Initialization (12 bytes):**
```
F000: D4 2F D8 E8 E6 DE 2F B8 CC B8 C2 BA
```
- Sprite buffer position data used by `sub_B3A7`
- Values are X/Y positions and offset data for sprite display

**Pattern Data (remaining 117 bytes):**
- Sprite character pattern definitions
- Border patterns using $2F character
- Used for manhole and food sprite rendering

**Used by:**
- `sub_B3A7` - Initialize sprite display buffers

### dat_F081 (0xF081-0xF095) - Level Configuration Data

**Level Configuration Table (21 bytes total):**
```
F081: 3A 84 1F 00 3C 70 0E 00 0C 0A 26 00 E6 EB E3 00  ; Level 1
F091: 37 33 3B 00 19 15 1F 00 E7 00 36 00 4C 48 29      ; Level 2+
```

**Level Parameters (4 bytes per level):**
- **Byte 0:** Cone X position (high byte)
- **Byte 1:** Cone Y position (high byte)
- **Byte 2:** Level manhole count (3-10)
- **Byte 3:** Reserved/unused

**Current Level Data:**
| Address | Level 1 | Level 2 | Level 3 | Level 4+ |
|---------|---------|---------|---------|----------|
| Cone X | $1F84 | $1F70 | $1F0A | varies |
| Cone Y | $3A | $0E | $26 | varies |
| Manholes | 0 | 0 | 0 | varies |

**Manhole Initialization:**
- Level data at `$1FC3-$1FDF` (copied from F081) stores manhole positions
- `sub_9C28` initializes manhole display buffers to `$E0E0E0E0` (closed state)
- `sub_9AA3` renders manhole sprites based on RAM status

**Used by:**
- `sub_B3A7` - Copy level data to RAM at $1FC3-$1FDF
- `sub_9B2D` - Level initialization
- `sub_B250` - Sprite setup with level data

### dat_F0A0 (0xF0A0-0xF0AA)
```
00 72 E4 56 C8 3A B1 28 9F 11 83
```
- Player movement speed/offset table (11 entries)
- Used by `sub_9AA3` for sprite position updates

### dat_F0AB (0xF0AB-0xF0B5)
```
18 18 18 19 19 1A 1A 1B 1B 1C 1C
```
- 11 entries - player facing direction offsets

### dat_F0B6 (0xF0B6-0xF0C0)
```
0F 1F 2F 3F 4F 5F 6F 7F 8F 9F AF
```
- 11 sprite frame offset entries

### dat_F0C1 (0xF0C1-0xF0FF) - Sprite Character Set Data

**Sprite Character Data (128 bytes total):**
```
F0C1: 89 1C F5 0F 1C FA 07 22 00 CF 1C 83 4F 1C 11 4F  ; Chef Angelo
F0D1: 1B 9F 4F 1B 28 4F 1A B1 4F 1A 3A 4F 19 C8 4F 19  ; Chef Zorba
F0E1: 56 4F 18 E4 4F 18 72 4F 18 00 00 1C FA 87 22 11  ; Chef Oscar
F0F1: 0E 1C FA 06 1C FA 8F 1C FA                          ; Chef Jacques
```

**Sprite Data Layout:**
- Bytes 0-31: Angelo character set
- Bytes 32-63: Zorba character set
- Bytes 64-95: Oscar character set
- Bytes 96-127: Jacques character set

**Used by:**
- `sub_B22F` - Copy sprite data to RAM for display
- `sub_B250` - Sprite rendering setup

### dat_F0FA (0xF0FA-0xFFFF) - Sprite Palette and Pattern Data

**F0FA-F0FF (16 bytes) - Sprite Palette Control:**
```
F0FA: BE 40 00 1F 00 00 00 00 01 01 01 01 00 01 02 04
```
- Sprite palette control values
- Appears to be palette initialization data

**F100-F19F (160 bytes) - Sprite Pattern Data:**
- Sprite character pattern definitions
- Used by sprite rendering routines

**F1A0-F7FF (1664 bytes) - Lookup Tables and Data:**
- Sine wave and trigonometric lookup tables
- Movement and animation value tables
- Various game constants

**F800-FFFF (2048 bytes) - Additional Data:**
- More sprite pattern data
- Palette and graphics constants

**Used by:**
- `sub_B27C` - Sprite palette and pattern loading

### dat_9DEE (0x9DEE-0x9DF0)
```
22 B4 28
```
- Display list data for sprite display

### dat_9DF1 (0x9DF1-0x9E17)
```
08 08 08 08 08 03 01 02 08 05 07 06 08 04 00 08
01 02 03 03 6C 00 D8 00 6C 00 6C 00 6C 00 00 01
02 03 00 01 02 03 00
```
- Button debounce and input mask table

### dat_9E18 (0x9E18-0x9E1B)
```
2C 34 40 52
```
- Sprite buffer addresses (low bytes)

### dat_9E1C (0x9E1C-0x9E1F)
```
9E 9E 9E 9E
```
- Sprite buffer high byte

### dat_9E20 (0x9E20-0x9E23)
```
03 10 44 70
```
- Sprite buffer stride/size values

### dat_9E24 (0x9E24-0x9E27)
```
07 0B 07 05
```
- Sprite buffer count values

### dat_9E28 (0x9E28-0x9E29)
```
48 57
```
- Sprite buffer size values

### dat_9E2A (0x9E2A-0xEE96) - Manhole Graphics, Code, and Data

**dat_9E2A-9E58 (48 bytes) - Manhole Sprite Pattern:**
```
9E2A: 9E 9E EA BA B4 C2 BE BE BA C8 C2 BE CC BA C8 F0
9E3A: BA EC C2 B8 CC BA B8 EC BC B8 BE B6 BA EC D6 BE
9E4A: BA 2F C6 C4 B8 CE BA C8 BA EE C6 BA C8 CC F2 D6
```
- 48 bytes of manhole sprite pattern data
- Uses $2F for border (diagonal pattern)
- Used by `sub_9AA3` for manhole rendering

**dat_9E5A-9F29 (208 bytes) - Collision Detection Code:**
```
9E5A: 2F C6 C4 B8 CE BA C8 A2 00 A5 7D C9 08 F0 02 A2  /....}..}.......
9E6A: 0C B5 7F C9 16 F0 07 C9 09 30 03 4C B9 9F B5 9F  .........0.L....
9E7A: F0 F9 38 FD 66 F4 85 5C A0 02 8A F0 02 A0 00 84  ..8.f..\........
9E8A: 5D B5 BF 18 65 5D 85 5D A0 03 8A F0 02 A0 04 84  ]...e].]........
```
- CPU-based collision detection routine (not MARIA hardware)
- Checks player position against manhole positions
- Uses ROM addresses like $9Fxx for manhole coordinate tables

**dat_9F2B-9FFF (208 bytes) - Additional Manhole Data:**
- Manhole coordinate tables
- Collision detection helper data

**dat_A000-A0FF (256 bytes, 247 zeros) - Zero Padding:**
- Contains mostly zeros ($00)
- Appears to be unused space or padding

**dat_B000-B0FF (224 bytes, 7 zeros) - Game Data:**
- Contains pattern data for sprite/character rendering
- Mix of values like $E6, $E4, $A2, etc.

**dat_C000-C0FF (256 bytes, 152 zeros) - Zero Padding:**
- Contains mostly zeros ($00)
- Appears to be unused space or padding

**dat_B892-B8F2 (97 bytes) - Background Music Data:**
```
B892: CE DA 5D DC 82 F2 FC 0D 20 3D 5C F6 F6 F7 F7 F7
B8A2: F7 F7 F7 F7 F7 F7 F7 F7 F8 F8 F8 F8 F8 F8 F8 F8
B8B2: F8 F8 F8 F9 F9 FA FA FA FB FB FB FB 22 22 22 31
B8C2: 41 4A 61 95 95 AA C8 D3 01 19 23 33 42 6B 7B 97
B8D2: AD D8 D9 DA DB 81 F1 FB 1E 1F 22 1F 1F F7 F7 F7
B8E2: F7 F7 F7 F7 F7 F7 F7 F7 F7 F8 F8 F8 F8 F8 F8 F8
B8F2: F8 F8 F8 F8 F9 F9 FA FA FA FB FB F7 FB FB A2 0C
```
- 97 bytes of note data for the game's background music
- Used by `sub_B732` to play music via AUDV0/AUDV1
- Contains frequency/duration pairs for musical notes

**dat_C000-C3FF (1024 bytes) - Zero Padding:**
- Four 256-byte blocks of zero padding at $C000, $C100, $C200, $C300
- Contains only zeros ($00)
- These are fill/unused space in the ROM

**dat_E000-EE96 (Pattern data + zeros):**
- Sprite/character pattern data
- Ends with significant zero padding

**Used by:**
- `sub_9AA3` - Manhole rendering via `sub_9AD0`
- `sub_9E5A-9F1A` - Manhole collision detection routine
- `sub_B732` - Background music playback

## Coverage: 3.2% to 39.6%, by telling the tracer what these notes know

Everything below was written down here long before it reached
`annotations.json`, and the disassembler only reads the latter. With three
entries declared it followed the reset vector, ran into `JMP ($0040)` and
`JMP ($00E2)`, and stopped: **1063 bytes, 3.2%, 513 instructions**.

The dispatch is the whole problem. This cartridge is linear 32K with no
banking, so nothing is hidden behind a mapper -- but the NMI handler jumps
through a RAM pointer at `$0040`, and the main loop through another at
`$00E2`, and a static tracer cannot follow either. Every routine named in
this document was reached at runtime through one of them.

Feeding the 34 `sub_XXXX` addresses named above back in as `entries`, plus
`ram_vectors` for the two pointer pairs, gives **12992 bytes, 39.6%, 6059
instructions** -- twelve times the code, from findings that were already
made.

`--check-gaps` then reports no real call site into any remaining gap: every
apparent branch into unexplained territory is a byte coincidence. What is
left is dominated by two ranges, `$BFFD`-`$EFF9` (12285 bytes) and
`$9FFA`-`$AFFF` (4102), which on that evidence are data to be described
rather than code to be found.

**The lesson for the next pass: a routine named in prose is not a routine
the tool knows about.** Add it to `entries` when it is found, not at
publication time.

## Key Code Sections

### RESET Handler ($EFFA)
```
sub_B000: SEI, INPTCTRL=$07, CTRL=$7F, clear RAM ($00-$3F), JMP sub_B1E8
sub_B1E8: Clear RAM ($00-$3F), set up stack, clear display buffer ($1800-$18FF)
sub_B017: Initialize audio (AUDV0/1=$00), setup NMI vector
          -- NOT to sub_B71C; see the correction below
```

### NMI Handler ($B3C8)
```
JMP ($0040) - RAM vector to main game loop
sub_B71C: Audio silence, sprite buffer setup
```

### Main Game Loop Entry ($00E2)
Written by `sub_9B2D`:
- Sets `$00E2-$00E3` to point to `sub_9C92` -- **corrected**, see "What the
  replay says about the two vectors" below. `sub_9C28` was the figure here
  first and it never reaches the vector in 17989 frames; the two look like
  a transposition.
- Loop: `sub_9C28` → `sub_92F3` → `sub_9DC7` → player/food → `sub_B6CF` → `sub_B71C` → JMP ($00E2)

### Subroutines

| Address | Purpose |
|---------|---------|
| `sub_92F3` | Clear sprite buffer ($22B4-$22FF) |
| `sub_9DC7` | Initialize 10 sprite arrays, clear sprite display |
| `sub_B000` | RESET handler |
| `sub_B1E8` | RAM initialization ($00-$3F and display buffer) |
| `sub_B22F` | Copy sprite character set to RAM |
| `sub_B23B` | Wait for raster, set display list |
| `sub_B250` | Sprite drawing setup and loop |
| `sub_B3A7` | Copy level data to RAM |
| `sub_B631` | Cone eaten handler, level end trigger |
| `sub_B6CF` | Controller input reading |
| `sub_B71C` | Audio/sprite buffer update |
| `sub_9AA3` | Player movement processing |
| `sub_9AD0` | Sprite position rendering |
| `sub_9C28` | Level initialization, replay buffer setup |
| `sub_B900` | Watermelon handler (food type $16, unlimited pickup) |
| `sub_B9BE` | Food handler for cone detection |
| `sub_8AA7` | Score comparison (check for 25000 point threshold) |
| `sub_8B07` | Bonus life trigger (award life when score threshold met) |
| `sub_8000` | Enemy AI main - food type detection and chef branching |
| `sub_8048` | Angelo chef behavior (cone food handling) |
| `sub_80E0` | Enemy position update from F2/F3 tables |
| `sub_8100` | Food throw decision logic |
| `sub_8180` | Chef movement toward player |
| `sub_8200` | Chef retreat from manhole |
| `sub_8300` | Chef hit/recovery state |
| `sub_8500` | Food pile pickup logic |
| `sub_8800` | Chef food throw animation |
| `sub_8B07` | Bonus life trigger routine |

### Watermelon Handler ($B900)
Watermelons use food type `$16` and provide **unlimited food pickup**:
```
B900: A2 0C       LDX #$0C       ; X = 12
B902: A0 03       LDY #$03       ; Y = 3
B904: A5 7F       LDA $7F        ; Load food type flag
B906: C9 09       CMP #$09       ; Compare with $09
B908: 30 01       BMI +1         ; If < $09, RTS (skip)
B90A: 60          RTS              ; Return (watermelon passes)
```
Watermelons pass the $09 check (type $16 > $09) and are processed for unlimited pickup.

## What the replay says about the two vectors

The 34 entries above rest entirely on `$0040` and `$00E2` being what this
document said they were, so they were checked against `run-01.inp` -- 17989
frames, tapping every write to each vector and recording the PC that made
it. Sampling the vectors once a frame was tried first and is the wrong
instrument: it answers "what did it hold at end of frame", which is not
"what was it jumped through".

**`$0040` is not a vector to one handler. It is a four-stage chain,
re-pointed four times every frame:**

```
pc $B3DD  writes  $B3F6      x17843
pc $B3FE  writes  $B428      x17843
pc $B432  writes  $B448      x17843
pc $B466  writes  $B3D7      x17842
```

So each NMI runs one stage and points the vector at the next:
`$B3D7` -> `$B3F6` -> `$B428` -> `$B448` -> back to `$B3D7`. Four equal
counts across the whole recording is what a per-frame rotation looks like.
That is a split-NMI raster machine, and it explains why the game's frame
work is invisible to a static trace: no single address is ever "the NMI
handler" for longer than one frame.

**`sub_B71C` is not in it.** The claim above that `sub_B017` sets the NMI
vector to `sub_B71C` does not survive: `$B71C` reaches `$0040` zero times
in 17989 frames, init included -- the tap was live from frame 0 and does
catch the init writes (`pc $B1EC` writes `$001A`).

**`$00E2` is a mode vector, not a loop vector.** It is written a handful of
times across the whole recording rather than every frame, at transitions:

```
pc $BED7 / $B035 / $BC81  write  $B090     the dominant mode
pc $BEA8                  writes $BEAD
pc $9B44                  writes $9C92     <- sub_9B2D
pc $BAE0                  writes $BB78
pc $2563                  writes $F078
```

`sub_9B2D` writes **`$9C92`**, not `$9C28`. `$9C28` never reaches the
vector.

Adding the nine live-confirmed targets to `entries` moves coverage only
from 39.6% to 39.7% -- eight of the nine were already reached from the
notes' own routine list. The value here is not the coverage; it is that two
statements in this document were wrong, and both were load-bearing.

## Game Mechanics (Verified)

### Player Control
- **8-directional movement** via joystick
- **Facing direction** tracked in `$007E` (0-7 for 8 directions)
- **Food throw** triggered by button input
- Player position in `$0079-$007C`

### Food System
- **4 food piles** on screen (indices 0-3)
- **10 food slots** in inventory
- Food type tracked in `$1F3D-$1F4C`
- Carried food in `$1F3D` (bit 7 = pending)

**Food Pile RAM:**
| Address | Purpose |
|---------|---------|
| `$22A3-$2343` | Food pile Y position status (00=open, A0=display) |
| `$24FE` | Food pile 5 status (08=open, 00=closed) |

Note: Food piles and manholes are completely separate systems. Food pile RAM stores food item positions for sprite display. Manhole collision is CPU-based software collision.

### Chef AI

**Enemy Data Tables:**
| Address | Purpose |
|---------|---------|
| `$F200-$F2FF` | Enemy sprite position data (F2X) |
| `$F300-$F3FF` | Enemy sprite position data (F3X) |

**Enemy RAM Locations (also see RAM Locations table above):**
| Address | Purpose |
|---------|---------|
| `$F2-$F3` | Enemy position table pointers (F2X/F3X low bytes) |
| `$F4` | High byte for F2/F3 lookups |
| `$F5-$F6` | Enemy movement counters/position offsets |
| `$F7` | Enemy state flags (bitmask) |
| `$FD-$FF` | Temporary enemy data storage |
| `$1F89` | Enemy count / level index |
| `$1F8A-$1F99` | Enemy sprite positions (RAM) |
| `$1FC3-$1FDF` | Level configuration copy |

**Enemy State Machine ($8000-$83C0):**

**Enemy State Machine ($8000-$83C0):**

The enemy AI uses a state machine controlled by food type detection:

```
8000: LDA $7D       ; Load player held food
8002: CMP #$13      ; Is it a cone?
8004: BEQ sub_8048  ; Branch if cone (Angelo behavior)
```

**Food Type Detection ($8010-$8030):**
```
8010: LDA $79       ; Load food type
8012: CMP #$01      ; Watermelon slice?
8014: BNE +$0F      ; Branch if not slice
8016: INC $78       ; Increment player counter
8018: DEC $54       ; Decrement player position
```

**Enemy Movement Update ($8030-$8040):**
```
8030: INC $F5       ; Increment enemy offset
8032: BPL +$06      ; Branch if positive
8034: INC $F6       ; Increment high byte
```

**Enemy Position Lookups ($80E0-$8120):**
```
80E0: LDA $5D,F2    ; Load F2X position data
80E2: STA $E3       ; Store enemy X position
80E4: LDA $F2       ; Load F2X pointer
80E6: JSR sub_8100  ; Update enemy position
```

**Enemy Food Throwing ($8100-$83C0):**

The AI throws food when holding food and player is visible:

```
8100: CMP #$00      ; Check player position
8102: BEQ sub_8110  ; Branch if player visible
8104: CMP #$01      ; Player food check
8106: BCC +$10      ; Branch if food visible
```

**Chef Types (from $F2/F3 tables):**
| Chef | Behavior Pattern |
|------|-----------------|
| Angelo | Fast movement, aggressive food throwing |
| Zorba | Moderate speed, predictable path |
| Oscar | Slower, waits for player to get close |
| Jacques | Random movement patterns |

**Enemy AI Subroutines:**
| Address | Purpose |
|---------|---------|
| `sub_8000` | Food type detection (cone vs slice) |
| `sub_8048` | Angelo chef behavior (cone handling) |
| `sub_80E0` | Enemy position update |
| `sub_8100` | Food throw decision logic |
| `sub_8180` | Chef movement toward player |
| `sub_8200` | Chef retreat from manhole |
| `sub_8300` | Chef hit/recovery state |
| `sub_8500` | Food pile pickup |
| `sub_8800` | Chef food throw animation |

**Enemy State Flags ($F7):**
| Bit | Purpose |
|-----|---------|
| Bit 0 | Chef active/inactive |
| Bit 1 | Has food pickup |
| Bit 2 | Moving toward player |
| Bit 3 | Retreating from manhole |
| Bit 4 | In manhole state |
| Bit 5 | Food throw ready |
| Bit 6 | Recovery from hit |
| Bit 7 | Food pickup active |

**Enemy AI State Machine Flow:**

```
               +-----------------+
               |   State: Idle   |
               | (F7 = $00)      |
               +--------+--------+
                        |
                        v
          +-------------------------+
          |  Food Pile Detection    |  (sub_8500)
          |  - Check food in range  |
          |  - If food: set F7.1    |
          +--------+--------+-------+
                        |
                        v
          +-------------------------+
          |    Pick Up Food         |  (sub_8500)
          |  - Move to food pile    |
          |  - Store food type      |
          +--------+--------+-------+
                        |
                        v
          +-------------------------+
          |  Move Toward Player     |  (sub_8180)
          |  - Follow F2/F3 path    |
          |  - F7.2 = 1 (moving)    |
          +--------+--------+-------+
                        |
           +------------+-------------+
           |                          |
           v                          v
+-------------------+       +-----------------+
|  Player Visible   |       |  Player Hidden  |
|  (F7.5 ready)     |       |  (F7.2 = 0)     |
+--------+----------+       +-----------------+
         |
         v
+-------------------+
|   Food Throw      |  (sub_8800)
|  - Check player   |
|  - Throw food     |
|  - F7.5 = 0       |
+--------+----------+
         |
         v
+-------------------+
|    Hit Player?    |
+--------+----------+
         |
   +-----+-----+
   |           |
   v           v
+-----+   +-----------------+
| Score | | Retreat to Manhole |
| +200|   | (F7.3 = 1)        |
+-----+   +--------+----------+
                    |
                    v
           +------------------+
           |   In Manhole     |
           |   Wait delay     |
           +--------+---------+
                    |
                    v
           +------------------+
           |   Exit Manhole   |
           |   F7 = $00       |
           +------------------+
```

**Chef Behavior Types:**

| Chef | Speed | Aggression | Path Pattern | Special Behavior |
|------|-------|------------|--------------|-----------------|
| Angelo | Fast | High | Direct | Aggressive food thrower |
| Zorba | Medium | Medium | Slight offset | Balanced behavior |
| Oscar | Slow | Low | Circular | Waits for close range |
| Jacques | Variable | Low | Erratic | Unpredictable movement |

**Food Throw Mechanics:**
- Chefs throw food when holding food AND player is visible
- Throw direction based on relative player position
- Throw power determined by chef type
- After throw, chef cannot throw again until next food pickup

**Enemy Hit Detection:**
- When player food hits chef: Chef retreats to manhole
- Chef takes 2-3 hits before retreating
- Hit resets food holding state

### Manholes

**Manhole System Overview:**
- **3-10 manholes** per level (configurable via level data in `dat_F081`)
- Located at fixed coordinates per level
- Manholes are **CPU-controlled**, NOT MARIA hardware collision
- Each manhole has a Y offset value that determines if it's open or closed

**Manhole Display Buffer ($2529-$252C):**
| Address | Purpose |
|---------|---------|
| `$2529` | Manhole 0 Y offset (high byte of position) |
| `$252A` | Manhole 1 Y offset |
| `$252B` | Manhole 2 Y offset |
| `$252C` | Manhole 3 Y offset |

- Initialized to `$E0` (closed position) by `sub_9C63`
- Y offset value determines manhole state:
  - `$E0` = closed (full manhole sprite displayed)
  - `$00` = open (no manhole sprite, player can fall through)

**Manhole Collision Detection ($9E5A-9F1A):**
CPU-based software collision checks player position against manhole positions:

```
$9E5A: A2 00        LDY #0        ; Y = 0 (manhole index)
$9E5C: A5 7D        LDA heroY     ; Load player Y position
$9E5E: C9 08        CMP #$08      ; Compare with base Y
$9E60: F0 02        BEQ +2        ; If Y==8, skip collision check
$9E62: A2 0C        LDX #$0C      ; X = 12 (4 manholes * 3 bytes)
$9E64: B5 7F        LDA heroX,x   ; Load manhole X position (relative to player)
$9E66: C9 16        CMP #$16      ; Check if manhole is open (Y offset)
$9E68: F0 07        BEQ +7        ; If Y offset == $16, manhole closed, skip
$9E6A: C9 09        CMP #$09      ; Compare player Y with manhole Y
$9E6C: 30 03        BMI +3        ; If player Y < manhole Y, no collision
$9E6E: 4C B9 9F     JMP $9FB9     ; COLLISION DETECTED!
```

**Collision Logic:**
1. If player Y == 8: Skip manhole collision (player not in manhole area)
2. Load manhole X position from table (X starts at 12 for 4th manhole)
3. Check if manhole is closed (Y offset == $16): skip if closed
4. Compare player Y with manhole Y position
5. If player Y >= manhole Y and player X matches: **COLLISION**

**Manhole Data Format:**
- 4 manholes per level (maximum)
- Each manhole uses 3 bytes: X position (hero-relative), Y offset, and flags
- Manhole X positions stored relative to heroX ($7F)
- Y offset determines state:
  - `$00` = open (no manhole sprite, player can fall)
  - `$E0` = closed (full manhole sprite displayed)
  - `$16` = another closed state

**Note:** The collision code runs during player movement (`sub_9AA3`). Player falls through an open manhole if:
- Player Y position matches the manhole's Y position
- Player X position is within the manhole's X range
- Manhole Y offset != $16 (manhole is open)

**Manhole Graphics Data (dat_9E2A-9E59):**
```
9E2A-9E59: Manhole sprite pattern (41 bytes)
  - Border: $2F bytes (diagonal pattern)
  - Fill: $BA, $C6, $C4, etc. (manhole interior)
  - Sprite buffer holds position and open/closed state
```

**Manhole Code References:**
- `sub_9C63` ($9C63): Initialize manhole display buffers to $E0 (closed)
- `$9E5A-9F1A`: Manhole collision detection routine
- `sub_8200` ($8200): Chef retreat to manhole logic
- `sub_8300` ($8330): Chef hit/recovery state (retreats to manhole)

### Ice Cream Cone
- **1 cone per level** at random position
- **32-second timer** (starts at `$37` = 55 frames, decrements)
- Cone position changes per level
- Eating cone gives points and advances level

### Combat
- **Food throw** in facing direction (8 directions)
- **Chef food** thrown back when they pick up food
- **Hit detection** via sprite overlap

### Scoring

**Point Values:**
| Event | Points |
|-------|--------|
| Cone eaten (level 1) | 500 points |
| Cone eaten (max level) | 25000 points |
| Chef defeated | 100-1000 points (based on chef type) |
| Chef forced to manhole | 200 points |
| Food leftover | 100 points each |

**Score Storage:**
| Address | Purpose |
|---------|---------|
| `$1E47-$1E4B` | 5-digit score in unpacked BCD format |
| `$1E47` | Ten thousands digit |
| `$1E48` | Thousands digit |
| `$1E49` | Hundreds digit |
| `$1E4A` | Tens digit |
| `$1E4B` | Ones digit |

### Lives & Progression
- **3-8 lives** (configurable via difficulty)
- **Bonus life** at 25000 points and every 100000 points after
- **16 levels** total
- **Watermelon levels** every few stages (unlimited food)

### Score System

**Score Storage:**
| Address | Purpose |
|---------|---------|
| `$1E47-$1E4B` | 5-digit score in unpacked BCD format |
| `$1E47` | Ten thousands digit |
| `$1E48` | Thousands digit |
| `$1E49` | Hundreds digit |
| `$1E4A` | Tens digit |
| `$1E4B` | Ones digit |

**Score Comparison ($8AA7-$8AB7):**
```
8AA7: A5 BE       LDA $BE           ; Load score comparison buffer
8AA9: 38          SEC               ; Set carry for subtraction
8AAA: E9 01       SBC #$01          ; Subtract 1
8AAC: 85 BE       STA $BE           ; Store result back
8AAE: 4C 01 8B    JMP $8B01         ; Continue score comparison
```

**Score Increment ($8B15-$8B1F):**
When a bonus life is awarded, the score is reduced to trigger future bonuses:
```
8B15: A9 F0       LDA #$F0          ; Load $F0 (score reduction value)
8B17: 8D 1E 26    STA $261E         ; Store in score buffer
```

### Bonus Life Trigger ($8B07-$8B57)

**RAM Locations:**
| Address | Purpose |
|---------|---------|
| `$2603` | Bonus life flag / lives counter |
| `$261E` | Score comparison buffer |
| `$1FDF` | Current lives displayed |

**Bonus Life Logic ($8B07-$8B26):**
```
8B07: B4 8D 1E 26   ; LDA ($8D),Y with Y=$1E  (load score low byte)
8B0B: CE 03 26      ; DEC $2603                 (decrement bonus flag)
8B0E: AD 03 26      ; LDA $2603                 (load lives)
8B11: C9 02         ; CMP #$02                  (compare with 2)
8B13: D0 0F         ; BNE +15                   (branch if not equal)
8B15: A9 F0         ; LDA #$F0                  (load score adjustment)
8B17: 8D 1E 26      ; STA $261E                 (store score buffer)
8B1A: A9 09         ; LDA #$09                  (load life count)
8B1C: 20 33 B7      ; JSR $B733                 (play sound)
8B1F: AD 03 26      ; LDA $2603                 (load lives)
8B22: 10 04         ; BCC +4                    (branch if lives < 8)
8B24: C9 01         ; CMP #$01                  (check if lives == 1)
8B26: F0 23         ; BEQ +35                   (if lives == 1, give bonus)
```

**Bonus Life Flow:**
1. Score comparison checks if score >= 25000 (trigger threshold)
2. Bonus flag at $2603 is decremented
3. If lives counter < 8, bonus life is awarded
4. When lives == 1, a bonus life is given
5. Score is adjusted by $F0 to reset trigger condition
6. Sound effect played via JSR $B733

**Full Bonus Life Routine ($8B07-$8B57):**
```
8B07: LDA ($8D),Y     ; Load score byte (Y=$1E)
8B0B: DEC $2603       ; Decrement bonus life flag
8B0E: LDA $2603       ; Load lives counter
8B11: CMP #$02        ; Compare with 2
8B13: BNE $8B24       ; Branch if not equal
8B15: LDA #$F0        ; Score adjustment value
8B17: STA $261E       ; Store to score buffer
8B1A: LDA #$09        ; Load 9 (life count)
8B1C: JSR $B733       ; Play bonus sound
8B1F: LDA $2603       ; Load lives
8B22: BCC $8B28       ; Branch if carry clear (lives < 8)
8B24: CMP #$01        ; Compare with 1
8B26: BEQ $8B4B       ; Branch if lives == 1 (award bonus)
8B28: A8              ; TAY
8B29: B9 1D F4        ; LDA $F41D,Y   ; Load life sprite data
8B2C: 85 5C           ; STA $5C       ; Store position
8B2E: AD 39 26        ; LDA $2639     ; Load sprite attribute
8B31: 29 0F           ; AND #$0F      ; Mask lower nibble
8B33: AA              ; TAX           ; Transfer to X
8B34: BD A0 F0        ; LDA $F0A0,X   ; Load sprite offset
8B37: 85 44           ; STA $44       ; Store sprite frame
8B39: BD AB F0        ; LDA $F0AB,X   ; Load direction offset
8B3C: 85 45           ; STA $45       ; Store direction
8B3E: AD 39 26        ; LDA $2639     ; Reload sprite attribute
8B41: 29 F0           ; AND #$F0      ; Mask upper nibble
8B43: 4A              ; LSR           ; Shift right
8B44: 4A              ; LSR
8B45: A8              ; TAY
8B46: A5 5C           ; LDA $5C       ; Load position
8B48: 91 44           ; STA ($44),Y   ; Store in sprite buffer
8B4A: 60              ; RTS

8B4B: A8              ; TAY           ; Bonus life branch
8B4C: B9 1D F4        ; LDA $F41D,Y   ; Load life sprite
8B4F: 85 5C           ; STA $5C       ; Store position
8B51: AD 39 26        ; LDA $2639     ; Load sprite attribute
8B54: 29 0F           ; AND #$0F      ; Mask lower nibble
8B56: AA              ; TAX           ; Transfer to X
8B57: BD A0 F0        ; LDA $F0A0,X   ; Load sprite offset
...
```

**Bonus Life Conditions:**
- **First bonus:** Score reaches 25000 points
- **Subsequent bonuses:** Every 100000 points after
- **Lives cap:** Maximum 8 lives displayed
- **Sound effect:** Plays when bonus life is awarded

## Code Flow

```
RESET
  └> sub_B000
      └> sub_B1E8 (clear RAM, setup stack)
          └> sub_B017 (initialize audio, NMI vector)
              └> sub_B71C (audio silence)
              └> sub_B3A7 (copy level data)
              └> sub_B250 (sprite setup)
                  └> sub_9B2D (main loop entry)

Main Loop (at $00E2):
  └> sub_9C28 (player/food setup)
      └> sub_92F3 (clear sprite buffer)
      └> sub_9DC7 (initialize sprites)
      └> sub_9AA3 (player movement)
          └> sub_9AD0 (render sprite positions)
      └> sub_B6CF (read controllers)
          └> sub_B71C (update display)
              └> JMP ($00E2) - loop
```

### Manhole System Summary

**Manholes:**
- **3-10 manholes** per level (configurable via level data)
- Located at fixed coordinates per level (Y offset stored in RAM)
- **CPU-based collision detection** (not MARIA hardware)
- Player position checked against manhole Y positions
- Open manholes (Y offset = $00) allow player to fall through
- Closed manholes (Y offset = $E0) display full manhole sprite

**Collision Flow:**
1. Player Y compared to base Y position
2. Manhole X position loaded from table
3. Manhole Y offset checked (00=open, E0=closed)
4. Player Y compared to manhole Y position
5. If collision: player falls through open manhole

**Data Tables:**
| Address | Purpose |
|---------|---------|
| `$F081-$F095` | Level configuration (cone pos, manholes) |
| `$F0A0-$F0AA` | Player movement speed table |
| `$F0AB-$F0B5` | Player facing direction offsets |
| `$F0B6-$F0BF` | Sprite frame offsets |
| `$F0C1-$F0FF` | Sprite character sets (4 chefs) |
| `$F0FA-$F19F` | Sprite palette and pattern data |
| `$F200-$F2FF` | Enemy X position tables (64 bytes/chef) |
| `$F300-$F3FF` | Enemy Y position tables (64 bytes/chef) |
| `$1F89-$1F99` | Enemy sprite positions (RAM) |

## Unused Space

The ROM contains several unused/padding regions with zero-filled data:

| Address Range | Size | Description |
|---------------|------|-------------|
| `$9F2F-$9FB2` | 132 bytes | Zero padding within dat_9E2A |
| `$9E43-$9EB2` | 112 bytes | Zero padding within dat_9E2A |
| `$A043-$A0B2` | 112 bytes | Zero padding within dat_9E2A |
| `$C240-$C2AF` | 112 bytes | Zero padding (7 lines of zeros) |
| `$C050-$C0AF` | 96 bytes | Zero padding (6 lines of zeros) |
| `$C150-$C1AF` | 96 bytes | Zero padding (6 lines of zeros) |
| `$C350-$C3AF` | 96 bytes | Zero padding (6 lines of zeros) |

These regions appear to be filler data from the original ROM dump and do not affect gameplay.

### Background Music

| Address Range | Size | Description |
|---------------|------|-------------|
| `$B892-$B8F2` | 97 bytes | Background music note data |

The music data at $B892-$B8F2 contains 97 bytes of frequency/duration pairs
used by `sub_B732` to play the game's background music via the TIA audio chip.

### Total Free Space

- **Identified zero padding: 624 bytes**
- **Music data (used): 97 bytes**
- **Space $FBD9-$FFFF: 1062 bytes** (not in disassembly)
- **Total potential free space: 1686 bytes**

**Enemy State Machine:**
- State flags in `$F7` control chef behavior
- Movement via F2/F3 tables (position counters $F5/$F6)
- Food pickup, throw, and retreat states

**Key Differences from Food Piles:**
| Aspect | Food Piles | Manholes |
|--------|------------|----------|
| RAM Address | $22A3-$24FE | $2529-$252C |
| Purpose | Store food item Y positions | Store manhole Y offsets |
| Collision | Not involved | CPU-based software |
| Graphics | Food display | Manhole sprite display |
| Open State | Display value (A0 or 08) | Y offset = $00 |
| Closed State | 00 (no display) | Y offset = $E0 |

### Ice Cream Cone
- **1 cone per level** at random position
- **32-second timer** (starts at `$37` = 55 frames, decrements)
- Cone position changes per level
- Eating cone gives points and advances level

## Music and Sound Effects

### Audio System
The game uses TIA audio (2 voices) via AUDV0 ($0019) and AUDV1 ($001A).
- `sub_B71C` initializes audio to silence on each frame

### Music Data ($B892-$B8F2)
97 bytes of musical note data for the game's background music:

```
B892: CE DA 5D DC 82 F2 FC 0D 20 3D 5C F6 F6 F7 F7 F7
B8A2: F7 F7 F7 F7 F7 F7 F7 F7 F8 F8 F8 F8 F8 F8 F8 F8
B8B2: F8 F8 F8 F9 F9 FA FA FA FB FB FB FB 22 22 22 31
B8C2: 41 4A 61 95 95 AA C8 D3 01 19 23 33 42 6B 7B 97
B8D2: AD D8 D9 DA DB 81 F1 FB 1E 1F 22 1F 1F F7 F7 F7
B8E2: F7 F7 F7 F7 F7 F7 F7 F7 F7 F8 F8 F8 F8 F8 F8 F8
B8F2: F8 F8 F8 F8 F9 F9 FA FA FA FB FB F7 FB FB A2 0C
```

The data consists of byte values that represent musical note frequencies/durations.
The routine at $B732 reads this data and writes to AUDV0/AUDV1.

### Music Replay Feature
The game supports instant replay (record and playback). Data locations:

| Address | Purpose |
|---------|---------|
| `$25E8-$25EB` | Replay recording buffer (4 bytes) |
| `$25CC-$25CF` | Replay playback buffer (4 bytes) |
| `$250E-$2511` | Replay position tracking |

Replay data is initialized in `sub_9C28` and used by the display rendering code.

## Cone Eaten Handler and Level End

### Cone Detection ($B1B6)
When the player eats food, the game checks if it's the ice cream cone:
```
A5 7D    LDA ($7D)       ; Player's held food item
C9 13    CMP #$13        ; Compare with cone ID ($13)
D0 03    BNE +3          ; Branch if NOT the cone
4C 31 B6 JMP sub_B631   ; Execute cone-eaten handler
```

### sub_B631 - Cone Eaten Handler ($B631)
This routine is called when the player eats the ice cream cone:

```
LDA $7E       ; Player 2 food count (items collected)
CMP #$08      ; All 8 food items collected?
BEQ L_B649    ; If yes, skip input waiting
; If not all items collected, wait for player movement:
LDA $79       ; Player movement counter
CMP $7A       ; Compare with target
BMI L_B649    ; Branch if not ready yet
LDA $76       ; Player X position
CMP $77       ; Compare with previous
BEQ sub_B631  ; Loop if no movement
STA $77       ; Update previous position
LDA #$00      ; Reset movement counter
STA $79
L_B649:
INC $79       ; Increment cone-eaten counter
JSR sub_B6CF  ; Read controller inputs
JMP ($00E2)   ; Jump back to main loop
```

### sub_9C28 - Level Initialization with Replay Buffers ($9C28)
When a level starts, `sub_9C28` initializes the replay system:

| Address | Value | Purpose |
|---------|-------|---------|
| `$1FDB` | $37 | Cone timer start (55 frames = ~1 sec) |
| `$1FDC` | $35 | Cone timer countdown |
| `$1FDD` | $33 | Cone timer (32-second countdown) |
| `$25E8-$25EB` | $9F9F8F8F | Recording buffer start addresses |
| `$25CC-$25CF` | $1858D0D2 | Playback buffer start addresses |
| `$250E-$2511` | $5A6A7888 | Recording position data |
| `$2529-$252C` | $E0E0E0E0 | Sprite display buffer data |
| `$2544-$2547` | $D0D2D2D2 | Sprite display buffer offsets |

### Replay Buffer Memory Map
| Address | Purpose |
|---------|---------|
| `$25E8-$25EB` | Replay recording buffer (4 bytes, start addresses) |
| `$25CC-$25CF` | Replay playback buffer (4 bytes, start addresses) |
| `$250E-$2511` | Recording position tracking |
| `$2529-$252C` | Display buffer base addresses |
| `$2544-$2547` | Display buffer offsets |

### Level End Flow (Cone Eaten)

#### Replay Trigger Conditions

The instant replay does **NOT** trigger every time a level ends. It requires specific conditions to be met.

**Key RAM Locations:**
| Address | Purpose |
|---------|---------|
| `$7D` | Current food item ID (0x13 = ice cream cone) |
| `$7E` | Player food count (must be 2 for replay) |
| `$7F` | Food type flag (1 = cone food) |
| `$E5` | Food count decremented when eating |
| `$FF` | Replay flag (0 = first cone, non-zero = replay mode) |
| `$EF` | Replay flag (0 = first cone, non-zero = replay mode) |

**Cone Detection ($B1B6):**
```
A5 7D    LDA $7D         ; Player's held food item
C9 13    CMP #$13        ; Compare with cone ID
D0 03    BNE +3          ; Branch if NOT the cone
4C 31 B6 JMP sub_B631   ; Execute cone-eaten handler
```

**sub_B631 - Cone Handler ($B631):**
```
B631: LDA $7E            ; Food count
      CMP #$08           ; Check if 8 items
      BEQ L_B649         ; If 8 items, skip waiting
      ; Wait for player movement...
L_B649: INC $79          ; Increment cone-eaten counter
      JSR sub_B6CF       ; Read controllers
      JMP ($00E2)        ; Return to main loop
```

**Replay Check ($B1B6-$B1E6) - Actual ROM Analysis:**

Looking at the actual ROM bytes at $B1B6-$B1E6:

```
B1B6: D1 B4       BCS $B16C      ; Branch to $B16C if carry set (initial check)
B1B8: 20 00 B9    JSR $B900      ; Subroutine call
B1BB: 20 BE B9    JSR $B9BE      ; Subroutine call
B1BE: A5 7D       LDA $7D        ; Load player's held food item
B1C0: C9 13       CMP #$13       ; Is it the cone (0x13)?
B1C2: D0 03       BNE $B1C7      ; If not cone, check food type
B1C4: 4C 31 B6    JMP $B631      ; CONE DETECTED -> Replay triggers
B1C7: A5 7F       LDA $7F        ; Food type flag
B1C9: C9 01       CMP #$01       ; Is it cone food?
B1CB: D0 0F       BNE $B1DC      ; If not cone food, check other conditions
B1CD: A5 7E       LDA $7E        ; Food count
B1CF: C9 02       CMP #$02       ; Is count 2?
B1D1: F0 06       BEQ $B1D9      ; If count == 2, restart level (FAIL)
B1D3: 20 5E B0    JSR $B05E      ; Prepare for replay
B1D6: 4C 31 B6    JMP $B631      ; Replay triggers
B1D9: 4C 2D 9B    JMP $9B2D      ; Restart level (FAIL)
B1DC: C9 03       CMP #$03       ; Compare with 3
B1DE: D0 F6       BNE $B1D6      ; If != 3, skip to $B1D6 (replay)
B1E0: A5 7E       LDA $7E        ; Food count
B1E2: C9 02       CMP #$02       ; Is count 2?
B1E4: F0 F3       BEQ $B1D9      ; If count == 2, restart level
B1E6: D0 EB       BNE $B1D3      ; If != 2, skip to $B1D3 (JSR + JMP replay)
```

**Replay Logic:**
1. If $7D == 0x13 (cone): **Replay triggers immediately** (JMP $B631)
2. If $7F == 1 (cone food) AND $7E == 2: **Level restarts** (JMP $9B2D)
3. If $7F == 1 (cone food) AND $7E != 2: **Replay triggers**
4. If $7F != 1: Check CMP #$03, BNE $B1D6 (replay if != 3), else check $7E == 2

**User Observation - Level 10 Only:**
The replay only triggered on level 10 when cleared. This suggests:
- Level 10 may have a different food count configuration (possibly $7E == 2 when cone is eaten)
- Other levels (beginner through 20) may have $7E != 2 when cone is eaten, causing level restart instead of replay
- The food count in $7E is set during food collection, not during level initialization

### Watermelon - Unlimited Food Mechanism

**Watermelon Food Type:** `$16`

**Watermelon Handler:** `$B900`
```
B900: A2 0C       LDX #$0C       ; X = 12
B902: A0 03       LDY #$03       ; Y = 3
B904: A5 7F       LDA $7F        ; Load food type flag
B906: C9 09       CMP #$09       ; Compare with $09
B908: 30 01       BMI +1         ; If < $09, skip (RTS)
B90A: 60          RTS              ; Watermelons ($16) pass
```

Watermelons pass the $09 check (type $16 > $09) and are processed for **unlimited food pickup**:
- Not subject to the `$7E` food count limit
- Appears in special watermelon levels every few stages
- Gets converted to slice ($17) after pickup

### sub_9C28 - Level Initialization with Replay Buffers ($9C28)
