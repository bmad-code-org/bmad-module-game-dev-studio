# Unreal Engine Architecture Knowledge

## Overview

Unreal Engine is a commercial game engine using an Actor-Component architecture with C++ and Blueprints. It features production-grade rendering (Nanite, Lumen), a Gameplay Framework with built-in multiplayer replication, and comprehensive tooling for large-scale game development.

## Architecture Fundamentals

### Actor-Component Model

Unreal's core architecture is based on Actors (entities in the world) and Components (modular functionality).

**Key concepts:**

- **UObject** — Base class for all Unreal objects. Provides reflection, serialization, garbage collection
- **AActor** — Any object placed in a level. Has a transform, can have components
- **UActorComponent** — Logic-only component (no transform)
- **USceneComponent** — Component with transform, can be attached to hierarchy
- **APawn** — Actor that can be possessed by a controller
- **ACharacter** — Pawn with movement component, capsule, skeletal mesh
- **APlayerController** — Receives input, possesses pawns
- **AGameModeBase** — Defines game rules, exists only on server in multiplayer. Use `AGameModeBase` for simple games; use `AGameMode` (subclass) when you need match state management (warmup, in-progress, post-match)

### Gameplay Framework

Unreal provides an opinionated framework for game structure:

```
UGameInstance                    → Persistent across level loads
├── AGameModeBase               → Game rules (server-only in MP)
│   └── AGameStateBase          → Replicated game state
├── APlayerController           → Player's input and camera
│   └── APlayerState            → Replicated per-player state
├── APawn / ACharacter          → Player's physical representation
└── AHUD                        → Player's heads-up display
```

**When to use what:**

| Class | Purpose | Lifetime |
|---|---|---|
| `UGameInstance` | Persistent data, level transitions, subsystems | Entire application |
| `AGameModeBase` | Spawn rules, basic game flow | Per level (server only) |
| `AGameMode` (subclass) | Adds match states (warmup, playing, post-match) | Per level (server only) |
| `AGameState` | Match score, timer, team info | Per level (replicated) |
| `APlayerController` | Input handling, UI ownership, camera | Per player connection |
| `APlayerState` | Player score, name, team | Per player (replicated) |
| `APawn/ACharacter` | Physical entity in world | Can respawn |

### Module System

Unreal projects are organized into modules:

```
Source/
├── MyGame/                      # Primary game module
│   ├── MyGame.Build.cs          # Module build rules
│   ├── MyGame.h / MyGame.cpp    # Module implementation
│   ├── Core/                    # Core systems
│   ├── Gameplay/                # Gameplay mechanics
│   ├── UI/                      # User interface
│   └── AI/                      # AI systems
├── MyGameEditor/                # Editor-only module (optional)
│   └── MyGameEditor.Build.cs
└── MyGameTests/                 # Test module
    └── MyGameTests.Build.cs
```

**Build.cs dependencies:**

```csharp
PublicDependencyModuleNames.AddRange(new string[] {
    "Core",
    "CoreUObject",
    "Engine",
    "InputCore",
    "EnhancedInput",
    "GameplayAbilities",    // If using GAS
    "GameplayTags",
    "GameplayTasks"
});
```

### Blueprint vs C++ Decision Guide

| Factor | C++ | Blueprints |
|---|---|---|
| **Performance** | Maximum performance | Slight overhead per node |
| **Compilation** | Slow compile, fast runtime | Instant iteration |
| **Complexity** | Any complexity | Gets unwieldy for complex logic |
| **Team access** | Programmers only | Designers and artists too |
| **Engine access** | Full API access | Most API exposed |
| **Debugging** | Standard C++ debuggers | Visual debugger in editor |
| **Source control** | Text-based diffs | Binary assets (harder to merge) |

**Blueprint binary merge conflict warning:**

Blueprints are binary assets (`.uasset`) and **cannot be text-merged**. Two developers editing the same Blueprint simultaneously will produce a merge conflict that must be resolved by discarding one person's work. Mitigations:

- **Use Perforce** (or Git LFS with file locking) for teams — lock Blueprint files during editing
- **Avoid monolithic Blueprints** — split large GameMode/GameState Blueprints into smaller, single-responsibility Blueprints
- **Put shared logic in C++** — anything touched by multiple developers should be in C++ (text-mergeable)
- **Use Blueprint Function Libraries** over editing core Blueprints for new utility functions
- **Communicate** — coordinate who is editing which Blueprint on small teams using Git

