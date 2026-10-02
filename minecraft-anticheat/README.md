# GrimAC + Vulcan config for an Elytra PvP server

Built for **Paper 1.21.11 + ViaVersion + ViaBackwards**, **GrimAC 2.3.73** (`grimac-bukkit-2.3.73`) and **Vulcan 2.9.7** (2.9.7.26).

| File | Goes to |
|---|---|
| `GrimAC/config.yml` | `plugins/GrimAC/config.yml` |
| `GrimAC/punishments.yml` | `plugins/GrimAC/punishments.yml` |
| `Vulcan/config.yml` | `plugins/Vulcan/config.yml` |

The two `config.yml` files are your current (default) files with a few edits. Every changed line is marked with
`[ElytraPvP]`, `[Vulcan]` or `[Vulcan+Grim]`. `punishments.yml` is rewritten from scratch and explains itself in its comments.

## Install

1. Stop the server and back up your three current files.
2. Copy the three files above into place. Leave Grim's `messages.yml` and `discord.yml` as they are.
3. **You need a ban plugin with a `tempban` command.** Paper has no temporary ban. The configs run
   `tempban <player> 1d <reason>`, which works the same in EssentialsX, LiteBans and AdvancedBan.
   If your ban plugin uses another syntax, only edit the lines that start with `tempban`.
4. Start the server (a full restart, not `/reload`). Check the console for `Error while loading punishments.yml`.
5. Staff permissions: `grim.alerts` + `grim.alerts.enable-on-join` and `vulcan.alerts`.
   Never give players `grim.exempt*`, `grim.nosetback*` or `grim.nomodifypacket*`.

Example ban reason shown to the player:
`Cheating detected by GrimAC (Reach) | Appeal on Discord: https://discord.gg/r5hvgmmVDg`

## Who handles what

| Area | Bans | Alerts only |
|---|---|---|
| Movement on foot: speed, fly, step, jesus, NoFall, ground spoof, phase | **Grim** (prediction engine) | Vulcan |
| Elytra flight | nobody (see "Elytra" below) | Vulcan Elytra checks |
| Timer | **Grim** | Vulcan |
| Reach / Hitboxes | **Grim** (precise, knows 1.21.11 spear/weapon ranges) | Vulcan |
| Knockback / velocity | **Grim** | Vulcan |
| NoSlow | **Grim** | Vulcan |
| Bad / illegal packets, crash exploits | **Grim** | Vulcan |
| Aim | **Vulcan** (22 aim checks) | Grim |
| KillAura | **Vulcan**, plus Grim Hitboxes / Reach / packet order | |
| AutoBlock, FastBow, FastUse, FastPlace, FastBreak, Scaffold, Tower, AirPlace, GhostHand, Inventory | **Vulcan** | Grim (place/break/inventory) |
| AutoClicker, Criticals, Improbable, Baritone | nobody: not reliable enough to ban for | Vulcan (Baritone: both) |

Both plugins alert on everything that is enabled.

## Why you got teleported back / had hits cancelled with no alert

Found in Grim's own source code (`PunishmentManager.java`, `Check.java`):

* Grim teleports you back on the **first** flag, but the default `punishments.yml` alerted much later.
  A rule only fires when the flag count is a multiple of the interval, so `100:40` first alerts at flag **120**.
  Aim's `20:40` first alerts at 40, bad packets at 20 and Hitboxes (`5:3`) at 6.
  Grim also cancels your hits while a teleport is pending.
* Any Grim check not listed in `punishments.yml` still runs and can still teleport you, but it **never alerts**.
  The default file does not list `Phase`, so every Phase teleport was silent.
* Vulcan's ghost-block fix also teleported players.
* Vanilla/Paper has its own checks. If the console says `moved too quickly!` or `moved wrongly!`,
  the server itself teleported the player, not an anticheat. Vanilla also silently ignores attacks from too far away.

Now every Grim group alerts on every flag (`1:1 [alert]`), and every check is listed except the elytra take-off checks,
which are switched off on purpose (see below). Vulcan never teleports anyone.
`/grim history <player>` shows every flag, because every flag is logged.

## Elytra: no teleports, no getting stuck

