# Bug fixes: technical details

This file explains each fix: what the player sees, what causes it in the game's code or data, and exactly what the patch changes. It is meant for anyone who wants to check the work, port it or build on it.

## Conventions

- **Addresses** are virtual addresses (VA) in the retail `default.xbe`. For this executable, file offset = VA − `0x10000`.
- **Abe.fpk offsets** are file offsets in the retail `Abe.fpk`.
- **Code caves.** The `.text` section has no large free area, so new code is placed in the `0x90` (NOP) padding that the compiler left after many `ret` instructions. A fix that needs more room than one padding run is split into small blocks, each in its own run, joined by `jmp` instructions. The patcher checks that every run it uses still contains only `0x90` before writing. Cave addresses may move in the final release if later fixes need the same space. The hook points do not move.
- **AI scripts.** The wrestler and referee AI is not in the executable. It is a set of flowchart scripts in `Abe.fpk` (`unit_a.xbi` to `unit_e.xbi`): 4096 records of 20 bytes each, laid out as pages of 8×8 cells. Each record calls one command from the AI command table at `0x25E514` (237 commands, for example `WaitRand`, `XT_PFall`, `X_RefType`). Page names below are the Japanese labels found in the scripts.
- **Match rule** is the byte at `0x39D962`: 0 Normal, 1 Hardcore, 2 Street Fight, 3 Ladder, 4 Table, 5 TLC, 6 Battle Royal, 7 Cage, 8 Hell in a Cell, 9 Royal Rumble.

---

## 1. Referee freezes before starting the pin count

**Symptom.** After a pin, the referee sometimes stands still for one to four seconds before he starts counting. It looks random and happens in every match type that can be won by pinfall. The opponent often kicks out before the count begins.

**Cause.** The referee is always AI slot 6; his AI object is at `0x27FED8` (`0x27CAA0 + 6 × 0x8B4`). Two of his scripts contain idle waits of random length:

| Script page | Purpose | Random wait |
|---|---|---|
| ﾚﾌﾘｰ移動総合 (page 290) | Movement and idle routine (shown as `REF_MOVE` in the debug viewer) | `WaitRand(60, 120)` frames |
| INT_OUTSIDE (page 152) | Reaction when a wrestler is outside the ring | `WaitRand(180, 240)` and `WaitRand(120, 150)` frames |

While the referee is inside one of these waits, the script does not react to the pin, so the count starts only when the wait ends. The count itself is not random: every wait in the count routine is fixed (`min == max`).

**Fix.** A hook in the `WaitRand` command (`0x11410`), at the point where it calls the random number generator (`0x11451`). If the AI object is the referee and the wait is random (`min != max`), the wait becomes 0 frames. Fixed waits are left alone, so the count rhythm, the count outside the ring and the disqualification wait do not change. Other wrestlers are not affected.

The referee still waits when he has to climb back into the ring, which is normal animation time, not a script wait.

```asm
; 0x11451  call 0x18D44B (rand)  ->  call 0x113A1
0x113A1  cmp  esi, 0x27FED8       ; is this the referee's AI object?
         jne  0x11403             ; no: original behaviour
         cmp  edi, [ebp+8]        ; min == max (fixed wait)?
         je   0x11403             ; yes: original behaviour
         jmp  0x11408
0x11403  jmp  0x18D44B            ; rand, as before
0x11408  xor  eax, eax            ; random wait -> 0 frames
         xor  edi, edi
         ret
```

| VA | File offset | Original | New |
|---|---|---|---|
| `0x11451` | `0x1451` | `e8f5bf1700` | `e84bffffff` |
| `0x113A1` | `0x13A1` | `90` × 15 | `81fed8fe2700755a3b7d087455eb58` |
| `0x11403` | `0x1403` | `90` × 10 | `e943c0170031c031ffc3` |

---

## 2. Referee count speeds

**Background.** The game has five referee choices. The counting routine (ｶｳﾝﾄ中, page 304) branches on the referee type with `X_RefType`, and each count is `A_Cmd(50,9,5)` followed by a fixed `WaitRand`:

| Type | Referee (menu) | Original wait per count | Patched |
|---|---|---|---|
| 0 | Earl Hebner (regular) | 60 frames | 60 (unchanged) |
| 1 | Tim White (fast moving, normal count) | 60 frames | 60 (unchanged) |
| 2 | Mike Chioda (fast counting) | 20 frames | **40** |
| 3 | Nick Patrick (slow moving, slow counting) | 100 frames | **80** |
| 4 | Random | Picks one of the four before the match | Unchanged |

**Problem.** Chioda's count was three times faster than a regular count, leaving almost no time to kick out. Nick Patrick's was very slow. The new values keep each referee's character with a smaller gap.

**Change.** Each wait is stored as two 32-bit values (min, max) in the script record.

| File | Offset | Original | New |
|---|---|---|---|
| `Abe.fpk` | `0x5F140` | `14000000 14000000` (20, 20) | `28000000 28000000` (40, 40) |
| `Abe.fpk` | `0x5F3C0` | `64000000 64000000` (100, 100) | `50000000 50000000` (80, 80) |

---

## 3. Royal Rumble: rolling out under the ropes

**Symptom.** In the Royal Rumble, pressing Black near the ropes (walking or running) takes the wrestler out of the ring under the bottom rope. Only going over the top rope eliminates, so a player can wait outside until the end and come back to throw out the last wrestler. In the original game, both the walking and the running exits are possible.

