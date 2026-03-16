# Midnight Hours — Complete Architecture Audit

> **Audited:** March 16, 2026 | **Source:** MidnightHoursMultiplayer.rbxl (1.1 MB)
> **Fork:** https://github.com/marguerite347/Midnight-Hours

---

## Game Overview

Midnight Hours is a **first-person multiplayer horror survival game** where players wake up in a house at night and must survive from **1:00 AM to 5:00 AM** while a monster hunts them. The game features:

- First-person locked camera with view bobbing
- A house environment with interactive doors, lights, drawers, phones, and hiding spots
- A monster that arrives at different times based on game events
- Multiple **endings** (8 total) triggered by different player choices
- A phone mechanic with a randomized 6-digit code (the "Billie" ending)
- Tools: Flashlight, Key, Lighter, Board, Gasoline, Gun, Axe, Meat, Poison, Taser, Radio
- Monetization: Revive purchases, donations, VIP gamepass, Radio gamepass, "Be Monster" gamepass
- Replay/teleport system to restart or return to lobby
- Role system (Billie, Santa, Police, Gingerbread, Monster character swaps)

---

## Time System (The Core Loop)

**Script:** `ReplicatedStorage.Maps.NightOne.ActionParts.Digital Clock.ClockManager`

The clock is the heartbeat of the entire game:

- **Start Time:** 1:00 AM
- **End Time:** 5:00 AM
- **Tick Rate:** 1 in-game minute = 2 real seconds
- **Total game duration:** ~8 minutes real-time (4 hours × 60 minutes × 2 seconds ÷ 60)
- Time stored in `ReplicatedStorage.Results.Time` as a decimal (e.g., 2.5 = 2:30 AM)
- `KeepTicking` BoolValue controls whether clock advances
- Clock display on an in-game digital clock (SurfaceGui)
- Alarm sound plays at 5:00 AM, `Results.Survived` set to true

---

## Script Inventory

### Server Scripts (ServerScriptService)

| Script | Lines | Purpose |
|--------|-------|---------|
| **MAIN** | 99 | Shard collection minigame loop — spawns "Baddie" monster + collectible shards. Separate from the main NightOne gameplay. Runs in a while-true loop. |
| **MainGameplay** | 722 | **THE CORE SCRIPT.** Manages NightOne: monster AI (SimplePath), time-based events (phone rings, lights out, glass shatter, monster entry), Billie/Police phone endings, player spawning, data stores, monster touch-kill logic, basement trigger. |
| **ProximityPrompts** | 247 | **Master interaction handler.** All ProximityPrompt logic: doors (open/close/lock), drawers, light switches, powerbox, phone answering, hiding spots, tool pickup, sink flooding, computer interaction, board placement. |
| **ProductPurchases** | 216 | Monetization: Revive products (time-based), donation products, gamepass purchase handling. Uses DataStore for purchase history. |
| **RolesServer** | 68 | Character swap system: Billie, Santa, Police, Gingerbread, Monster roles. Replaces player character model and gives role-specific tools. |
| **BeMonsterGamepass** | 32 | "Be Monster" gamepass handler — spawns player as monster model if they own the gamepass. |
| **ReplayHandler** | 56 | Replay lobby system — players vote ready, then TeleportService sends them to a new private server. |
| **MenuTeleport** | 9 | Simple teleport back to lobby place (ID: 10005865562). |
| **PlrCollissions** | 33 | Disables player-to-player collisions using PhysicsService collision groups. Also awards gamepass badges. |
| **RemoveData** | 49 | Admin tool for removing player data from DataStores. |
| **ChatTag** | 27 | Chat tags for Dev, Tester, Mod, VIP, Member based on group rank or gamepass. |
| **ChristmasBadge** | 5 | Awards a badge when HideGift remote fires (Christmas event). |

### Client Scripts (StarterGui / StarterPlayer / StarterPack / ReplicatedFirst)

