# Roblox Engine Architecture Knowledge

## Overview

Roblox is a cloud-based game platform and engine where every game runs across a mandatory server-client boundary enforced at the engine level. Games are written in **Luau** (a typed superset of Lua 5.1 developed by Roblox). Everything in the game world lives in the **DataModel** — a hierarchical tree of `Instance` objects. Roblox handles hosting, matchmaking, CDN, and publishing automatically. There is no traditional "export" step — publishing to Roblox is your deployment.

## Architecture Fundamentals

### The DataModel and Service Tree

The `game` global is the root of the DataModel. All game content lives inside top-level **Services**, each with a specific purpose and replication behavior:

| Service | Accessible By | Purpose |
|---|---|---|
| `Workspace` | Server + Client | Physical game world — Parts, Models, Terrain |
| `ReplicatedStorage` | Server + Client | Shared modules, RemoteEvents, shared assets |
| `ReplicatedFirst` | Server + Client | Runs on client before anything else (loading screens) |
| `ServerScriptService` | Server only | Server scripts — never sent to clients |
| `ServerStorage` | Server only | Server-only assets, never replicated |
| `StarterGui` | Server (templates) | UI templates cloned to each player's `PlayerGui` |
| `StarterPack` | Server (templates) | Tools cloned to each player's `Backpack` |
| `StarterPlayer` | Server (templates) | `StarterPlayerScripts` and `StarterCharacterScripts` cloned per player |
| `Players` | Server + Client | Connected player instances |
| `Lighting` | Server + Client | Visual settings, sky, atmosphere |
| `SoundService` | Server + Client | Global audio settings |
| `Teams` | Server + Client | Team assignments |

**Canonical DataModel structure:**

```
game (DataModel)
├── Workspace                    ← Physical world
│   ├── Terrain
│   ├── Map (Model)
│   └── [dynamic game objects]
├── ReplicatedStorage            ← Shared by server and client
│   ├── Modules/
│   │   ├── SharedModule (ModuleScript)
│   │   └── Signal (ModuleScript)
│   └── Events/
│       ├── DamageEvent (RemoteEvent)
│       └── GetDataFunction (RemoteFunction)
├── ReplicatedFirst              ← Client loading screen lives here
├── ServerScriptService          ← Server-only scripts
│   ├── init.server.lua (Script)
│   └── Services/
│       ├── DataService (ModuleScript)
│       └── CombatService (ModuleScript)
├── ServerStorage                ← Server-only assets
├── StarterGui                   ← UI templates → copied to PlayerGui
│   └── MainHUD (ScreenGui)
├── StarterPlayer
│   ├── StarterPlayerScripts     ← LocalScripts → run per player
│   └── StarterCharacterScripts  ← LocalScripts → run per character spawn
└── Players
    └── [PlayerName]
        ├── PlayerGui            ← Their live UI instances
        ├── Backpack             ← Their tools
        └── Character (in Workspace) ← Their avatar model
```

### Script RunContext — The Server-Client Split

Roblox has three script types. Every script runs in a context determined by its type and `RunContext` property:

| Script Type | RunContext | Where it runs |
|---|---|---|
| `Script` | `Legacy` (default) | Server only — must be in a server container (`ServerScriptService`, `Workspace`) |
| `Script` | `Server` | Server only — can also live in `ReplicatedStorage` (not recommended there) |
| `Script` | `Client` | Client only — can live in `ReplicatedStorage` alongside client ModuleScripts |
| `LocalScript` | n/a | Client only — legacy placement in `StarterGui`, `StarterPlayerScripts`, etc. |
| `ModuleScript` | n/a | No context — runs inside whichever script `require()`s it |

**The modern recommended layout** (from the official scripting docs):
- Server logic: `Script` (`RunContext: Server`) in `ServerScriptService` alongside server-only `ModuleScript`s
- Client logic: `Script` (`RunContext: Client`) in `ReplicatedStorage` alongside client-only `ModuleScript`s
- Shared code: `ModuleScript`s in `ReplicatedStorage`

The `RunContext: Client` in `ReplicatedStorage` approach is preferable to `LocalScript` in `StarterPlayerScripts` because when you click an error in the Output, it links to the stable `ReplicatedStorage` path rather than the ephemeral copied-player path.

**Critical rule:** A ModuleScript in `ReplicatedStorage` can be required by both server and client, but each context gets its own independent copy. State is NOT shared between the server's instance and a client's instance of the same module.

### Client Startup Sequence

When a client first joins, script execution follows a predictable order. Knowing this sequence prevents the most common client-side timing bugs:

