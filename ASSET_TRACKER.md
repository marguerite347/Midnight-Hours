# ASSET TRACKER — Midnight Hours Fork

> **Last Updated:** March 16, 2026 — Initial audit
> **Rule:** Every new mechanic, script, or asset MUST be added here. This is the cheat sheet.

---

## Server Scripts (ServerScriptService)

| Asset/Feature Name | Where It Lives in Studio | What It Requires to Work | What It Does |
|---|---|---|---|
| Main Shard Loop | `ServerScriptService > MAIN` | Needs `ReplicatedStorage.Baddie`, `ReplicatedStorage.Shards`, `Workspace.Sounds.Collect`, `ReplicatedStorage.collected` RemoteEvent, `ReplicatedStorage.ShardParticle`, `ReplicatedStorage.ShardBreakParticle` | Spawns collectible shards and a "Baddie" monster in a loop. Players touch shards to collect them. Separate from NightOne gameplay. |
| Core Night Gameplay | `ServerScriptService > MainGameplay` | Needs `ReplicatedStorage.Monster`, `ServerStorage.ServerModules.SimplePath`, `Workspace.KeyPoints`, `ReplicatedStorage.MainValues`, `ReplicatedStorage.Maps.NightOne`, `Workspace.Sounds.*`, `ServerStorage.ServerModules.GlassBreak`, `ServerStorage.ServerModules.DataStore2`, `ServerStorage.ServerModules.BadgeModule` | THE CORE SCRIPT. Manages NightOne timeline, monster pathfinding, phone mechanic (6-digit Billie code), 911 police ending, time-based horror events, player data, monster touch-kill. |
| Interaction Handler | `ServerScriptService > ProximityPrompts` | Needs all ProximityPrompts in ActionParts, `Workspace.Sounds.*`, `ServerStorage.Tools.*`, `ReplicatedStorage.MainRemotes.*` | Master handler for ALL interactions: doors, drawers, lights, powerbox, phone, hiding, tools, sinks, computer. |
| Monetization | `ServerScriptService > ProductPurchases` | Needs `ReplicatedStorage.MainRemotes.PurchaseRemote`, DataStoreService, `ServerStorage.ServerModules.BadgeModule` | Handles revive purchases (reset time + respawn), donation tracking, gamepass purchases. |
| Role System | `ServerScriptService > RolesServer` | Needs `ReplicatedStorage.MainRemotes.ChosenRole`, character models in script/ServerStorage, `ServerStorage.Tools.Axe/Taser` | Swaps player character to Billie/Santa/Police/Gingerbread/Monster with role-specific tools. |
| Be Monster Gamepass | `ServerScriptService > BeMonsterGamepass` | Needs `ReplicatedStorage.MainRemotes.BeMonster`, `ReplicatedStorage.BeMonster` model, gamepass ID 105199685 | Lets gamepass owners play as the monster. |
| Replay System | `ServerScriptService > ReplayHandler` | Needs `ReplicatedStorage.MainRemotes.ReplayMode`, `ReplicatedStorage.ReplayerLobby`, TeleportService | Replay lobby: players ready up → teleport to fresh private server. |
| Lobby Teleport | `ServerScriptService > MenuTeleport` | Needs `ReplicatedStorage.MainRemotes.ReturnToLobby`, TeleportService | Teleports player to lobby place (ID: 10005865562). |
| Player Collisions | `ServerScriptService > PlrCollissions` | Needs PhysicsService | Disables player-to-player collisions. Awards gamepass badges on join. |
| Data Removal | `ServerScriptService > RemoveData` | Needs `ServerStorage.RemovePlayerData` BindableEvent, DataStoreService | Admin tool: removes player data from all stores. |
| Chat Tags | `ServerScriptService > ChatTag` | Needs ChatServiceRunner (legacy chat) | Assigns Dev/Tester/Mod/VIP/Member chat tags based on user ID, group rank, or gamepass. |
| Christmas Badge | `ServerScriptService > ChristmasBadge` | Needs `ReplicatedStorage.MainRemotes.HideGift` | Awards Christmas gift badge. |

---

## Client Scripts