| Script | Location | Purpose |
|--------|----------|---------|
| **Main** | `StarterGui.Main.Main` | **Core client script.** Camera management, monster proximity VFX (FOV change, camera shake, screen distortion, string music intensity), death sequence (jumpscare camera lock on monster face), spectate mode after death, title/subtitle typewriter text, cutscene handling. |
| **RemoteHandler** | `StarterGui.RemoteHandler` | Handles ClientRemote events: computer camera, hide/unhide camera, revive, explosion shake, door collision enabling. Also manages plank visibility. |
| **ViewBobbing** | `StarterGui.ViewBobbing` | Head-bob effect while walking. Uses sinusoidal camera offset on RenderStepped. Controlled by PlrFolder.ViewBob value. |
| **CustomPrompts** | `StarterGui.CustomPrompts` | Custom ProximityPrompt UI — replaces default Roblox prompts with styled circular button prompts with hold-to-activate progress bars. |
| **Startup** | `StarterGui.Startup` | Sets ambient lighting after 5-second delay. |
| **MenuHandler** | `StarterGui.MenuScreen.BlackoutFrame.MenuHandler` | Blackout overlay management, DeathFunction binding (camera to menu view). |
| **DeathHandler** | `StarterGui.MenuScreen.ReviveFrame.DeathHandler` | Death UI — shows revive/menu/replay buttons after ClientDied event fires. |
| **VictoryHandler** | `StarterGui.MenuScreen.VictoryFrame.VictoryHandler` | Victory UI — shows ending type name, menu/replay buttons after ClientWon event fires. |
| **ReplayHandler** | `StarterGui.MenuScreen.ReplayFrame.ReplayHandler` | Replay lobby UI — ready-up system with player count. |
| **BadgeScript** | `StarterGui.MenuScreen.Endings.BadgeScript` | Displays earned ending badges. |
| **MonsterScript** | `StarterGui.MenuScreen.BecomeMonster.MonsterScript` | UI for "Be Monster" gamepass. |
| **RolesHandler** | `StarterGui.Roles.Roles.RolesHandler` | Role selection UI. |
| **FlashlightHandler** | `StarterPack.Flashlight.FlashlightHandler` | Flashlight tool — attaches SurfaceLight to camera, follows camera CFrame. |
| **Controller** | `ReplicatedFirst.Controller` | Disables chat on gamepad. |
| **RbxCharacterSounds** | `StarterPlayer.StarterPlayerScripts.RbxCharacterSounds` | Default Roblox character sounds. |

### Monster AI Scripts (ReplicatedStorage)

| Script | Location | Purpose |
|--------|----------|---------|
| **AI** | `ReplicatedStorage.Baddie.AI` | Original monster AI for the "Baddie" (shard minigame). Uses PathfindingService + Heartbeat loop. Finds nearest player, raycasts for line-of-sight, tweens movement. Kill animation on proximity. Speed scales with player count. |
| **NewAI** | `ReplicatedStorage.Baddie.NewAI` | Newer AI using a `Bot` module. Simple: spawn, step on Heartbeat. |

### Tool Scripts (ServerStorage.Tools)

| Tool | Scripts | Mechanics |
|------|---------|-----------|
| **Flashlight** | FlashlightHandler (Local) | Camera-attached SurfaceLight |
| **Board** | BoardHandler (Server) | Barricade placement on windows/doors |
| **Gasoline** | GasHandler (Server) | Pour gasoline + ignite with lighter = explosion/fire |
| **Gun** | Client + Server + PartCache | Shotgun with ammo, reload, raycasting |
| **Radio** | Client + RadioServer | Music player tool (gamepass-gated) |
| **Key** | KeyHandler (Server) | Opens SafeDoor |
| **Axe** | Handle.Script (Server) | Melee weapon |
| **Meat** | Script (Server) | Place in food bowl to distract monster |
| **Poison** | Script (Server) | Combine with meat to make PoisonedMeat |
| **PoisonedMeat** | Script (Server) | Kills monster → "Poisoned" ending |
| **Taser** | LocalScript + Script (Server) | Stun weapon (Police role) |
| **Gift/SantaGift** | Scripts (Server) | Christmas event items |