**Recommended hybrid approach:**

- **C++** for: base classes, core systems, performance-critical code, networking, AI logic
- **Blueprints** for: prototyping, UI, level scripting, designer-tunable parameters, VFX triggers
- **Pattern:** Create C++ base classes, expose variables and functions to Blueprints, let Blueprints handle configuration and scripting

```cpp
// C++ base class
UCLASS(Blueprintable)
class AWeaponBase : public AActor
{
    GENERATED_BODY()

protected:
    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Weapon")
    float BaseDamage = 10.f;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Weapon")
    float FireRate = 0.5f;

    // C++ implementation, callable from Blueprint
    UFUNCTION(BlueprintCallable, Category = "Weapon")
    void Fire();

    // Blueprint can override this
    UFUNCTION(BlueprintNativeEvent, Category = "Weapon")
    void OnFire();
    virtual void OnFire_Implementation();
};

// Then create BP_Shotgun, BP_Rifle, etc. in Blueprints
// inheriting from AWeaponBase
```

## Coding Best Practices

### UPROPERTY Specifiers

```cpp
// Visible in Editor, read-only at runtime
UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Components")
UStaticMeshComponent* Mesh;

// Editable per-instance in Editor
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Stats")
float MaxHealth = 100.f;

// Editable only on the Blueprint default, not per-instance
UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Config")
TSubclassOf<AProjectile> ProjectileClass;

// Not visible in Editor, replicated in multiplayer
UPROPERTY(Replicated, BlueprintReadOnly, Category = "State")
int32 CurrentAmmo;

// Replicated with notification
UPROPERTY(ReplicatedUsing = OnRep_Health)
float Health;

void OnRep_Health();
```

### UFUNCTION Specifiers

```cpp
// Callable from Blueprints
UFUNCTION(BlueprintCallable, Category = "Combat")
void TakeDamage(float Amount, AActor* Instigator);

// Can be overridden in Blueprints
UFUNCTION(BlueprintNativeEvent, Category = "Combat")
void OnDeath();

// Pure function (no side effects, no exec pin in BP)
UFUNCTION(BlueprintPure, Category = "Stats")
float GetHealthPercent() const;

// Server RPC (multiplayer)
UFUNCTION(Server, Reliable)
void ServerFire(FVector Location, FRotator Rotation);

// Client RPC
UFUNCTION(Client, Reliable)
void ClientShowHitMarker();

// Multicast RPC
UFUNCTION(NetMulticast, Unreliable)
void MulticastPlayFireEffect();
```

### Smart Pointers and Memory

```cpp
// Use TObjectPtr for UPROPERTY members (UE5+ only — use raw pointers in UE4)
UPROPERTY()
TObjectPtr<UStaticMeshComponent> MeshComp;

// Use TWeakObjectPtr for non-owning references
TWeakObjectPtr<AActor> TargetActor;

// Use TSoftObjectPtr for assets loaded on demand
UPROPERTY(EditDefaultsOnly)
TSoftObjectPtr<UTexture2D> IconTexture;

// Use TSharedPtr/TUniquePtr for non-UObject data
TSharedPtr<FMyDataStructure> SharedData;
TUniquePtr<FMyProcessor> Processor;

// NEVER use raw new/delete for UObjects — use NewObject/CreateDefaultSubobject
UMyComponent* Comp = CreateDefaultSubobject<UMyComponent>(TEXT("MyComp"));
UMyObject* Obj = NewObject<UMyObject>(this);
```

### Enhanced Input System

```cpp
// Input Action setup (C++)
UPROPERTY(EditDefaultsOnly, Category = "Input")
UInputAction* MoveAction;

UPROPERTY(EditDefaultsOnly, Category = "Input")
UInputAction* JumpAction;

UPROPERTY(EditDefaultsOnly, Category = "Input")
UInputMappingContext* DefaultMappingContext;

void AMyCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
    auto* EIC = CastChecked<UEnhancedInputComponent>(PlayerInputComponent);
    EIC->BindAction(MoveAction, ETriggerEvent::Triggered, this, &AMyCharacter::Move);
    EIC->BindAction(JumpAction, ETriggerEvent::Started, this, &AMyCharacter::StartJump);
}

void AMyCharacter::Move(const FInputActionValue& Value)
{
    FVector2D Input = Value.Get<FVector2D>();
    AddMovementInput(GetActorForwardVector(), Input.Y);
    AddMovementInput(GetActorRightVector(), Input.X);
}
```

