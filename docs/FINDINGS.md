# Food Fight -- findings so far

Day one. Almost everything below is a measurement or an explicitly-flagged
hypothesis, and the two are kept distinct: where something is read from the
code but never watched running, it says so.

## The cartridge

| | |
|---|---|
| ROM | `Food Fight (NTSC) (Atari) (1987) (E64A5CDE).a78` |
| Size | 32,896 bytes (128-byte header + 32,768) |
| Mapper | linear 32K, no banking, mapped `$8000-$FFFF` |
| Vectors | NMI `$B3C8`, RESET `$EFFA`, IRQ `$EFFD` |
| Controllers | joystick / joystick |
| High scores | high score cart |
| Audio | TIA two voices; no POKEY declared |

Linear, so every address is unambiguous and the cross-bank reference
resolution that caused repeated errors in the banked projects simply does not
apply here. The same 128-byte-header 32K-linear shape as Galaga and Pole
Position II in the series.

## Day one coverage

Tracing from the three hardware vectors reaches **3.2%** of the ROM (1,063 of
32,768 bytes, 513 instructions), leaving 31,705 bytes in 11 gaps.

That is the lowest opening figure of any project in this series -- lower than
Pole Position II's 16.8% -- and one gap accounts for most of it:

     $B732-$EFF9    14,536 bytes   over half the cartridge, contiguous

    $8000-$92F2     4,851 bytes
    $9DEE-$AFFF     4,626 bytes
    $EFFE-$FFFF     4,098 bytes
    $9385-$9A8D     1,801 bytes
     ...and six gaps under 700 bytes each

The NMI vector sits just below the big gap at `$B3C8`, and RESET/IRQ sit at its
very edge, `$EFFA`-`$EFFD`. So the reachable code clusters at the two ends of
`$B732-$EFF9` and the body of it is not reached -- the signature of a main
routine reached through an indirect jump rather than the vectors.

A game of this shape -- a single-screen cooking game, no scrolling -- has its
bulk in graphics and data. That is a hypothesis from the game type, not a
finding: nothing here has been rendered or traced yet.

## No missed code, and two RAM vectors

`disasm.py --check-gaps` classifies every apparent `JSR`/`JMP` whose operand
lands in a gap. Here there are **442 candidates and not one is real** -- every
one is the `$20`/`$4C`/`$6C` byte occurring inside data or mid-instruction. So
no traced code branches into any gap.

That is not the same as "the gaps hold no code", because of these:

    rom:B3CE   JMP ($0040)
    rom:B64E   JMP ($00E2)

Two indirect jumps through RAM. The first is two bytes after the NMI handler;
the second sits just below the top of the 14,536-byte gap, which is exactly
where a main loop would live. Their targets cannot be known statically, and in a
sibling project exactly this construct hid a handler from the tracer for a long
time. Resolving them -- with a live probe, or a `ram_vectors` entry once the
values are known -- is the first job, and it is almost certainly what moves the
coverage figure.

## What's open

Everything. Specifically, and in the order they look worth doing:

* The two RAM vectors above -- resolve `$0040` and `$00E2` live (or seed them
  from a `watch.lua` dump) and re-trace; expect the 14,536-byte gap to collapse.
* The graphics: render `$8000-$92F2` and `$9DEE-$AFFF`, and find what points at
  them.
* The remaining gaps, none larger than 1,801 bytes.
* The audio tooling: TIA two voices only, so `audiotrace.py` and a capture will
  still contribute -- the game has jingles even if the effect set is small.