---

## Remote Events & Functions

### RemoteEvents (ReplicatedStorage.MainRemotes)

| Remote | Direction | Purpose |
|--------|-----------|---------|
| **ClientRemote** | Server→Client | Multi-purpose: Computer camera, Hide/Unhide, Revived, Explosion shake, DoorsCOLLIDE |
| **ClientDied** | Server→Client | Triggers death UI |
| **ClientWon** | Server→Client | Triggers victory UI with ending type string |
| **CutsceneEvent** | Server→Client | Triggers cutscene sequences (NightOneBeginning, BasementKey, etc.) |
| **PurchaseRemote** | Client→Server | Purchase requests (products, gamepasses, revives) |
| **ReturnToLobby** | Client→Server | Teleport player back to lobby |
| **PlankRemote** | Server→Client | Show/hide board placement prompts |
| **BeMonster** | Client→Server | Toggle monster mode (gamepass) |
| **FixCamera** | Server→Client | Fix camera after character swap |
| **ChosenRole** | Client→Server | Role selection (Billie, Santa, Police, etc.) |
| **HideGift** | Client→Server | Hide gift placement (Christmas event) |
| **BadgeRemote** | — | Badge-related communication |
| **PlayRemote** | — | Play-related communication |
| **ReplayLeaving** | Server→Client | Replay teleport countdown |

### RemoteFunction

| Remote | Purpose |
|--------|---------|
| **ReplayMode** | Replay lobby ready-up system (InvokeServer) |

### BindableFunction

| Remote | Purpose |
|--------|---------|
| **DeathFunction** | Internal death handling |
| **ReviveFunction** | Reset clock to specified time on revive |

---

## Game Values (ReplicatedStorage.MainValues)

| Value | Type | Purpose |
|-------|------|---------|
| **CurrentMap** | ObjectValue | Reference to current map's Workspace folder |
| **KeepTicking** | BoolValue | Controls whether the clock advances |
| **LastPosition** | CFrameValue | Player's position before hiding (for unhide restore) |
| **SinkOnAmount** | NumberValue | Tracks how many sinks are on (7 = flood ending) |

## Results (ReplicatedStorage.Results)

| Value | Type | Purpose |
|-------|------|---------|
| **Time** | NumberValue | Current game time as decimal (1.0 = 1:00 AM, 2.5 = 2:30 AM) |
| **Survived** | BoolValue | Set true when clock reaches 5:00 AM |
| **PlrOwned** | Folder | Tracks tools player has picked up (for revive restore) |

---

## Night One Timeline (Time-Based Events)

| Time | Event |
|------|-------|
| **1:00 AM** | Game starts. Players spawn in house. Cutscene: "You wake up in the dead of night. Survive until 5:00." Key, Lighter, and phone code note spawn at random locations. |
| **1:30 AM** | Phone rings (first call). Players can answer to hear story audio. |
| **1:45 AM** | Female scream in the distance. |
| **2:00 AM** | Monster spawns outside, pathfinds to front door. Lights flicker briefly. |
| **2:30 AM** | Monster at the door (attempts entry). |
| **2:45 AM** | Phone rings again (second call). |
| **3:00 AM** | **Power outage.** All lights off. Powerbox prompt changes to "Reset". Fake monsters appear briefly. |
| **3:30 AM** | Lights flicker rapidly 20 times (tension building). Fake monsters removed. |
| **3:45 AM** | Phone rings again (third call). |
| **4:00 AM** | Monster howl. Glass shatters (monster breaks into house). |
| **4:30 AM** | Monster spawns at final indoor position. Chases nearest player. |
| **5:00 AM** | **Survival!** "Morning" ending. Badge awarded. |