### Project Structure Conventions

```
Source/
├── MyGame/
│   ├── MyGame.Build.cs
│   ├── Core/
│   │   ├── MyGameInstance.h/.cpp
│   │   ├── MyGameMode.h/.cpp
│   │   └── MyGameState.h/.cpp
│   ├── Characters/
│   │   ├── MyCharacterBase.h/.cpp
│   │   └── Components/
│   │       ├── HealthComponent.h/.cpp
│   │       └── CombatComponent.h/.cpp
│   ├── Weapons/
│   │   ├── WeaponBase.h/.cpp
│   │   └── ProjectileBase.h/.cpp
│   ├── AI/
│   │   ├── AIControllerBase.h/.cpp
│   │   └── BehaviorTrees/
│   ├── UI/
│   │   ├── HUD/
│   │   └── Widgets/
│   ├── Data/
│   │   ├── DataTables/
│   │   └── DataAssets/
│   └── Utils/
Content/
├── Blueprints/
│   ├── Characters/
│   ├── Weapons/
│   ├── AI/
│   └── UI/
├── Maps/
├── Art/
│   ├── Characters/
│   ├── Environment/
│   ├── VFX/
│   └── Materials/
├── Audio/
│   ├── Music/
│   ├── SFX/
│   └── MetaSounds/
├── Data/
│   ├── DataTables/
│   └── Curves/
└── Input/
    ├── Actions/
    └── Contexts/
```

**Naming conventions:**

- C++ files: `PascalCase` matching class name
- C++ classes: Prefix with `A` (Actor), `U` (UObject/Component), `F` (struct), `E` (enum), `I` (interface), `T` (template)
- Blueprints: `BP_PascalCase` (e.g., `BP_PlayerCharacter`)
- Materials: `M_Name` or `MI_Name` (instance)
- Textures: `T_Name_Suffix` (e.g., `T_Brick_D` for diffuse, `_N` for normal)
- Widget Blueprints: `WBP_Name`
- Data Tables: `DT_Name`
- Input Actions: `IA_Name`
- Input Mapping Contexts: `IMC_Name`

## Performance Optimization

### General Guidelines

- **Use `Tick` sparingly** — disable tick on actors that don't need per-frame updates (`PrimaryActorTick.bCanEverTick = false`)
- **Timers over Tick** — for periodic checks, use `GetWorldTimerManager().SetTimer()`
- **Avoid `TActorIterator` in Tick** — cache references or use subsystems
- **Use Gameplay Tags** over string comparisons for type checking

### Rendering Performance

- **Nanite** — use for high-poly static meshes (automatic LOD, no manual LOD setup)
- **Lumen** — dynamic GI; switch to baked lighting for lower-end targets
- **Virtual Shadow Maps** — pair with Nanite for automatic shadow LOD
- **HLOD (Hierarchical LOD)** — for open world, auto-combines distant objects
- **Instanced Static Meshes** — for foliage, debris, repeated objects
- **World Partition** — streaming system for open worlds (replaces World Composition)
- **Niagara** — use over Cascade for VFX (GPU-accelerated)

### Physics Performance

- **Simple collision** over complex for gameplay actors
- **Collision channels** — configure precisely, avoid `BlockAll` defaults
- **Physics substepping** — enable for fast objects; set max substeps
- **Async physics** — available in UE5 for background physics computation
- **Sweep over raycast** for character movement (built into Character Movement Component)

### Memory and Loading

- **Soft references** (`TSoftObjectPtr`) for assets not always needed
- **Async loading** with `FStreamableManager`:

```cpp
FStreamableManager& StreamableManager = UAssetManager::GetStreamableManager();
StreamableManager.RequestAsyncLoad(AssetPath,
    FStreamableDelegate::CreateUObject(this, &AMyActor::OnAssetLoaded));
```

