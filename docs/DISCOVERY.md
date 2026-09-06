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
| `$1E47-$1E48` | Score digits (unpacked BCD) |
| `$1F3D-$1FDF` | Food inventory (16 slots) |
| `$1FD7-$1FFF` | Game state (lives, level, cone timer) |
| `$2200-$26FF` | Sprite display buffers |
| `$F000-$F1FF` | Constant data tables |

## Data Tables (at $F000-$F1FF)

### dat_F000 (0xF000-0xF080)
- 128 bytes of text/graphics data
- Contains ASCII-like bytes ending with "MATH" and score-related strings
- `0xF06D-0xF07E`: "MATH" and score values

### dat_F081 (0xF081-0xF095)
- 21 bytes, used by `sub_B3A7`
- Likely level configuration data:
  - Bytes 0-3: Level 1 parameters
  - Bytes 4-7: Level 2 parameters
  - etc.

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

### dat_F0C1 (0xF0C1-0xF0FF)
- 64 bytes of sprite graphic data (character sets)

### dat_F0FA (0xF0FA-0xF19F)
- 168 bytes of sprite palette and pattern data

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

### dat_9E2A-9EFF
- Extended sprite palette data (16 bytes)

## Key Code Sections

### RESET Handler ($EFFA)
```
sub_B000: SEI, INPTCTRL=$07, CTRL=$7F, clear RAM ($00-$3F), JMP sub_B1E8
sub_B1E8: Clear RAM ($00-$3F), set up stack, clear display buffer ($1800-$18FF)
sub_B017: Initialize audio (AUDV0/1=$00), setup NMI vector to sub_B71C
```

### NMI Handler ($B3C8)
```
JMP ($0040) - RAM vector to main game loop
sub_B71C: Audio silence, sprite buffer setup
```

### Main Game Loop Entry ($00E2)
Written by `sub_9B40`:
- Sets `$00E2-$00E3` to point to `sub_9C28`
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
| `sub_B631` | Player facing/movement update |
| `sub_B6CF` | Controller input reading |
| `sub_B71C` | Audio/sprite buffer update |
| `sub_9AA3` | Player movement processing |
| `sub_9AD0` | Sprite position rendering |
| `sub_9C28` | Food management, player initialization |

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

### Chef AI
- **4 chefs** tracked (Angelo, Zorba, Oscar, Jacques)
- Chefs move toward player position
- Chefs can throw food back at player
- Chefs retreat when hit, reappear from manholes

### Manholes
- **3-10 manholes** per level (configurable)
- Deadly if player position overlaps
- Chefs can be forced into manholes (temporary retreat)

### Ice Cream Cone
- **1 cone per level** at random position
- **32-second timer** (starts at `$37` = 55 frames, decrements)
- Cone position changes per level
- Eating cone gives points and advances level

### Combat
- **Food throw** in facing direction (8 directions)
- **Chef food** thrown back when they pick up food
- **Hit detection** via sprite overlap

### Scoring (Inferred)
- Cone eaten: 500-25000 points (increases per level)
- Chef defeated: 100-1000 points
- Chef forced to manhole: 200 points
- Leftovers: 100 points each

### Lives & Progression
- **3-8 lives** (configurable via difficulty)
- **Bonus life** at 25000 points and every 100000 points
- **16 levels** total
- **Watermelon levels** every few stages (unlimited food)

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

## Next Investigation Steps

1. **Cone timer logic**: Find where `$1FDB-$1FDD` is decremented
2. **Manhole collision**: Locate the code that checks player against manhole positions
3. **Level progression**: Find level counter and cone position table
4. **Sound effects**: Trace AUDV0/AUDV1 writes for SFX
5. **Sprite collision**: Find player-chef food collision detection

## Music and Sound Effects

### Audio System
The game uses TIA audio (2 voices) via AUDV0 ($0019) and AUDV1 ($001A).
- `sub_B71C` initializes audio to silence on each frame

### Music Data ($B892-$B8F2)
126 bytes of musical note data for the game's background music:

```
B892-BA71: 4-bar music pattern sequence
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