---

## Endings (8 Total)

| Ending | Badge ID | Trigger |
|--------|----------|---------|
| **Night One: Morning** | 2127663682 | Survive until 5:00 AM |
| **Night One: Poisoned** | 2130465400 | Feed monster poisoned meat |
| **Night One: SWAT** | 2141489371 | Call 911 → Police arrive → Monster kills police |
| **Night One: Billie** | 2141489456 | Dial the 6-digit code from the note → Billie arrives and kills you |
| **Flood Ending** | 2130460097 | Turn on all 7 sinks → house floods |
| **Died Badge** | 2127132055 | Die to monster |
| **VIP Badge** | 2128359650 | Own VIP gamepass |
| **Donor Badge** | 2127132051 | Donate any amount |
| **Christmas Gift** | 2130029569 | Find hidden gift |
| **Be Monster Badge** | 2129507754 | Own Be Monster gamepass |

---

## Monster AI Architecture

### Main Monster (SimplePath-based)
- Uses `SimplePath` module from ServerStorage.ServerModules
- Pathfinds between KeyPoints (named Parts in workspace)
- `CurrentPart` variable determines chase target
- At front door: knocks → waits → checks if door open/unlocked → breaks door if locked → enters
- If meat in food bowl: diverts to eat → if poisoned → dies (Poisoned ending)
- Touch-kills players on contact
- Can break doors on touch
- Speed: 20 (normal), scales with player count for Baddie AI

### Baddie Monster (Heartbeat-based)
- Runs every frame on Heartbeat
- Finds nearest player torso within range
- Raycasts for line-of-sight
- Tweens CFrame for smooth movement
- Kill animation plays on proximity (≤5 studs)
- Speed: 30 base + 5 per additional player

---

## Audio System

All sounds are in `Workspace.Sounds` (global, non-spatial). **37 sound objects** including:

**Horror/Ambient:** MonsterHowl, FemaleScream, PowerOutage, GlassShatter, DoorBreak, DoorShake (1-4)
**Interaction:** DoorOpen, DoorClose, Lock, LockedDoor, LightSwitch, Drawer, Vault, Switch, Flush
**Phone:** PhoneRing, Phone1/2/3 (story calls), PhoneDial, PhoneButton
**Monster:** MonsterFood, MonsterHurt
**Game:** ClockBeep, Alarm, Collect, Explosion, Fire, PoliceSiren, Yell
**Music:** LobbyMusic

Several sounds have **PitchShiftSoundEffect** and **DistortionSoundEffect** children for horror processing (Phone, DoorKnock, DoorShakes, MonsterHurt).

**CRITICAL NOTE:** All audio is currently **global** (not spatial). This is the single biggest opportunity for horror improvement — converting to 3D spatial audio using RollOffMode.InverseTapered.

---

## Map Structure

```
Workspace.Maps.NightOne/
  ├── SpawnPart (player spawn)
  ├── MonsterFinalSpawn (4:30 AM monster position)
  ├── BasementTrigger (triggers monster in basement)
  ├── House/ (main building geometry)
  │   └── DoorFrames/
  ├── PhoneNumbers/ (phone keypad with ClickDetectors)
  │   └── ActionScreen (display)
  └── ActionParts/ (cloned from ReplicatedStorage.Maps.NightOne)
      ├── Doors (with DoorOpen/DoorClosed CFrame targets)
      ├── Glass (breakable windows)
      ├── Boards (barricade positions)
      ├── FrontDoor
      ├── SafeDoor
      ├── Powerbox
      ├── Key
      ├── Lighter
      ├── OutsideTrigger (instant death outside boundary)
      ├── Digital Clock (with ClockManager script)
      └── Various interactive furniture with ProximityPrompts
```

**KeyPoints** (Workspace.KeyPoints): Named waypoint Parts used by SimplePath:
- `OutsideFrontDoor`, `Spawn1`, `MonsterPoliceSpawn`