- **Level streaming** for large worlds (World Partition in UE5)
- **Garbage collection** — avoid creating many UObjects per frame
- **Object pooling** for projectiles, effects, AI actors

### C++ Performance Tips

- **Avoid `FString` operations in hot paths** — use `FName` for comparisons
- **Use `TInlineAllocator`** for small arrays: `TArray<FHitResult, TInlineAllocator<4>>`
- **`FORCEINLINE`** for small, frequently-called functions
- **Avoid virtual function calls** in tight loops — use templates or data-oriented design
- **Use `ParallelFor`** for parallelizable work

## Popular Third-Party Plugins

### Essential Development

| Plugin | Purpose | Source |
|---|---|---|
| **Gameplay Ability System (GAS)** | Abilities, attributes, effects framework | Engine built-in (module) |
| **Common UI** | Cross-platform UI framework with input routing | Engine plugin |
| **Enhanced Input** | Modern input handling | Engine built-in |
| **Mass Entity (Mass AI)** | Large-scale entity simulation (ECS-like) | Engine (production since UE 5.4) |
| **Online Subsystem Steam** | Steamworks integration | Engine built-in |

### Gameplay Systems

| Plugin | Purpose | Source |
|---|---|---|
| **Gameplay Ability System** | RPG stats, buffs, cooldowns, combos | Epic (included) |
| **ALS (Advanced Locomotion System)** | Production-quality character movement | Marketplace (community) |
| **Narrative** | Branching dialogue and quest system | Marketplace |
| **Voxel Plugin** | Voxel terrain generation and editing | Marketplace |
| **Easy Multi Save** | Save/load system with serialization | Marketplace |

### Visual and Audio

| Plugin | Purpose | Source |
|---|---|---|
| **MetaSounds** | Procedural audio graph system | Engine built-in |
| **FMOD for Unreal** | Professional audio middleware | FMOD website (free for indie) |
| **Wwise** | AAA audio middleware | Audiokinetic website |
| **Niagara** | GPU particle and VFX system | Engine built-in |
| **Water System** | Ocean and water body rendering | Engine plugin |

### Networking and Backend

| Plugin | Purpose | Source |
|---|---|---|
| **Online Subsystem** | Platform abstraction for networking | Engine built-in |
| **Epic Online Services (EOS)** | Auth, matchmaking, lobbies, voice, anti-cheat | Epic (free) |
| **Steamworks** | Steam API (achievements, leaderboards, P2P) | Engine + Steamworks SDK |
| **PlayFab** | Cloud backend (analytics, leaderboards, economy) | Microsoft (free tier) |
| **Nakama** | Open-source game server | Docker + Unreal SDK |

### Tools and Infrastructure

| Plugin | Purpose | Source |
|---|---|---|
| **Gameplay Debugger** | In-game AI and gameplay visualization | Engine built-in |
| **Unreal Insights** | Profiling and telemetry | Engine built-in |
| **Gauntlet** | Automated testing framework | Engine built-in |
| **Horde** | Distributed build system | Epic (for large teams) |

## Common Architectural Patterns

### Subsystem Pattern

Subsystems are engine-managed singletons tied to a specific lifetime:

```cpp
// Game Instance Subsystem — lives for entire application
UCLASS()
class UInventorySubsystem : public UGameInstanceSubsystem
{
    GENERATED_BODY()

public:
    virtual void Initialize(FSubsystemCollectionBase& Collection) override;

    UFUNCTION(BlueprintCallable)
    void AddItem(FName ItemID, int32 Count = 1);

    UFUNCTION(BlueprintPure)
    int32 GetItemCount(FName ItemID) const;

private:
    TMap<FName, int32> Inventory;
};

// Access from anywhere:
UInventorySubsystem* Inv = GetGameInstance()->GetSubsystem<UInventorySubsystem>();
```

Available lifetimes: `UEngineSubsystem`, `UEditorSubsystem`, `UGameInstanceSubsystem`, `UWorldSubsystem`, `ULocalPlayerSubsystem`

### Data-Driven Design with Data Tables

