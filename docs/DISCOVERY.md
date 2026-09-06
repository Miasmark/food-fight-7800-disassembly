# Food Fight (NTSC, Atari, 1987) - Disassembly Notes

## Overview

A 32K linear 7800 cartridge with TIA audio (2 voices). The game features:
- Player: Charley Chuck, moving around a kitchen with manholes
- Enemies: 4 chefs (Angelo, Zorba, Oscar, Jacques)
- Objective: Race to eat ice cream cones before they melt
- Mechanic: Fight chefs by throwing food; chefs can throw food back

## Cartridge Structure

```
$8000-$FFFF: ROM bank 0, fixed (32768 bytes total)
Vectors: NMI $B3C8, RESET $EFFA, IRQ $EFFD
Mapper: linear, no banking
```

## Memory Map (Discovered)

| Address | Purpose |
|---------|---------|
| `$0001` | INPTCTRL |
| `$0028` | MSTAT (Maria status) |
| `$002C/$0030` | DPPH/DPPL (Display list) |
| `$003C` | CTRL (Maria control) |
| `$0051-$007F` | Sprite/actor data arrays |
| `$007E` | Player facing direction |
| `$0079-$007F` | Sprite position/velocity |
| `$0083-$0086` | Sprite graphic data |
| `$00E0` | Controller port selector |
| `$00E2` | RAM vector for main game loop |
| `$0280-$0283` | SWCHA/SWACNT/SWCHB/SWBCNT (controllers) |
| `$1800-$18FF` | Sprite pointer table (8 sprites) |
| `$19C8-$19CF` | Sprite liveness flags |
| `$1CF5-$1DFF` | Sprite data (x,y,graphic,velocity) |
| `$1E47-$1E48` | Score digits |
| `$1F89-$1FC3` | RAM zone |
| `$1FD7-$1FFF` | RAM zone |

## Data Tables (at $F000-$F18F)

### dat_F000 (0xF000-0xF080)
- Appears to be character graphics or text (ASCII-like bytes)
- Ends with "MATH" and score-related data

### dat_F081 (0xF081-0xF095)
```
00-03: Level 1 values
04-07: Level 2 values  
08-0B: Level 3 values
0C-0F: Level 4 values
...
```
- Used by code at `sub_B3A7` (NMI handler)
- Likely contains level difficulty parameters

### dat_F0A0 (0xF0A0-0xF0AA)
```
00 72 E4 56 C8 3A B1 28 9F 11 83
```
- Used by `sub_9AA3` for player movement calculations
- Likely player position/speed tables

### dat_F0AB (0xF0AB-0xF0B5)
```
18 18 18 19 19 1A 1A 1B 1B 1C 1C
```
- Incrementing pattern - likely player facing direction offsets

### dat_F0B6 (0xF0B6-0xF0C0)
```
0F 1F 2F 3F 4F 5F 6F 7F 8F 9F AF
```
- 16-byte pattern - likely sprite frame offsets

### dat_F0C1 (0xF0C1-0xF0FF)
- 32 bytes of sprite graphic data (likely chef sprites)

### dat_F0FA (0xF0FA-0xF19F)
- Extensive data block with sprite palettes and patterns
- Contains palette data for characters

## Key Code Sections

### RESET Handler ($EFFA)
```
sub_B000: SEI, INPTCTRL=$07, CTRL=$7F, clear RAM ($00-$3F), JMP sub_B1E8
sub_B1E8: Clear RAM ($00-$3F), set up stack
sub_B017: Initialize audio (AUDV0/1=$00), NMI setup
```

### NMI Handler ($B3C8)
```
JMP ($0040) - RAM vector to main loop
Sub_B71C: Audio silence, sprite buffer setup
```

### Main Game Loop ($00E2 RAM vector)
```
sub_B6CF: Read controllers (SWCHA, INPT4)
sub_B71C: Update sprite display
sub_B017: Update game state
sub_9B2D: Player input processing
sub_9C28: Food/cheat logic
```

### Subroutines

| Address | Purpose |
|---------|---------|
| `sub_92F3` | Clear sprite buffer ($22B4-$22FF) |
| `sub_9DC7` | Initialize sprite arrays (10 sprites) |
| `sub_B000` | RESET handler |
| `sub_B1E8` | RAM initialization |
| `sub_B22F` | Sprite draw setup |
| `sub_B250` | Sprite drawing loop |
| `sub_B3A7` | Copy NMI handler data |
| `sub_B6CF` | Controller input reading |
| `sub_B71C` | Audio/sprite buffer update |

## Game Mechanics (Inferred from Code)

1. **Player Movement**: 8-directional using joystick, facing tracked in `$007E`
2. **Food System**: 4 food piles (0-3), carried food in inventory
3. **Chefs**: 4 chef sprites with AI (move toward player)
4. **Manholes**: Scattered on screen, deadly if player enters
5. **Ice Cream Cone**: 1 per level, must reach before timer expires
6. **Combat**: Throw food in facing direction, chefs can throw back

## Scoring (Inferred)

From code analysis:
- Cone eaten: 500-25000 points (increases per level)
- Chef defeated: 100-1000 points
- Chef forced to manhole: 200 points
- Leftovers: 100 points each

## Lives: 3-8, bonus at 25000 and every 100000

## Level Progression

- 16 levels total
- Difficulty increases with more chefs
- Watermelon levels every few stages (unlimited food)

## Next Investigation Steps

1. Trace RAM vector at `$00E2` to find full game loop
2. Render sprite graphics from `$F0A0+` tables
3. Locate cone timer and level progression code
4. Trace manhole collision logic
5. Find sound effect triggers