| Asset/Feature Name | Where It Lives in Studio | What It Requires to Work | What It Does |
|---|---|---|---|
| Core Client (Camera/VFX/Death) | `StarterGui > Main > Main` | Needs `ReplicatedStorage.Monster`, `Workspace.CurrentCamera`, sounds in Main GUI (Drone, Strings, Jump, Die, etc.), `ReplicatedStorage.Results.Time`, `ReplicatedStorage.MainRemotes.CutsceneEvent/ClientWon` | Monster proximity effects (FOV, shake, distortion, string music), death jumpscare sequence (camera locks on monster face), spectate mode, title/subtitle typewriter, cutscene handling. |
| Remote Handler | `StarterGui > RemoteHandler` | Needs `ReplicatedStorage.MainRemotes.ClientRemote/PlankRemote`, `Workspace.Computer`, CameraShaker module | Handles: computer camera view, hide/unhide camera, revived transition, explosion shake, door collision enable, plank visibility. |
| View Bobbing | `StarterGui > ViewBobbing` | Needs `ReplicatedStorage.PlrData.[PlayerName].ViewBob` BoolValue | Sinusoidal head-bob on RenderStepped when walking. Disabled during cutscenes/hiding. |
| Custom Prompts UI | `StarterGui > CustomPrompts` | Needs ProximityPromptService | Replaces default Roblox ProximityPrompt UI with custom circular button prompts with hold-progress bars. |
| Startup Lighting | `StarterGui > Startup` | Needs Lighting service | Sets ambient to dark gray after 5s delay. |
| Menu/Blackout Handler | `StarterGui > MenuScreen > BlackoutFrame > MenuHandler` | Needs `ReplicatedStorage.MainRemotes.DeathFunction`, `Workspace.MenuCamera` | Manages blackout overlay. Binds death camera to menu view. |
| Death UI | `StarterGui > MenuScreen > ReviveFrame > DeathHandler` | Needs `ReplicatedStorage.MainRemotes.ClientDied/PurchaseRemote/ReplayMode/ReturnToLobby`, `Workspace.DoorView` | Shows death screen with Revive/Menu/Replay buttons after dying. |
| Victory UI | `StarterGui > MenuScreen > VictoryFrame > VictoryHandler` | Needs `ReplicatedStorage.MainRemotes.ClientWon/ReturnToLobby/ReplayMode`, `Workspace.DoorView` | Shows victory screen with ending type name and Menu/Replay buttons. |
| Flashlight | `StarterPack > Flashlight > FlashlightHandler` | Needs `ReplicatedStorage.FlashlightPart` (with SurfaceLight) | Camera-attached flashlight. Light follows camera CFrame when tool equipped. |
| Gamepad Controller | `ReplicatedFirst > Controller` | Needs UserInputService | Disables chat UI on gamepad input. |

---

## Monster Models

| Asset/Feature Name | Where It Lives in Studio | What It Requires to Work | What It Does |
|---|---|---|---|
| Main Monster | `ReplicatedStorage > Monster` | Needs Humanoid, PrimaryPart (SM_Monster_UpperTorso_JFG), Meat part, Animator | Primary antagonist. Pathfinds via SimplePath. Touch-kills players. Can eat meat (distraction). Has kill animation + eating animation. |
| Baddie | `ReplicatedStorage > Baddie` | Needs AI script, NewAI script, Humanoid, HumanoidRootPart | Shard minigame monster. Heartbeat-driven AI with pathfinding, LOS raycasting, tween movement. |
| BeMonster | `ReplicatedStorage > BeMonster` | Needs Animate script, KillScript on HumanoidRootPart | Player-controlled monster model (gamepass). |
| FakeMonsters | `ReplicatedStorage > FakeMonsters` | Parented to Workspace at 3:00 AM | Fake monster silhouettes that appear during power outage for jump-scare effect. Removed at 3:30 AM. |

---

## Tools (ServerStorage.Tools)