**Cause.** The Cage match already refuses these moves, but no check exists for the Royal Rumble. Two action handlers decide whether a wrestler inside the ring may leave:

| Handler | Action | Refused path |
|---|---|---|
| `0xAB4E0` | Run + Black (slide out under the ropes) | `0xAB4EF` (`xor al,al` / return "not allowed") |
| `0xAAFC0` | Walk + Black (step out through the ropes) | `0xAB0A4` (`xor al,al` / return "not allowed") |

**Fix.** At the start of each handler, if the rule byte is 9 (Royal Rumble), the handler jumps to its own "not allowed" path. Nothing else changes: entering the ring (`0xAB670`, `0xAB840`) works as before, eliminations over the top rope work as before, and the Battle Royal is not touched (fighting outside is normal there).

Logic of both hooks:

```asm
; original first instructions, moved into the cave
cmp  byte [0x39D962], 9   ; Royal Rumble?
je   <refused path>       ; yes: leaving is not allowed
jmp  <rest of handler>    ; no: original behaviour
```

| VA | File offset | Original | New | Part |
|---|---|---|---|---|
| `0xAB4E0` | `0x9B4E0` | `8b442404568bf1` | `e95e53feff9090` | Hook (run) |
| `0x90843` | `0x80843` | `90` × 12 | `8b442404568bf1e9d4000000` | Cave |
| `0x90923` | `0x80923` | `90` × 12 | `803d62d9390009e984030000` | Cave |
| `0x90CB3` | `0x80CB3` | `90` × 11 | `0f8436a80100e929a80100` | Cave |
| `0xAAFC0` | `0x9AFC0` | `56578b7c240c` | `e9bd5efeff90` | Hook (walk) |
| `0x90E82` | `0x80E82` | `90` × 11 | `56578b7c240ce915010000` | Cave |
| `0x90FA2` | `0x80FA2` | `90` × 12 | `803d62d9390009e996080000` | Cave |
| `0x91844` | `0x81844` | `90` × 11 | `0f845a980100e977970100` | Cave |

---

## 4. Royal Rumble: random women among the men

**Status: awaiting in-game test.**

**Symptom.** In both Royal Rumble modes, the menu lets you pick your own wrestler and the first three CPU entrants. The rest are random, and the game mixes women in with the men even when only men were picked.

**Cause.** The entrant list is built at `0xA0578` into the table at `0x39D9D8`. The first four entries are the picked wrestlers (`0xFF` = empty). The rest are drawn with `rand & 0x7F` and accepted if the validity check `0x408C0` passes. That check does not look at gender.

**Fix.** After the validity check (`0xA0618`), a random entrant who is a woman is accepted only if at least one of the four picked wrestlers is a woman. Otherwise the game draws again. If you pick a woman yourself, or as one of the first three CPUs, women can still appear at random as before.

The gender comes from the roster: entry = `0x2F0F78 + id × 0x9E4`, byte `+0x94` = 1 for a woman.

```asm
test al, al ; je redraw                    ; original validity result
id = bl ; if roster[id].female == 0 -> ok  ; a man is always accepted
for i = 0..3:                              ; the four picked wrestlers
    p = [0x39D9D8 + i] ; if p == 0xFF -> next
    if roster[p].female == 1 -> ok         ; a woman was picked: accept
redraw: jmp 0xA05F0
ok:     xor eax, eax ; jmp 0xA061E         ; original path
```

| VA | File offset | Original | New | Part |
|---|---|---|---|---|
| `0xA0618` | `0x90618` | `84c074d433c0` | `e90516ffff90` | Hook |
| `0x91C22` | `0x81C22` | `90` × 13 | `84c00f84c6e90000e9c2370000` | Validity result |
| `0x953F1` | `0x853F1` | `90` × 14 | `0fb6c369c0e4090000e9f4f8ffff` | Roster entry of the drawn wrestler |
| `0x94CF3` | `0x84CF3` | `90` × 12 | `80b80c102f0001e9a3120000` | Is the drawn wrestler female? |
| `0x95FA2` | `0x85FA2` | `90` × 11 | `0f850a3d0000e956090000` | No: accept |
| `0x96903` | `0x86903` | `90` × 7 | `33c9e967000000` | Loop start |
| `0x96971` | `0x86971` | `90` × 12 | `0fb681d8d93900e924110000` | Read picked wrestler i |
| `0x97AA1` | `0x87AA1` | `90` × 13 | `3cff0f84a8290000e9b5090000` | Skip empty entry |
| `0x98463` | `0x88463` | `90` × 11 | `69c0e4090000e9d3000000` | Roster entry of picked wrestler |
| `0x98541` | `0x88541` | `90` × 12 | `80b80c102f0001e925170000` | Is the picked wrestler female? |
| `0x99C72` | `0x89C72` | `90` × 11 | `0f843a000000e9d4070000` | Yes: accept |
| `0x9A451` | `0x8A451` | `90` × 15 | `4183f9040f8c16c5ffffe990610000` | Next i, else draw again |
| `0x99CB2` | `0x89CB2` | `90` × 7 | `33c0e965690000` | Accept (original path) |

---

## Verification

All fixes above, applied together to a clean retail `default.xbe`, give:

| File | SHA-1 |
|---|---|
| `default.xbe` with fixes 1, 3 and 4 | `b166779d83b3eea503a7276d933930c0f747887f` |

This hash will change in the final release as more fixes are added.
