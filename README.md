# WWE Raw 2 (Xbox) - Fix Patch

An unofficial patch for **WWE Raw 2** on the original Xbox. It fixes bugs that shipped with the retail game and brings back content that was left unused on the disc. The goal is the game as it should have been: as close to the original as possible, changing only what is needed.

> **This repository contains no game data.** The patcher only changes copies of your own files. You need your own legally obtained copy of the game. Your original files are never modified.

> **Status: work in progress.** The fixes below are documented as they are finished and tested on a real console. The patcher will be released once, when all the planned fixes are done. There will be no partial releases.

## Fixes

| # | Fix | File | Status |
|---|---|---|---|
| 1 | The referee no longer freezes for a few seconds before starting a pin count | `default.xbe` | Tested on a real console |
| 2 | Referee count speeds rebalanced: Mike Chioda (fast) and Nick Patrick (slow) | `Abe.fpk` | Tested on a real console |
| 3 | Royal Rumble: you can no longer roll out under the ropes (walking or running) and hide outside until the end | `default.xbe` | Tested on a real console |
| 4 | Royal Rumble: women only appear as random entrants if a woman is picked in the menu | `default.xbe` | Awaiting in-game test |

The cause of each bug, how it was fixed and every changed byte are in [docs/BUGFIXES.md](docs/BUGFIXES.md).

## Planned or under investigation

- Title Match mode, which is in the game's menu files but disabled (being tested)
- Running stamina: the stamina value exists and affects damage and reversals, but the game never uses it up (paused until the bug fixes are done)
- Pin spam, Season mode interferences (allies and enemies swapped), AI taking control of the player after an interference, ladder and cage climbing, target switching, belt loss, AI idle waits, collisions

## What you need

- Your own *WWE Raw 2* disc, extracted to a folder (the one that contains `default.xbe`)
- Windows 7 or later (the patcher will be a PowerShell script started from a `.bat` file)
- A softmodded or modded console, or an emulator, to play the patched files

## Supported version

| File | Size (bytes) | SHA-1 | SHA-256 |
|---|---|---|---|
| `default.xbe` | 2 596 864 | `19dfb6d26077829854473d33aa7a19bfb8f4b064` | `87660468e4694479e9b51b05ad7f7fd4bc445b28ebcb0c794ebe751a65d8e7c2` |
| `Abe.fpk` | 409 836 | `c9a3aee707d68d3f67711d8fd7e45aa6249191a0` | `9b8b373c1c1669e0aea87d9e0ca0ed9806cc25479876a7e23d676befed428bc6` |

Retail release, title ID `5451000D`, version 1, all regions. Before writing anything, the patcher checks that every byte it changes still has its original value, so an already patched file or a different version is refused rather than broken.

## Credits

Patch, testing on real hardware and research direction by **XadPT**. Reverse engineering done with the help of AI (Claude).

## License

The patcher scripts and documentation are released under the MIT License (see `LICENSE`). *WWE Raw 2* and its contents belong to their respective owners. This project is not affiliated with them.
