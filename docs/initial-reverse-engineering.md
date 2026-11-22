# Initial reverse-engineering notes

## Goal

Make the Dawn of Ragnarök Raven form usable on the main-game maps in Assassin's Creed Valhalla 1.7.0. The game should start on a normal save without first visiting the DLC map, and the player should not need Cheat Engine running.

## What the supplied table does

`RavenMod.CT` contains three Auto Assembler hooks. The notes below summarize their behavior; the table itself is not copied into the repository.

| Executable RVA | Observed behavior |
| --- | --- |
| `0x029578A0` | Returns immediately from a routine associated with purging Raven ability data. |
| `0x01F9E080` | Returns immediately from the runtime processing routine for the Ragnarök ability dataset. |
| `0x0294BE40` | Makes the Raven morph availability check return true. |

The table's signatures and injection comments identify these locations for its tested build. They still need to be checked against the user's executable before any patch is produced.

## What this tells us

The proof of concept changes live game code; it does not load the Raven ability's assets or initialize its dataset. Its instructions require visiting Dawn of Ragnarök first, and its description warns that forcing the ability without the data present can crash the game.

Baking the three code changes into the executable could remove the need for a live Cheat Engine process. By itself, that would not meet the no-preload requirement. We still need to find how the game creates or loads the Raven ability dataset on a clean main-game launch, then make that initialization available in the main game.

## Next investigation

1. Validate the table's byte signatures and RVAs against the exact 1.7.0 `ACValhalla.exe`.
2. Identify the Raven ability dataset and its dependencies in the game's Forge data.
3. Trace how those resources are registered when entering Dawn of Ragnarök and determine whether a data patch can register them at startup, or whether a small version-specific code hook is needed.
4. Only then choose the install format and implement the no-preload solution.

## Asset-tool compatibility

The user's open AnvilToolkit window is version 1.2.10. Its game selector does not list Assassin's Creed Valhalla, so this installed build cannot browse Valhalla's Forge archives. The current AnvilToolkit supported-game list also omits Valhalla.

[Blacksmith](https://github.com/theawesomecoder61/Blacksmith) documents Valhalla Forge browsing and extraction, including filelist generation. It explicitly cannot repack or replace files, so it can help identify Raven data but cannot by itself package a Forge-based mod. The official latest release page identifies v1.9.5.1 as its release for Valhalla 1.3.1; compatibility with the user's 1.7.0 build remains to be verified.

The game executable and Forge archives are needed for steps 1–3. Keep those files local; do not commit them to this repository.

## Non-destructive workflow

Initial inspection is read-only. Any first patch attempt should target a separate executable copy in the workspace, not the live game folder. Before a live install is considered, the patcher must verify the exact 1.7.0 signatures, create and verify a restorable backup, and document the restore command. No game files are changed as part of the current investigation.