| Tool Name | Scripts | What It Does |
|---|---|---|
| Flashlight | FlashlightHandler (Local) | Camera-attached light source. Default starting tool. |
| Key | KeyHandler (Server) | Unlocks SafeDoor. Spawns at random location. |
| Lighter | (no script — used via GasHandler) | Used to ignite gasoline. Spawns at random location. |
| Board | BoardHandler (Server) | Barricades doors/windows. Shows placement prompts when equipped. |
| Gasoline | GasHandler (Server) | Pour on basement door → ignite with lighter → explosion (BasementDoorExplosion cutscene). |
| Gun | Client + Server + PartCache module | Shotgun. Shoot to damage/push monster. Limited ammo with reload. |
| Axe | Handle.Script (Server) | Melee weapon. Given to Billie role. Can break basement door. |
| Meat | Script (Server) | Place in food bowl to distract monster. |
| Poison | Script (Server) | Combine with meat → PoisonedMeat. |
| PoisonedMeat | Script (Server) | Placed meat + poison = monster eats and dies (Poisoned ending). |
| Taser | LocalScript + Script (Server) | Stun weapon. Given to Police role. |
| Radio | Client + RadioServer | Music player. Requires Radio gamepass (81177604). |
| Gift | Script (Server) | Christmas event item. |
| SantaGift | Handle.Script + LocalScript (Server) | Santa role gift-giving tool. |

---

## Audio Assets (Workspace.Sounds)

| Sound Name | Purpose | Effects Applied |
|---|---|---|
| Collect | Shard collection | Flange + Chorus |
| DoorBreak | Door being broken down | — |
| DoorOpen / DoorClose | Door interaction | — |
| DoorKnock | Monster knocking | Distortion + PitchShift |
| DoorShake 1-4 | Monster shaking door | Distortion + PitchShift |
| Lock / LockedDoor | Lock interaction | — |
| LightSwitch | Light toggle | — |
| Drawer | Drawer interaction | — |
| Vault | Safe door opening | — |
| Switch | Powerbox toggle | — |
| Flush | Toilet flush | — |
| PhoneRing | Phone ringing | — |
| Phone1 / Phone2 / Phone3 | Story phone calls (rbxassetid) | PitchShift + Distortion |
| PhoneDial | Dialing sound | — |
| PhoneButton | Keypad press | — |
| FemaleScream | 1:45 AM event | — |
| MonsterHowl | 4:00 AM event | — |
| MonsterHurt | Monster poisoned | PitchShift + Reverb |
| MonsterFood | Monster eating | — |
| PowerOutage | 3:00 AM blackout | — |
| GlassShatter | 4:00 AM window break | — |
| Explosion | Gasoline explosion | — |
| Fire | Fire sound | — |
| ClockBeep | Hourly clock chime | — |
| Alarm | 5:00 AM alarm | — |
| PoliceSiren | 911 ending | — |
| Yell | NPC death scream | — |
| LobbyMusic | Background music | — |
| Error | Error feedback | — |

---

## Modules (ServerStorage.ServerModules)

| Module Name | What It Does |
|---|---|
| SimplePath | Pathfinding wrapper. Creates paths, fires Reached/Blocked/WaypointReached/Error events. Has `GetNearestCharacter()` utility. |
| GlassBreak | Fractures a Part into realistic glass shards. Used for window breaking at 4:00 AM. |
| DataStore2 | Community DataStore wrapper with caching and ordered stores. Used for donation tracking. |
| BadgeModule | Utility for awarding badges with error handling. |

---

## Key Workspace Objects

| Object | Location | Purpose |
|---|---|---|
| MenuCamera | `Workspace.MenuCamera` | Camera position for death/menu screen |
| DoorView | `Workspace.DoorView` | Camera position for death/victory overlays |
| BeMonsterSpawn | `Workspace.BeMonsterSpawn` | Spawn point for "Be Monster" players |
| PlrMonsters | `Workspace.PlrMonsters` | Container for player-controlled monster characters |
| Computer | `Workspace.Computer` | In-game computer with donation/VIP/Radio purchase UI |
| Water | `Workspace.Water` | Flood water plane (rises when 7 sinks activated) |
| MeatHolder | `Workspace.MeatHolder` | Food bowl for monster meat distraction |
| DigitNum | `Workspace.DigitNum` | SurfaceGui displaying the 6-digit Billie phone code |
| BabyMonster | `Workspace.BabyMonster` | Small monster with Animate script |
| KeyPoints | `Workspace.KeyPoints` | Pathfinding waypoints: OutsideFrontDoor, Spawn1, MonsterPoliceSpawn |

---

## CUSTOM ADDITIONS LOG

> **All new features added by Claude go below this line. Update after EVERY change.**

| Date | Asset/Feature Name | Where It Lives in Studio | What It Requires to Work | What It Does |
|---|---|---|---|---|
| — | *(No custom additions yet)* | — | — | — |

---

*Follow Rule 2: Update this ledger after EVERY code change. If the flashlight breaks, Marguerite checks here first.*