1. **`ReplicatedFirst` loads** — Scripts in `ReplicatedFirst` run immediately. These scripts can access `ReplicatedFirst` contents directly (no `WaitForChild` needed) but must use `WaitForChild` for anything in other services.
2. **Client finishes loading** — The rest of the DataModel replicates from the server.
3. **`game.Loaded` fires** — `game:IsLoaded()` returns `true`. This is the signal that the full DataModel is available.
4. **`StarterPlayerScripts` and `ReplicatedStorage` client scripts run** — At this point, scripts can access `ReplicatedStorage` contents directly without `WaitForChild`.
5. **Character spawns** — `Players.LocalPlayer.Character` becomes available. Connect `Players.LocalPlayer.CharacterAdded` to handle it — never assume the character exists at script startup.
6. **`StarterCharacterScripts` run** — These run on every respawn, not just the first spawn.

**Key timing rule:** Property changes replicated from the server may arrive at the client before or after a related `RemoteEvent.OnClientEvent`. Do not assume that a FireClient call and its associated state change arrive atomically. Use `WaitForChild` or verify state explicitly rather than assuming replication order.

### The Replication Model

Roblox uses **FilteringEnabled** (enabled by default and cannot be disabled in production) which means:

- **Server → All clients**: Property changes made on the server replicate automatically
- **Client → Server**: Client changes to Instances do NOT replicate — they only affect that client
- **RemoteEvent**: One-way async communication between contexts (fire and forget)
- **RemoteFunction**: Two-way synchronous communication (request/response — avoid on server-called-from-client; can yield indefinitely if client disconnects)
- **BindableEvent / BindableFunction**: Same-context communication only (server ↔ server, or client ↔ client)

```lua
-- Server-side (Script in ServerScriptService)
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local remoteEvent = ReplicatedStorage.Events.DamageEvent

remoteEvent.OnServerEvent:Connect(function(player: Player, targetId: number, amount: number)
    -- ALWAYS validate on server — never trust client input
    if isValidTarget(player, targetId) and isValidAmount(amount) then
        applyDamage(targetId, amount)
    end
end)

-- Client-side (LocalScript)
local remoteEvent = game:GetService("ReplicatedStorage").Events.DamageEvent
remoteEvent:FireServer(targetId, amount)
```

## Luau Language Guide

Luau is a typed superset of Lua 5.1. Type annotations are optional but strongly recommended — they enable tooling support and catch bugs before runtime.

### Type Inference Modes

Place one of these on the first line of a Script to control how strictly the type checker runs:

```lua
--!nocheck    -- type checking disabled entirely
--!nonstrict  -- (default) only checks explicitly annotated variables
--!strict     -- checks all types based on annotation and inference
```

Set a project-wide default via `Workspace.LuauTypeCheckMode`. Use `--!strict` on all new ModuleScripts — it catches the most errors at edit time. The default `--!nonstrict` mode misses most type mismatches.

### Type System

```lua
-- Basic types
local health: number = 100
local name: string = "Player"
local isAlive: boolean = true

-- Array and dictionary types
local items: {string} = {}
local scores: {[string]: number} = {}

-- Type aliases
type PlayerData = {
    userId: number,
    coins: number,
    level: number,
    inventory: {string},
}

-- Optional fields
type Config = {
    maxHealth: number,
    respawnTime: number?,  -- nil or number
}

-- Function types
type DamageHandler = (target: Model, amount: number) -> boolean
```

### Module Pattern

The standard Roblox module pattern returns a table of functions:

```lua
-- Services/CombatService.lua (ModuleScript in ServerScriptService)
local CombatService = {}

local RunService = game:GetService("RunService")

-- Private state (not in return table)
local _activeCombats: {[number]: boolean} = {}

function CombatService.DealDamage(target: Model, amount: number): boolean
    assert(RunService:IsServer(), "DealDamage must be called on the server")

    local humanoid = target:FindFirstChildOfClass("Humanoid")
    if not humanoid or humanoid.Health <= 0 then return false end

    humanoid:TakeDamage(amount)
    return true
end

function CombatService.IsInCombat(userId: number): boolean
    return _activeCombats[userId] == true
end

return CombatService
```

### Requiring Modules

```lua
-- Requiring a shared module from a server Script (ServerScriptService)
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local SharedUtil = require(ReplicatedStorage:WaitForChild("Modules"):WaitForChild("SharedUtil"))

-- Requiring a shared module from a client Script (RunContext: Client in ReplicatedStorage)
-- WaitForChild is an important safety measure on the client due to replication order
local SharedUtil = require(ReplicatedStorage:WaitForChild("Modules"):WaitForChild("SharedUtil"))

-- Requiring a sibling module (script.Parent is always resolved before the script runs)
local DataService = require(script.Parent.DataService)
```

`WaitForChild` is the official recommended pattern when requiring from `ReplicatedStorage`, especially in client scripts where replication order means a module may not exist yet. On the server, direct paths work but `WaitForChild` is still safe and shown in the official docs. Avoid it only in performance-critical hot paths where the module is already guaranteed to exist.

### Task Library — Modern Async