---

## Dependencies & Modules

| Module | Location | Purpose |
|--------|----------|---------|
| **SimplePath** | ServerStorage.ServerModules | Pathfinding wrapper with Reached/Blocked/Error events |
| **GlassBreak (FractureModule)** | ServerStorage.ServerModules | Shatters glass parts into fragments |
| **DataStore2** | ServerStorage.ServerModules | Community DataStore wrapper for player data persistence |
| **BadgeModule** | ServerStorage.ServerModules | Badge awarding utility |
| **CameraShaker** | StarterGui.RemoteHandler.CameraShaker | Camera shake effects (explosion preset) |

---

## Monetization IDs

| Item | Type | ID |
|------|------|----|
| VIP Gamepass | Gamepass | 81177284 |
| Radio Gamepass | Gamepass | 81177604 |
| Be Monster Gamepass | Gamepass | 105199685 |
| Revive @ 1:30 | Product | 1338719988 |
| Revive @ 2:30 | Product | 1284467481 |
| Revive @ 3:00 | Product | 1284467594 |
| Revive @ 3:30 | Product | 1284468083 |
| Revive @ 4:00 | Product | 1284467883 |
| Lobby Place | Place | 10005865562 |
| Multiplayer Place | Place | 10793702665 |

---

## Known Code Issues (Confirmed from Audit)

1. **`SetPrimaryPartCFrame` used everywhere** — Deprecated. Should use `PivotTo()`.
2. **Global audio only** — No spatial 3D sound. Everything plays from `Workspace.Sounds`.
3. **No cleanup of connections** — Many `Touched`/`Changed` connections leak (never disconnected).
4. **`wait()` used instead of `task.wait()`** — Mixed usage, some old-style `wait()` calls.
5. **Spaghetti architecture** — MainGameplay is 722 lines handling monster AI, time events, phone logic, NPC paths, and player management all in one script.
6. **Hardcoded badge/product IDs** — All IDs inline, no config module.
7. **`FindPartOnRayWithIgnoreList`** — Deprecated ray API used in Baddie AI. Should use `workspace:Raycast()`.
8. **PhysicsService collision groups** — Deprecated API, should use `BasePart.CollisionGroup`.
9. **No error handling on monster pathfinding** — SimplePath errors just retry indefinitely.
10. **`spawn()` used instead of `task.spawn()`** — In Main client script.
11. **Creator's note:** "code is a sloppy mess" and "largely not refactored" — confirmed accurate.

---

## Transformation Opportunities (What to Keep, What to Rewrite)

### KEEP (Good Foundation)
- First-person camera lock with view bobbing
- Door/drawer/light switch interaction system (ProximityPrompts)
- Hiding mechanic
- Time-based event progression (clock system)
- Map structure with ActionParts cloning pattern
- Glass fracture system
- Basic tool framework

### REWRITE (Critical Improvements)
- **Monster AI** → State machine with patrol/chase/frenzy/lore behaviors (HOLLOWBORNE pattern)
- **Audio** → Full spatial 3D positional audio system (Phasmophobia-style)
- **VFX** → Screen distortion, vignette, atmosphere tweens based on proximity
- **Lighting** → Dynamic horror lighting with ColorCorrection + Atmosphere manipulation
- **Code architecture** → Break MainGameplay into modular scripts per system

### ADD (New Systems)
- Environmental storytelling (phantom echoes, shadow memories, blood seep)
- Dread event system (random spatial horror sounds)
- Player heartbeat system (scales with monster proximity)
- Sonar/radar HUD
- Lore progression engine (5 phases of escalating horror)
- Jump scare system with proper audio ducking
- New monster behaviors (kneeling, scratching, reaching)
- Multiple distinct monsters with different rules (Doors-style)

---

*This document was generated from a complete MCP-based audit of every script in MidnightHoursMultiplayer.rbxl*