* `exempt-elytra: true` (Grim, a real 2.3.73 option that is not in the default file): Grim does not simulate
  players while they are gliding. Grim 2.3.73 still has open elytra false positives: gliding onto slabs/stairs
  gets players stuck, plus stepping up while gliding and riptide + elytra (Grim issues
  [#2883](https://github.com/GrimAnticheat/Grim/issues/2883), [#2301](https://github.com/GrimAnticheat/Grim/issues/2301),
  [#1333](https://github.com/GrimAnticheat/Grim/issues/1333)). Walking, landing, Timer, Reach, Hitboxes and packets are still checked.
* Grim's ElytraA-I take-off checks are switched off. In 2.3.73 they cancel the take-off and toggle sneaking,
  which can leave players stuck crouching ([#2859](https://github.com/GrimAnticheat/Grim/issues/2859), only fixed after 2.3.73).
* `allow-sprint-jumping-when-using-elytra` stays `true`. `false` teleports gliding players who sprint-jump.
* Vulcan: every check has `setback: false`, ghost-block setbacks are off (the real blocks are resent instead), and `cancel-hits` is off.
  Vulcan's Elytra checks are alert-only: community reports say they false on dive-then-pull-up.
* Trade-off: no plugin auto-bans elytra **flight** cheats (ElytraFly / elytra speed). Staff see Vulcan's Elytra alerts and decide.
  If you would rather have Grim check gliding players too, set `exempt-elytra: false`. You get better detection, plus the rare elytra setback.
* Keep the vanilla gamerule `/gamerule minecraft:elytra_movement_check true` (the default; in 1.21.11 it replaced
  `disableElytraMovementCheck`). It only catches huge teleport-like jumps. Turn it off only if the console shows
  `moved too quickly!` for legit elytra players.

## Ban thresholds (higher than usual, to avoid false bans)

Grim counts the flags of a whole group in the last 5 minutes:

| Grim group | Checks | Ban at |
|---|---|---|
| Movement | Simulation, GroundSpoof, NoFall | 200 |
| Phase | Phase | 40 |
| NoSlow | NoSlow | 80 |
| Timer | Timer, TimerLimit, TickTimer, NegativeTimer | 100 |
| Knockback | AntiKB, AntiExplosion | 30 |
| Reach | Reach | 15 |
| Hitboxes | Hitboxes | 20 |
| Packets | BadPackets, PacketOrder, Post | 80 |
| Exploits | Crash (not CrashE), Exploit | 10 |
| Combat / Building / Misc | Aim, MultiInteract, Place/Break, Sprint, MultiActions, Vehicle, Chat, Baritone, TransactionOrder, CrashE | alert only |

Vulcan: every check that can ban now has `max-violations` set to about 1.5x Vulcan's default, and always at least +2
(2x for FastBreak and Inventory). For example, Aim A goes from 1 to 3 and KillAura A from 3 to 5. Vulcan still resets all violations every 8 minutes.

## Grim + Vulcan together

Both plugins send "ping" packets to every player and use the same negative ID range for them. When the IDs collide,
Grim can swallow Vulcan's reply and Vulcan kicks the player for an *out of order confirmation packet*.
Settings used for this:

* Grim `disable-pong-cancelling: true`. Vulcan's developer asks Grim users to enable this.
* Vulcan `connection.other-anticheat-support: true` and `connection.ignore-ids-over-0: true`, the matching
  options used in community Grim + Vulcan setups.
* Grim `Reach.enable-post-packet` stays `false`. It confuses Vulcan's packet-order checks.
* Grim's `TransactionOrder` is alert-only, because these ID collisions can trigger it.

Vulcan disguises those kicks as `Internal Exception: io.netty.handler.timeout.ReadTimeoutException`.
To check whether it happens, look for `[Vulcan] ... was kicked for sending an invalid / out of order confirmation packet` in the console.
If it keeps happening, the fix has to come from Grim or Vulcan themselves, so report it to Vulcan support with both plugin versions.

## Known limits

* Nobody auto-bans elytra **flight** cheats. That is the price of no elytra teleports (see above).
* One Grim 2.3.73 behaviour cannot be turned off in the config: if a client starts gliding while Grim thinks it
  is on the ground, Grim resyncs it. Legit clients almost never do this. It is reworked in Grim builds after 2.3.73.
* The Grim/Vulcan ping-ID collision can only be fully fixed inside the plugins (see above).

## Tuning after a few days

* A check falsely flags legit players: raise its ban number (Grim: the `tempban` line in its group; Vulcan: `max-violations`),
  or make it alert-only (Grim: delete the `tempban` line; Vulcan: `punishable: false`).
* Too much chat spam: in a Grim group, change `"1:1 [alert]"` to `"5:5 [alert]"` (that alerts at flags 5, 10, 15...).
* After a fight with spears (new in 1.21.11), check that Vulcan does not alert KillAura/Aim on legit players.
* Grim 2.3.73 is the latest GitHub release, but later Grim dev builds fix more 1.21.11 bugs (spear stabs, trig table, elytra resync).
  Those builds rename a few checks (`BadPacketsT`, `InvalidPlaceA/B`), so re-check `punishments.yml` after updating.

## Sources

* Grim 2.3.73 source code: [GrimAnticheat/Grim @ v2.3.73](https://github.com/GrimAnticheat/Grim/tree/v2.3.73)
  (`PunishmentManager`, `Check`, `OffsetHandler`, `ExemptElytraFeature`, `PacketPingListener`, elytra checks).
* Vulcan default config by its developer: [freppp's Vulcan config gist](https://gist.github.com/freppp/6f669d666835011f3661025c2d9533e2)
* Grim + Vulcan transaction conflict: [Grim #2779](https://github.com/GrimAnticheat/Grim/issues/2779),
  [LightningGrim #127](https://github.com/Axionize/LightningGrim/issues/127),
  [GrimFork notes](https://github.com/BlazeDev07/GrimFork), and Grim [#2482](https://github.com/GrimAnticheat/Grim/issues/2482) (post packet).
* Elytra problems in Grim: [#2883](https://github.com/GrimAnticheat/Grim/issues/2883), [#2301](https://github.com/GrimAnticheat/Grim/issues/2301),
  [#2859](https://github.com/GrimAnticheat/Grim/issues/2859), [#1333](https://github.com/GrimAnticheat/Grim/issues/1333).
* Vulcan's own update notes for 2.9.7 (Grim users should enable `disable-pong-cancelling`).
* Community false-positive reports: Vulcan false flags on elytra dive/pull-up and sometimes on aim,
  Grim false flags on riptide and elytra.