```lua
-- task library replaces deprecated wait(), spawn(), delay()
-- AVOID: wait(1)   -- imprecise, yields the entire script thread
-- PREFER: task.wait(1)  -- accurate, lightweight yield

-- Fire and forget on next frame
task.spawn(function()
    doSomethingAsync()
end)

-- Delayed execution
task.delay(2, function()
    cleanupObject()
end)

-- Defer to end of current frame
task.defer(function()
    updateUI()
end)

-- Pcall pattern for error-safe async calls
local success, result = pcall(function()
    return game:GetService("DataStoreService")
        :GetDataStore("PlayerData")
        :GetAsync(tostring(player.UserId))
end)

if success then
    return result
else
    warn("DataStore error:", result)
    return getDefaultData()
end
```

### Deferred Events and SignalBehavior

`Workspace.SignalBehavior` controls when event handlers run. New places created from templates default to `Enum.SignalBehavior.Deferred`, which is the recommended setting. The legacy `Immediate` behavior will eventually be retired.

| Behavior | What happens |
|---|---|
| `Deferred` (recommended) | Event handlers are queued and run at the next engine resumption point. The engine runs serial batches between physics, rendering, and input phases. |
| `Immediate` | Event handlers run synchronously, inline inside whatever code triggered the event — can cause re-entrancy and subtle ordering bugs. |

**Why Deferred is better:** With Immediate behavior, triggering 1,000 property changes causes 1,000 event callbacks to interleave before anything else runs. Deferred batches them, giving the engine consistent windows for physics and rendering. It also eliminates a class of re-entrancy bugs where a changed event fires before the change has fully applied.

**The pattern that breaks under Deferred:**

```lua
-- BROKEN under Deferred — success is always false here
local success = false
event:Connect(function()
    success = true
end)
doSomethingToTriggerEvent()
return success  -- callback hasn't run yet

-- CORRECT — yield until the event fires
local success = false
event:Connect(function()
    success = true
end)
doSomethingToTriggerEvent()
task.wait()  -- let the deferred handler run
return success
```

**`Once()` for single-use listeners:**

```lua
-- Prefer Once() over manually disconnecting inside the callback
event:Once(function(value)
    -- handles first invocation only; automatically disconnects
    applyInitialValue(value)
end)
```

Under Deferred mode, calling `:Disconnect()` inside a connected handler drops all still-pending queued invocations of that same handler. Using `:Once()` is clearer intent and avoids the subtlety.

## Data Persistence