```cpp
// Define row structure
USTRUCT(BlueprintType)
struct FWeaponData : public FTableRowBase
{
    GENERATED_BODY()

    UPROPERTY(EditAnywhere)
    float Damage = 10.f;

    UPROPERTY(EditAnywhere)
    float FireRate = 1.f;

    UPROPERTY(EditAnywhere)
    TSoftObjectPtr<UStaticMesh> WeaponMesh;

    UPROPERTY(EditAnywhere)
    TSubclassOf<AProjectile> ProjectileClass;
};

// Look up at runtime
if (FWeaponData* Data = WeaponTable->FindRow<FWeaponData>(WeaponID, TEXT("")))
{
    Damage = Data->Damage;
}
```

### Gameplay Ability System (GAS) Overview

GAS is Unreal's built-in framework for RPG-style abilities:

- **Ability System Component** — attached to actors that use abilities
- **Gameplay Abilities** — define what happens (castable actions)
- **Gameplay Effects** — modify attributes (damage, buffs, duration)
- **Gameplay Attributes** — numeric values (health, mana, strength)
- **Gameplay Tags** — hierarchical labels for state and filtering
- **Gameplay Cues** — cosmetic responses to gameplay events

**Use GAS when:** RPG systems, complex buff/debuff, ability cooldowns, attribute interactions, multiplayer-replicated combat.

**Skip GAS when:** Simple action games, platformers, puzzle games — the overhead isn't justified.

## Licensing and Cost

Unreal Engine uses a **royalty model:**

- **Free to use** for development and learning
- **5% royalty** on gross revenue after the first **$1,000,000 USD** per product per calendar year
- **Custom license** available for large studios (negotiated with Epic, typically eliminates royalty)
- **Epic Games Store exclusivity** can waive the royalty entirely

**Cost implications:**

- No per-seat fees — entire team uses it for free during development
- The $1M threshold means most indie projects pay nothing
- Marketplace assets have their own costs (some free monthly assets via subscription)
- Console platform fees are separate from engine royalty
- Verify current royalty terms via WebSearch — Epic has modified terms historically

## When NOT to Use Unreal

Unreal may be the wrong choice when:

- **2D-only game** — Unreal's 2D support (Paper2D) is minimal and largely unmaintained. Godot, Unity, or Phaser are far better choices for 2D
- **Small team building a simple game** — Unreal's complexity, build times, and project size add overhead that isn't justified for simple projects
- **Web/browser target** — Unreal has no practical web export path. Use Phaser or Godot for browser games
- **Rapid prototyping with quick iteration** — C++ compile times and Editor startup time slow iteration compared to Godot or Unity
- **Mobile-first casual game** — Unreal's minimum build size and resource consumption are high for casual mobile. Unity or Godot are more appropriate
- **Solo developer with no C++ experience** — While Blueprints allow visual scripting, performance-critical or complex systems still need C++. The learning curve is steep

## Platform-Specific Caveats

- **Console:** Unreal has built-in console export but requires platform SDK access (PlayStation Partners, Nintendo Developer, ID@Xbox). Each platform has extensive certification requirements (TRC/TFR). Build times for console targets are significantly longer
- **Mobile:** Minimum APK/IPA sizes are large (~100MB+). Thermal throttling is a major concern — profile on real devices. Vulkan vs OpenGL ES choice affects performance significantly
- **Steam Deck:** Works well but requires testing with the Steam Deck's input and performance profile. Forward rendering may be needed for performance
- **Source control:** Perforce is strongly recommended for teams due to binary asset locking. Git LFS works for small teams but lacks file locking. Plan source control strategy early — migrating later is painful

## Architecture Decision Checklist

When choosing Unreal for a project, verify:

- [ ] C++ to Blueprint ratio planned for the team
- [ ] Rendering features needed: Nanite, Lumen, Virtual Shadow Maps, or forward rendering
- [ ] World Partition needed for open world (vs standard level streaming)
- [ ] Gameplay Ability System appropriate for combat/RPG systems
- [ ] Networking model: dedicated servers, listen servers, or P2P
- [ ] Enhanced Input configured for target platforms
- [ ] Audio approach: MetaSounds, FMOD, or Wwise
- [ ] Target Unreal version selected (latest stable recommended)
- [ ] Build pipeline: local, Unreal Build Tool, or Horde for CI
- [ ] Source control strategy for binary assets (Perforce recommended for large teams, Git LFS for small)
