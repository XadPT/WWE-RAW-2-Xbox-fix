# WWE Raw 2 (Xbox) - Fix Patch

An unofficial patch for **WWE Raw 2** on the original Xbox. It fixes bugs that shipped with the retail game and brings back content that was left unused on the disc. The goal is the game as it should have been: as close to the original as possible, changing only what is needed.

> **No release yet.** The patch will be released once, when everything below is finished and tested on a real console. There will be no partial or beta releases. This page only shows how the work is going.

> **This repository will never contain game data.** The patcher will only change copies of your own files, so you will need your own copy of the game.

## Done

Tested on a real console.

- **Referee pin count:** the referee no longer stands still for a few seconds before he starts counting a pin.
- **Referee count speeds:** Mike Chioda's fast count and Nick Patrick's slow count are now closer to a regular count, while still feeling different.
- **Royal Rumble:** you can no longer leave the ring under the ropes (walking or running) and hide outside until the end. Going over the top rope still eliminates you as usual.

## In progress

- **Royal Rumble entrants:** women only appear among the random entrants if you pick a woman in the menu (built, waiting for the in-game test).
- **Title Match:** the mode is still in the game's menu files but was disabled. It is being brought back and tested.

## Planned

- **Bug fixes:** pin spam, Season mode interferences (allies and enemies swapped), the AI taking control of the player after an interference, ladder and cage climbing, target switching, belt loss, AI idle waits and collisions.
- **Running stamina:** the game already has a stamina value, but nothing ever uses it up, so you can run forever. Stamina will drain while running and come back over time, with the tired animations that are already in the game.
- **Unused content:** other features, animations and modes left on the disc, where they can be restored without changing the original game.

## Credits

Patch, testing on real hardware and research direction by **XadPT**. Reverse engineering done with the help of AI (Claude).

## License

The patcher scripts and documentation will be released under the MIT License (see `LICENSE`). *WWE Raw 2* and its contents belong to their respective owners. This project is not affiliated with them.