`DataStoreService` is Roblox's built-in persistence layer. Roblox's official docs identify five problems every production player data system must solve: **in-memory access** (cache data on the server, only hit DataStore on join/leave/periodic saves), **efficient storage** (one key per player, one `UpdateAsync` call), **replication** (replicate a copy to the client via `RemoteEvent`), **error handling** (pcall + retry with backoff), and **session locking** (prevent two servers writing the same player's data simultaneously).

### Key API choices

Use `UpdateAsync` over `SetAsync` when multiple servers could write the same key. `UpdateAsync` reads the current value before writing, preventing overwrites from racing servers. `SetAsync` is faster but unsafe for concurrent access.

```lua
-- UpdateAsync: safe for concurrent server writes
local success, newData = pcall(function()
    return dataStore:UpdateAsync("Player_" .. player.UserId, function(currentData)
        currentData = currentData or { coins = 0, level = 1 }
        currentData.coins += coinsEarned
        return currentData
    end)
end)
```

### In-Memory Pattern (required for production)

Never call `DataStoreService` every time gameplay state changes — DataStore has rate limits and latency. The correct pattern:

1. Load from DataStore on `Players.PlayerAdded` → store in a server-side table
2. Read/write the in-memory table during gameplay
3. Auto-save periodically (every 60–120 seconds) with `UpdateAsync`
4. Final save on `Players.PlayerRemoving` and `game:BindToClose`

```lua
local DataStoreService = game:GetService("DataStoreService")
local Players = game:GetService("Players")

local playerStore = DataStoreService:GetDataStore("PlayerData")
local DEFAULT_DATA = { coins = 0, level = 1, inventory = {} }

-- In-memory cache: lives for the duration of the server session
local _playerData: {[number]: {}} = {}

local function loadData(player: Player)
    local key = "Player_" .. player.UserId
    local success, data = pcall(function()
        return playerStore:GetAsync(key)
    end)

    if success then
        -- Merge saved data with defaults so new fields appear on existing players
        local loaded = data or {}
        for k, v in DEFAULT_DATA do
            if loaded[k] == nil then loaded[k] = v end
        end
        _playerData[player.UserId] = loaded
    else
        warn("Failed to load data for", player.Name, "- using defaults")
        _playerData[player.UserId] = table.clone(DEFAULT_DATA)
    end
end

local function saveData(player: Player)
    local key = "Player_" .. player.UserId
    local data = _playerData[player.UserId]
    if not data then return end

    local success, err = pcall(function()
        playerStore:UpdateAsync(key, function()
            return data  -- always write the current in-memory state
        end)
    end)

    if not success then
        warn("Failed to save data for", player.Name, ":", err)
    end
end

Players.PlayerAdded:Connect(loadData)

Players.PlayerRemoving:Connect(function(player)
    saveData(player)
    _playerData[player.UserId] = nil
end)

-- Flush all data when server shuts down
game:BindToClose(function()
    for _, player in Players:GetPlayers() do
        saveData(player)
    end
end)
```

### Session Locking with MemoryStore

Session locking prevents two servers loading the same player's data simultaneously (a common cause of item duplication bugs). Use `MemoryStoreService` — Roblox's built-in ephemeral store — as a lock registry:

```lua
local MemoryStoreService = game:GetService("MemoryStoreService")
local sessionLocks = MemoryStoreService:GetHashMap("SessionLocks")

local SERVER_ID = game.JobId
local LOCK_TTL = 30  -- seconds; refresh before expiry

local function acquireLock(userId: number): boolean
    local key = tostring(userId)
    local success, acquired = pcall(function()
        return sessionLocks:UpdateAsync(key, function(current)
            -- Only take lock if no server holds it, or this server already holds it
            if current == nil or current == SERVER_ID then
                return SERVER_ID
            end
            return nil  -- returning nil cancels the write
        end, LOCK_TTL)
    end)
    return success and acquired == SERVER_ID
end

local function releaseLock(userId: number)
    pcall(function()
        sessionLocks:RemoveAsync(tostring(userId))
    end)
end
```

Roblox publishes reference sample code for a complete player data + purchase handling system (covering ordered retries, session locking, and atomic purchase grants) on the Creator Store. See: [Implement player data and purchasing systems](https://create.roblox.com/docs/cloud-services/data-stores/player-data-purchasing).

## Architecture Patterns

### Service Architecture (Recommended)

Organize all server logic into ModuleScript services loaded by a single init Script. This provides clear boundaries and avoids scattered top-level Scripts.

```lua
-- ServerScriptService/init.server.lua (Script)
-- Single entry point that initializes all services in order

local services = {
    require(script.Parent.Services.DataService),
    require(script.Parent.Services.CombatService),
    require(script.Parent.Services.MatchService),
}

-- Init phase: synchronous setup (no yielding)
for _, service in services do
    if service.Init then
        service:Init()
    end
end

-- Start phase: concurrent startup (can yield)
for _, service in services do
    if service.Start then
        task.spawn(service.Start, service)
    end
end
```

### Signal / Event Bus Pattern

`BindableEvent` is the official Roblox mechanism for same-context pub/sub. For a lightweight internal event bus with no Instance overhead, a plain Lua connection table is idiomatic:

```lua
-- Vanilla Lua signal — no external dependencies
type Connection = { Disconnect: (Connection) -> () }
type Signal<T...> = {
    Connect: (Signal<T...>, (T...) -> ()) -> Connection,
    Fire:    (Signal<T...>, T...) -> (),
}

local function createSignal<T...>(): Signal<T...>
    local listeners = {}

    return {
        Connect = function(_, fn)
            listeners[fn] = true
            return {
                Disconnect = function(_)
                    listeners[fn] = nil
                end
            }
        end,
        Fire = function(_, ...)
            for fn in listeners do
                task.spawn(fn, ...)
            end
        end,
    }
end

-- Central events module (ModuleScript in ServerScriptService or ReplicatedStorage)
local GameEvents = {
    PlayerDied   = createSignal(),
    RoundStarted = createSignal(),
    ItemPickedUp = createSignal(),
}

-- Subscribe
local conn = GameEvents.PlayerDied:Connect(function(player, killer)
    updateLeaderboard(player, killer)
end)

-- Fire
GameEvents.PlayerDied:Fire(player, killer)

-- Disconnect when no longer needed
conn:Disconnect()
```

For same-context events inside a single script you can also use a plain `BindableEvent` Instance — the trade-off is that BindableEvents carry Instance lifecycle overhead and require explicit `:Destroy()` cleanup.

### Cross-Server Messaging

`MessagingService` lets servers in the same experience communicate with each other. Common use cases: global announcements ("a rare item dropped!"), live server browser, and cross-server friend notifications.

```lua
-- Server: subscribe all clients to a topic when they join
local MessagingService = game:GetService("MessagingService")
local Players = game:GetService("Players")

local TOPIC = "GlobalAnnouncement"

Players.PlayerAdded:Connect(function(player)
    local ok, connection = pcall(function()
        return MessagingService:SubscribeAsync(TOPIC, function(message)
            -- message.Data is the string you published
            -- message.Sent is the Unix timestamp of when it was published
            print("Broadcast received:", message.Data)
        end)
    end)
    if ok then
        -- Unsubscribe when player leaves to avoid leaked connections
        player.AncestryChanged:Connect(function()
            connection:Disconnect()
        end)
    end
end)

-- Server: publish to all subscribed servers (including this one)
local function broadcastAnnouncement(text: string)
    pcall(function()
        MessagingService:PublishAsync(TOPIC, text)
    end)
end
```

**Limits:** `MessagingService` has rate limits (messages per minute per server, and per topic). Wrap calls in `pcall`. Not suitable for high-frequency per-player data — use `RemoteEvent` for that. For ephemeral cross-server state, combine with `MemoryStoreService`.

### Cleanup Pattern

`RBXScriptConnection` objects keep their referenced callbacks alive until disconnected. Track connections explicitly and disconnect on cleanup — this is idiomatic Roblox without any external library:

```lua
local Enemy = {}
Enemy.__index = Enemy

function Enemy.new(model: Model): Enemy
    local self = setmetatable({}, Enemy)
    self._model = model
    self._connections = {}  -- plain table of RBXScriptConnection

    table.insert(self._connections,
        model.Humanoid.Died:Connect(function()
            self:_onDied()
        end)
    )

    return self
end

function Enemy:_onDied()
    -- handle death
    self:Destroy()
end

function Enemy:Destroy()
    for _, conn in self._connections do
        conn:Disconnect()
    end
    table.clear(self._connections)
    self._model:Destroy()
end

return Enemy
```

## UI Architecture

Roblox UI lives inside `PlayerGui`, populated from `StarterGui` templates. Each ScreenGui is a layer in a stacking compositor.

```
PlayerGui (auto-created per player)
├── MainHUD (ScreenGui, ResetOnSpawn = false)
│   ├── HealthBar (Frame)
│   ├── Crosshair (ImageLabel)
│   └── Hotbar (Frame)
├── MenuGui (ScreenGui, Enabled = false)
└── NotificationGui (ScreenGui, DisplayOrder = 10)
```

**Key ScreenGui properties:**

| Property | Use |
|---|---|
| `ResetOnSpawn = false` | Keep GUI across respawns — use for persistent HUD |
| `DisplayOrder` | Higher value renders on top |
| `IgnoreGuiInset` | Extend into the top safe area |

**UI frameworks:**

| Option | Best For |
|---|---|
| Vanilla Roblox UI | Simple, few screens |
| **Fusion** | Modern reactive UI, clean state management |
| **Roact** | React-style declarative UI (older, widely documented) |

**UI → Server communication rule:** LocalScript fires `RemoteEvent` → server validates → server updates authoritative state → replication propagates result to clients. Never apply gameplay state changes in a LocalScript based on UI input alone.

## StreamingEnabled

When `Workspace.StreamingEnabled = true` (recommended for large worlds), Instances stream in as players approach them. Client scripts cannot assume Workspace children exist at startup.

```lua
-- WRONG — assumes map loaded
local map = workspace.Map
local spawnPoint = map.SpawnPoint

-- CORRECT — wait with timeout
local map = workspace:WaitForChild("Map", 10)
if not map then
    warn("Map failed to load within timeout")
    return
end
```

Always provide a timeout to `WaitForChild`. An infinite wait silently hangs when assets fail to stream in.

## Networking Architecture for Multiplayer

Roblox enforces server-authoritative gameplay. Clients cannot be trusted for anything that affects game state.

**Authoritative flow:**
```
Client Input → RemoteEvent → Server Validates → Apply State → Replicate to Clients
```

### UnreliableRemoteEvent

`UnreliableRemoteEvent` has the same API as `RemoteEvent` (FireServer/FireClient/FireAllClients, OnServerEvent/OnClientEvent) but drops the guarantee of delivery and ordering. The engine can drop packets or deliver them out of order. In exchange, it has lower latency and lower bandwidth overhead.

**Use UnreliableRemoteEvent for:**
- Continuous position or state that will be overwritten on the next update anyway (character animations, projectile positions, HUD value pings)
- Visual effects where a missed packet is invisible (particle triggers, non-gameplay sounds)

**Use RemoteEvent (reliable) for:**
- Authoritative game state changes (damage dealt, item granted, round started)
- Anything the player needs to receive exactly once (a loot drop, a purchase confirmation)

```lua
-- ReplicatedStorage/Events/PositionSync (UnreliableRemoteEvent)
-- Client → Server: stream local position 10× per second
local RunService = game:GetService("RunService")
local positionSync = ReplicatedStorage.Events.PositionSync  -- UnreliableRemoteEvent

local accumulator = 0
RunService.Heartbeat:Connect(function(dt)
    accumulator += dt
    if accumulator >= 0.1 then
        accumulator = 0
        local root = Players.LocalPlayer.Character
            and Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        if root then
            positionSync:FireServer(root.Position, root.Orientation)
        end
    end
end)

-- Server: validate and apply (still validate — don't trust client position)
positionSync.OnServerEvent:Connect(function(player, position, orientation)
    -- Sanity check max displacement per tick before applying
    if isPositionValid(player, position) then
        updateNetworkOwnerPosition(player, position, orientation)
    end
end)
```

### Anti-Cheat Principles

- Validate everything: position, damage amounts, item quantities, timing
- Rate-limit remote event handlers per player
- Never send data to a client it doesn't need to see
- Sanity-check values against server-known bounds

```lua
-- Rate limiting remote handlers
local rateLimits: {[number]: number} = {}
local RATE_LIMIT_SECONDS = 0.1

remoteEvent.OnServerEvent:Connect(function(player: Player, ...)
    local now = os.clock()
    local lastCall = rateLimits[player.UserId] or 0

    if now - lastCall < RATE_LIMIT_SECONDS then
        return  -- Drop silently (don't kick on first violation)
    end
    rateLimits[player.UserId] = now

    handlePlayerAction(player, ...)
end)
```

### Throttling Remote Events

Remote events are rate-limited by the engine — firing too frequently drops events. Batch position/state updates.

```lua
-- Batch sync instead of per-frame remote calls
local SYNC_RATE = 0.1  -- 10 Hz

local accumulator = 0
RunService.Heartbeat:Connect(function(dt: number)
    accumulator += dt
    if accumulator >= SYNC_RATE then
        accumulator = 0
        syncAllPositions()  -- Single remote call with all data
    end
end)
```

## Performance Optimization

### Instance Count

- Roblox renders every `BasePart` — keep part count as low as possible
- Use `Union` operations to merge static geometry
- Enable `StreamingEnabled` for large maps
- Prefer `MeshPart` over `SpecialMesh` on a `Part`
- Use `Model:PivotTo()` for moving complex models (moves entire model, not individual parts)

### Object Pooling

```lua
local ObjectPool = {}
ObjectPool.__index = ObjectPool

function ObjectPool.new(template: Instance, initialSize: number)
    local self = setmetatable({}, ObjectPool)
    self._template = template
    self._pool = {}

    for _ = 1, initialSize do
        local obj = template:Clone()
        obj.Parent = nil
        table.insert(self._pool, obj)
    end

    return self
end

function ObjectPool:Get(parent: Instance): Instance
    local obj = table.remove(self._pool) or self._template:Clone()
    obj.Parent = parent
    return obj
end

function ObjectPool:Return(obj: Instance)
    obj.Parent = nil
    table.insert(self._pool, obj)
end
```

### Common Performance Traps

- **Write event-driven code** — avoid per-frame calculations whenever possible. At 60 FPS the entire frame budget is ~16.67 ms; even a small per-frame calculation compounds quickly
- **Break up long-running code** — a 100 ms operation running every frame caps you at 10 FPS. Split it into chunks using `task.wait()` or explore Parallel Luau (multithreading) for CPU-heavy work
- **Use built-in materials over custom textures** — built-in materials use far less memory. Reserve custom texture budget for visuals that are central to your game's identity
- **Don't store everything in `ReplicatedStorage`** — the client loads everything in that container. Move anything the client doesn't need to `ServerStorage`
- **Cache `game:GetService()`** at the top of each script — never call it in loops or per-frame
- **Cache Instance references** — `:FindFirstChild()` and `:WaitForChild()` are expensive in hot paths
- **Disconnect unused event listeners** — call `connection:Disconnect()` on any `RBXScriptConnection` that is no longer needed to stop unnecessary callbacks from firing
- **Avoid `Instance.new()` in tight loops** — pool reusable objects
- **String concatenation in loops** — use `table.concat(parts, "")` instead of `a .. b .. c`
- **Use `RunService.Heartbeat`** for server-side game loops, `RunService.RenderStepped` only for client rendering work
- **Test on lower-end devices early** — the Studio device emulator is not accurate for memory usage (it runs server + client together). Test on a real baseline mobile device throughout development, not just at the end

### Parallel Luau

For CPU-heavy work that doesn't need to modify the DataModel every frame, Parallel Luau distributes that work across multiple OS threads. The unit of isolation is an **Actor** — an `Instance` that wraps one or more scripts and executes them independently of other Actors.

**When to consider it:** NPC behavior for large numbers of characters, per-frame raycasting validation, procedural generation, pathfinding calculations.

```lua
-- Pattern: Actor-based parallel execution
-- In ServerScriptService, place each NPC's behavior Script under an Actor instance

-- NPC/BehaviorScript (Script under Actor)
local RunService = game:GetService("RunService")

RunService.Heartbeat:ConnectParallel(function()
    -- This callback runs in a parallel thread
    -- Safe: read instance properties that are "Read Parallel" or "Safe"
    local position = script.Parent.Parent.HumanoidRootPart.Position  -- read OK

    local nextTarget = computeNextTarget(position)  -- pure computation OK

    task.synchronize()  -- switch back to serial thread to write

    -- Safe to write properties now
    script.Parent.Parent.Humanoid:MoveTo(nextTarget)
end)
```

**Thread safety levels** (check the API reference for each property/function):

| Level | Can read in parallel | Can write in parallel |
|---|---|---|
| `Safe` | Yes | Yes |
| `Read Parallel` | Yes | No |
| `Local Safe` | Yes (own Actor only) | No (from other Actors) |
| `Unsafe` | No | No |

**Key constraints:**
- `require()` cannot be called inside a desynchronized (parallel) phase — require all modules before calling `task.desynchronize()`
- Scripts within the same Actor always execute sequentially — give each independently parallel unit its own Actor
- Use `task.desynchronize()` to enter parallel and `task.synchronize()` to return to serial

**Cross-Actor communication options:**
- `Actor:SendMessage(topic, ...)` / `script:BindToMessage(topic, fn)` — lightweight message passing between Actors
- `SharedTable` — a special table type that can be safely read/written from multiple Actors simultaneously (for sharing large amounts of data efficiently)
- Read-only DataModel access — all Actors can read `Read Parallel` properties directly

Parallel Luau adds real complexity. Profile first — most games never need it. Reserve it for bottlenecks that show up in Studio's MicroProfiler.

## Testing in Roblox

Roblox Studio has no CLI test runner. The official primary testing approach is Studio's built-in playtest modes:

| Mode | How to Access | What It Tests |
|---|---|---|
| **Play** | F5 or Play button | Local single-player — client + server on one machine |
| **Play Here** | Right-click a location | Spawns at a specific point |
| **Team Test** | Test tab → Team Test | Starts a real server + one or more connected clients; closest to production |
| **MCP `execute_luau`** | AI agent tool | Run assertions in the live game context without leaving the session |
| **`TestService`** | `game:GetService("TestService")` | Server-side assertion logging; used by CI integrations |

Studio also provides a **Client/Server toggle** during playtesting — switch the Explorer and Output between the server context and any client to inspect state on both sides.

For structured unit tests, **TestEZ** (maintained in the Roblox GitHub org at `Roblox/testez`) is the closest to an official framework:

```lua
-- TestEZ spec format
return function()
    describe("CombatService", function()
        it("reduces health on valid damage", function()
            local dummy = createDummyModel()
            CombatService.DealDamage(dummy, 50)
            expect(dummy.Humanoid.Health).to.equal(50)
        end)

        it("ignores damage on a dead humanoid", function()
            local dummy = createDeadDummy()
            CombatService.DealDamage(dummy, 100)
            expect(dummy.Humanoid.Health).to.equal(0)
        end)
    end)
end
```

## Tooling and Ecosystem

Studio is the only required tool for Roblox development. Everything below is optional — useful at certain scales or in certain workflows, but none of it is necessary to ship a game.

### Optional External Tooling

| Tool | What it adds | Worth it when... |
|---|---|---|
| **Rojo** | Syncs a filesystem project into Studio — enables `git` version control and external editor support | Team of 2+, or you want git history over Studio's built-in version history |
| **Wally** | Package manager for community libraries (analogous to npm) | You're adopting multiple community libraries and want dependency pinning |
| **Roblox LSP** | Luau language server for VS Code (type hints, autocomplete outside Studio) | You prefer writing code in an external editor alongside Rojo |
| **TestEZ** | Structured unit testing framework (Roblox GitHub org) | You want a spec-style test suite beyond Studio's playtest modes |

### Optional Community Libraries

These are not part of the Roblox engine. Each adds capability at the cost of a dependency:

| Library | What it adds | Worth it when... |
|---|---|---|
| **Fusion** | Reactive, state-driven UI | UI is complex enough that manual state sync becomes painful |
| **Roact** | React-style declarative UI | Team already knows React patterns |
| **ProfileService** | DataStore session locking + loss protection | Production game where data integrity is critical |
| **Knit** | Service/controller framework with auto-wired RemoteEvents | Larger codebase that benefits from enforced structure |

## AI-Assisted Development

Roblox Studio has an **MCP (Model Context Protocol)** server that gives AI agents direct access to the Studio Editor. When connected, an agent can inspect and modify your game without relying on chat descriptions.

Available tools through the Roblox Studio MCP (21 total):

| Tool | What it does |
|---|---|
| `explore_subagent` | Investigate the place in parallel and return a compact summary |
| `search_game_tree` | Explore the instance hierarchy as a flat JSON array |
| `inspect_instance` | Inspect any Instance's properties and children |
| `script_read` | Read a Script, LocalScript, or ModuleScript |
| `script_grep` | Search script content by pattern |
| `script_search` | Find scripts by name |
| `multi_edit` | Apply multiple edits to a script atomically |
| `execute_luau` | Run arbitrary Luau in the live game context |
| `console_output` | Retrieve output logs while the game is running |
| `start_stop_play` | Start or stop a playtest session |
| `playtest_subagent` | Spawn a test character that runs gameplay scenarios in its own context |
| `character_navigation` | Move the player character to a position or Instance |
| `keyboard_input` | Simulate key presses, key holds, and text input |
| `mouse_input` | Simulate mouse clicks, movement, and scrolling |
| `screen_capture` | Capture the current Studio viewport in Play mode |
| `generate_mesh` | Generate a textured 3D mesh from a text prompt |
| `generate_material` | Generate a custom material or texture |
| `generate_procedural_model` | Generate procedural models that scale and adapt automatically |
| `insert_from_creator_store` | Insert assets, plugins, and models from the Creator Store |
| `list_roblox_studios` | List connected Studio sessions |
| `set_active_studio` | Switch between connected Studio instances |

**Key advantage over other engine MCPs:** `execute_luau` lets you run Luau directly in the live game — inspect live state, test function behavior, fire remote events, and assert game invariants without leaving the AI session.

Full capability list, install steps, and supported clients are catalogued in `engine-mcps.yaml`.

## Roblox-Specific Architectural Constraints

Things that make Roblox genuinely different from other engines:

**No filesystem access** — Scripts cannot read or write files. All persistence goes through DataStore or external HTTP APIs (`HttpService:RequestAsync`). There is no equivalent to `File.ReadAllText`.

**Sandboxed Luau** — No FFI, no native libraries, no OS system calls. The security model is enforced by the engine. You cannot link C libraries or call into native code.

**Publishing is deployment** — There is no build/export step. Publishing to Roblox replaces your current live game. Studio has built-in version history; external `git` via Rojo is an option for teams that need it but is not required.

**Studio vs Production differences** — `DataStoreService` uses a Studio sandbox in edit mode and requires a published PlaceId in production. Some services behave differently. Always test `RunService:IsStudio()` guards carefully.

**Multi-place experiences** — A single Roblox experience can span multiple Places (e.g., Lobby + game world). Each Place has its own DataModel and server pool. Use `TeleportService` to move players between places. Data must be passed via DataStore or `TeleportService:TeleportAsync()` options.

**Avatar system** — Player characters are loaded per-player and re-created on every respawn. Always use `Players.CharacterAdded` to handle the character — never assume it exists at LocalScript startup.

**Client-authoritative character movement** — Roblox's default character controller is client-authoritative with server anti-cheat. Custom movement systems (vehicles, ability movement, zero-gravity) require explicit server reconciliation to avoid exploits.

**IP protection limits** — Content inside `ReplicatedStorage` and client `LocalScript`s can be accessed by determined players. Server-side logic in `ServerScriptService` is not sent to clients and is substantially safer.

## Licensing and Cost

Roblox development is **free**. There are no seat fees, no royalties, and no engine license.

- **Engine** — Free. No revenue thresholds or licensing tiers
- **Distribution** — Roblox handles PC, mobile (iOS/Android), Xbox, and VR deployment automatically. No App Store submissions
- **Revenue** — Games monetize through Robux (in-game currency). Roblox takes a platform cut; developers receive approximately 70% after marketplace fees via DevEx (Developer Exchange)
- **DevEx** — Convert earned Robux to USD. Requires a Premium membership and minimum Robux balance; verify current rates at the Roblox DevEx page

## When NOT to Use Roblox

Roblox is the wrong choice when:

- **Full engine control is required** — The engine is a black box. Rendering, physics, and engine internals cannot be modified
- **Complex custom shaders or rendering pipelines are needed** — Shader access is limited; custom render pipelines are not supported
- **Offline play is required** — Roblox requires internet connection and authentication for all play sessions
- **Non-Robux monetization is needed** — All commerce flows through Robux; you cannot accept credit cards or other payment methods directly
- **IP protection is critical** — Client-side code and ReplicatedStorage content can be accessed by players; not suitable for heavily proprietary logic on the client
- **The target audience is outside the Roblox ecosystem** — The platform has a specific demographic and social layer; this is a feature for some products and a constraint for others

## Architecture Decision Checklist

When starting a Roblox project, verify:

- [ ] Script organization: services in `ServerScriptService`, shared modules in `ReplicatedStorage`
- [ ] Data persistence: in-memory cache pattern implemented with `UpdateAsync`; session locking strategy via `MemoryStoreService` decided (or ProfileService community library evaluated)
- [ ] RemoteEvent/RemoteFunction layout defined, with rate limiting strategy; `UnreliableRemoteEvent` used for continuous/non-critical data (position sync, visual FX)
- [ ] `Workspace.SignalBehavior` set to `Deferred` (template default — verify it is set); event handler code audited for patterns that assume immediate fire
- [ ] FilteringEnabled confirmed on (default, never disable)
- [ ] `StreamingEnabled` evaluated for map size
- [ ] Multi-place strategy evaluated (TeleportService, cross-place data); cross-server messaging via `MessagingService` considered if global announcements or server browsers are needed
- [ ] UI approach decided: vanilla Roblox UI is the default; Fusion or Roact only if complexity warrants it
- [ ] Testing approach decided: Studio playtest and Team Test cover most needs; TestEZ or MCP `execute_luau` for structured unit tests
- [ ] External tooling evaluated: Rojo (git workflow), Wally (package management), Roblox LSP (external editor) — adopt only if the overhead is justified for your team size and workflow
- [ ] DataStore key naming strategy documented (especially for multi-place data sharing)
- [ ] Cleanup strategy established: explicit connection table pattern (or community cleanup library) applied consistently for all `RBXScriptConnection` lifecycles
