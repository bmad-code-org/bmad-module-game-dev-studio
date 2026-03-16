# Unity Engine Architecture Knowledge

## Overview

Unity is a commercial game engine using a component-based architecture with C# scripting. GameObjects hold Components that define behavior. Unity supports 2D and 3D, with rendering pipelines (URP, HDRP, Built-in), a mature asset store, and broad platform export.

## Architecture Fundamentals

### GameObject-Component Model

Unity's core pattern is Entity-Component: GameObjects are containers, Components add behavior.

**Key concepts:**

- **GameObject** — Empty container in a scene. Has a Transform. Holds components
- **MonoBehaviour** — Base class for custom components. Receives lifecycle callbacks
- **ScriptableObject** — Data container that lives as an asset, not in scenes
- **Prefab** — Reusable GameObject template. Can be instanced and overridden
- **Assembly Definition** — Code compilation unit that controls dependencies and compilation speed

**Typical hierarchy:**

```
Scene
├── --- MANAGERS ---
│   ├── GameManager
│   ├── AudioManager
│   └── UIManager
├── --- ENVIRONMENT ---
│   ├── Terrain / Tilemap
│   ├── Props
│   └── Lighting
├── --- ENTITIES ---
│   ├── Player
│   │   ├── PlayerController
│   │   ├── PlayerInput
│   │   ├── Health
│   │   ├── Rigidbody
│   │   └── Collider
│   └── Enemies
├── --- UI ---
│   ├── Canvas (HUD)
│   └── Canvas (Menus)
└── --- CAMERAS ---
    └── Main Camera + Cinemachine
```

### MonoBehaviour Lifecycle

Understanding execution order is critical for architecture:

```
Awake()           → Called once when object instantiates (before Start)
OnEnable()        → Called when object/component enabled
Start()           → Called once before first Update (after all Awake)
FixedUpdate()     → Called every physics step (default 50Hz)
Update()          → Called every frame
LateUpdate()      → Called every frame after all Updates
OnDisable()       → Called when object/component disabled
OnDestroy()       → Called when object destroyed
```

**Rules:**

- Use `Awake()` for self-initialization (setting up internal references)
- Use `Start()` for cross-object initialization (finding other objects)
- Use `FixedUpdate()` for physics and movement
- Use `LateUpdate()` for camera follow, UI updates after gameplay
- Never rely on execution order between MonoBehaviours without `Script Execution Order`

### Assembly Definitions

Split code into assemblies for faster compilation and clean dependency boundaries:

```
Assets/
├── Scripts/
│   ├── Core/
│   │   ├── Core.asmdef              → Core systems, no game dependencies
│   │   ├── EventSystem.cs
│   │   └── ServiceLocator.cs
│   ├── Gameplay/
│   │   ├── Gameplay.asmdef           → References: Core
│   │   ├── Player/
│   │   └── Combat/
│   ├── UI/
│   │   ├── UI.asmdef                 → References: Core, Gameplay
│   │   └── Screens/
│   └── Infrastructure/
│       ├── Infrastructure.asmdef     → References: Core
│       ├── SaveSystem.cs
│       └── NetworkManager.cs
└── Tests/
    ├── EditMode/
    │   └── EditModeTests.asmdef      → References: Core, Gameplay
    └── PlayMode/
        └── PlayModeTests.asmdef      → References: Core, Gameplay
```

**Benefits:**

- Changes to `UI.asmdef` don't recompile `Core` or `Gameplay`
- Enforces dependency direction (UI depends on Gameplay, not reverse)
- Required for testability (test assemblies reference game assemblies)

**Trade-offs on larger projects:**

- Managing dependency graphs across many assemblies adds overhead
- Circular dependency issues require restructuring (extract shared interfaces into a common assembly)
- Teams of 10+ people may spend significant time resolving asmdef reference issues
- Start simple (2-3 assemblies), split further only when compile times justify it

### ScriptableObject Architecture

ScriptableObjects are Unity's most powerful data pattern:

```csharp
[CreateAssetMenu(fileName = "NewWeapon", menuName = "Game/Weapon Stats")]
public class WeaponStats : ScriptableObject
{
    [Header("Base Stats")]
    public float baseDamage = 10f;
    public float attackSpeed = 1f;
    public float range = 5f;

    [Header("Scaling")]
    public AnimationCurve damageFalloff;

    public float DPS => baseDamage * attackSpeed;

    public float GetDamageAtRange(float distance)
    {
        float t = Mathf.Clamp01(distance / range);
        return baseDamage * damageFalloff.Evaluate(t);
    }
}
```

**Use ScriptableObjects for:**

- Item/weapon/enemy stat definitions
- Ability configurations
- Audio event definitions
- Game settings and tuning values
- Shared runtime data channels (ScriptableObject events pattern)

**ScriptableObject Event Channel pattern:**

```csharp
[CreateAssetMenu(menuName = "Events/Void Event")]
public class VoidEvent : ScriptableObject
{
    private readonly List<VoidEventListener> _listeners = new();

    public void Raise()
    {
        for (int i = _listeners.Count - 1; i >= 0; i--)
            _listeners[i].OnEventRaised();
    }

    public void Register(VoidEventListener listener) => _listeners.Add(listener);
    public void Unregister(VoidEventListener listener) => _listeners.Remove(listener);
}
```

## Coding Best Practices

### Component Communication

```csharp
// PREFERRED: GetComponent cached in Awake
private Health _health;
private Rigidbody _rb;

private void Awake()
{
    _health = GetComponent<Health>();
    _rb = GetComponent<Rigidbody>();
}

// AVOID: GetComponent in Update (expensive)
void Update()
{
    GetComponent<Health>().TakeDamage(1); // BAD
}

// For optional dependencies
private void Awake()
{
    if (TryGetComponent<AudioSource>(out var audio))
        _audioSource = audio;
}
```

### SerializeField vs Public

```csharp
// PREFERRED: Private with SerializeField (encapsulated but Inspector-visible)
[SerializeField] private float moveSpeed = 5f;
[SerializeField] private GameObject bulletPrefab;
[SerializeField] private Transform firePoint;

// Use Header and Tooltip for organization
[Header("Movement")]
[SerializeField] private float acceleration = 10f;
[SerializeField] private float deceleration = 15f;

[Header("Combat")]
[Tooltip("Seconds between attacks")]
[SerializeField, Range(0.1f, 5f)] private float attackCooldown = 0.5f;

// AVOID: public fields for serialization
public float speed = 5f; // Exposes internal state
```

### Coroutines vs Async/Await

```csharp
// Coroutines: good for frame-based sequences
private IEnumerator SpawnWave(int count, float interval)
{
    for (int i = 0; i < count; i++)
    {
        SpawnEnemy();
        yield return new WaitForSeconds(interval);
    }
}

// Async: good for I/O, network, complex async logic
// Requires UniTask (any Unity version) or Awaitable (Unity 2023.1+)
private async Awaitable LoadLevelAsync(string levelName)
{
    var op = SceneManager.LoadSceneAsync(levelName);
    while (!op.isDone)
    {
        OnLoadingProgress?.Invoke(op.progress);
        await Awaitable.NextFrameAsync();
    }
}

// Awaitable is built-in since Unity 2023.1 (also called Unity 6 in later branding):
await Awaitable.WaitForSecondsAsync(1f);
await Awaitable.NextFrameAsync();
// For Unity 2022 LTS and earlier, use UniTask instead
```

### Dependency Injection (Lightweight)

```csharp
// Simple service locator pattern (no framework needed)
public static class Services
{
    private static readonly Dictionary<Type, object> _services = new();

    public static void Register<T>(T service) where T : class
        => _services[typeof(T)] = service;

    public static T Get<T>() where T : class
        => _services[typeof(T)] as T;
}

// Register in bootstrap scene
Services.Register<IAudioManager>(audioManager);
Services.Register<ISaveSystem>(saveSystem);

// Or use VContainer / Zenject for complex projects
```

### Project Structure Conventions

```
Assets/
├── Art/
│   ├── Sprites/
│   ├── Models/
│   ├── Materials/
│   ├── Textures/
│   ├── Animations/
│   └── Shaders/
├── Audio/
│   ├── Music/
│   ├── SFX/
│   └── Mixers/
├── Prefabs/
│   ├── Characters/
│   ├── Environment/
│   ├── UI/
│   └── VFX/
├── Scenes/
│   ├── Boot.unity
│   ├── MainMenu.unity
│   └── Levels/
├── Scripts/
│   ├── Core/
│   ├── Gameplay/
│   ├── UI/
│   └── Infrastructure/
├── Data/                    # ScriptableObject instances
│   ├── Items/
│   ├── Enemies/
│   └── Config/
├── Resources/               # Only for runtime-loaded assets
├── StreamingAssets/          # Platform-specific raw files
├── Plugins/                  # Native plugins
└── Tests/
    ├── EditMode/
    └── PlayMode/
```

**Naming conventions:**

- Files and folders: `PascalCase` (e.g., `PlayerController.cs`, `MainMenu.unity`)
- C# classes: `PascalCase`
- Methods: `PascalCase`
- Private fields: `_camelCase` with underscore prefix
- Parameters and locals: `camelCase`
- Constants: `PascalCase` or `UPPER_SNAKE_CASE`
- Interfaces: `IPascalCase` prefix

## Performance Optimization

### General Guidelines

- **Avoid `Find()` at runtime** — cache references in `Awake()`/`Start()`
- **Use object pooling** for frequently spawned objects (bullets, effects, enemies)
- **Minimize `Instantiate`/`Destroy`** in gameplay — pool instead
- **Avoid allocations in Update** — no `new`, `string` concatenation, LINQ, or `foreach` on non-List collections

### Rendering (URP Focus)

- **Static Batching** — enable for non-moving objects
- **GPU Instancing** — enable on materials for many identical objects
- **SRP Batcher** — enabled by default in URP, keep shaders compatible
- **Sprite Atlas** — combine 2D sprites to reduce draw calls
- **LOD Groups** — for 3D with multiple detail levels
- **Occlusion Culling** — bake for indoor/complex 3D scenes

### Physics Performance

- **Use layers** — configure collision matrix to skip unnecessary checks
- **Prefer simple colliders** — Sphere/Capsule over Mesh
- **Use `CompareTag()`** not `tag ==` (avoids string allocation)
- **Rigidbody.isKinematic** — for objects moved by code, not physics
- **Fixed Timestep** — default 0.02s (50Hz) is fine for most games; increase for precision

### Memory and Loading

- **Addressables** — async asset loading with memory management for larger projects
- **Resources folder** — only for assets that MUST be loaded by string path at runtime
- **Asset Bundles** — for DLC, mods, or reducing initial build size
- **Async scene loading:**

```csharp
private IEnumerator LoadSceneAsync(string sceneName)
{
    AsyncOperation op = SceneManager.LoadSceneAsync(sceneName);
    op.allowSceneActivation = false;

    while (op.progress < 0.9f)
    {
        loadingBar.value = op.progress;
        yield return null;
    }

    // Ready to activate
    op.allowSceneActivation = true;
}
```

### C# Performance Tips

- **Structs for small, short-lived data** (Vector math, ray results)
- **`Span<T>` and `stackalloc`** for temporary buffers (advanced)
- **`NativeArray<T>`** with Jobs system for parallel computation
- **Avoid boxing** — don't pass value types as `object`
- **Use `StringBuilder`** for string building in loops

## Popular Third-Party Packages

### Essential Development

| Package | Purpose | Source |
|---|---|---|
| **DOTween** | Tweening and animation | Asset Store (free) |
| **UniTask** | Zero-allocation async/await | OpenUPM / GitHub |
| **NaughtyAttributes** | Inspector enhancement (buttons, labels) | OpenUPM / GitHub |
| **Odin Inspector** | Advanced Inspector/serialization | Asset Store (paid) |
| **VContainer** | Lightweight DI container | OpenUPM |

### Gameplay Systems

| Package | Purpose | Source |
|---|---|---|
| **Cinemachine** | Camera system (Unity package) | Package Manager |
| **Input System** | Modern input handling (Unity package) | Package Manager |
| **ProBuilder** | In-editor level prototyping (Unity package) | Package Manager |
| **Yarn Spinner** | Dialogue and branching narrative | OpenUPM / GitHub |
| **Ink** | Narrative scripting language | GitHub + Unity integration |

### Visual and Audio

| Package | Purpose | Source |
|---|---|---|
| **Shader Graph** | Visual shader authoring (Unity package) | Package Manager |
| **VFX Graph** | GPU particle system (Unity package) | Package Manager |
| **FMOD for Unity** | Professional audio middleware | FMOD website (free for indie) |
| **Master Audio** | Audio management suite | Asset Store (paid) |
| **Feel / Nice Vibrations** | Game juice and haptic feedback | Asset Store |

### Networking

| Package | Purpose | Source |
|---|---|---|
| **Netcode for GameObjects** | Unity's official networking | Package Manager |
| **Photon Fusion/Quantum** | Hosted multiplayer solution | Photon website |
| **Mirror** | Open-source networking (Fork of UNet) | Asset Store (free) |
| **FishNet** | High-performance open-source networking | Asset Store (free) |
| **Steamworks.NET** | Steam API integration | GitHub |

### Infrastructure

| Package | Purpose | Source |
|---|---|---|
| **Addressables** | Async asset management (Unity package) | Package Manager |
| **Unity Test Framework** | NUnit-based testing (Unity package) | Package Manager |
| **Zenject/Extenject** | Full DI framework | OpenUPM |
| **R3 (Reactive Extensions)** | Reactive programming (successor to UniRx) | NuGet / GitHub |
| **MessagePipe** | High-performance pub/sub messaging | NuGet / GitHub |

## Common Architectural Patterns

### Manager Singleton Pattern

```csharp
public class GameManager : MonoBehaviour
{
    public static GameManager Instance { get; private set; }

    private void Awake()
    {
        if (Instance != null && Instance != this)
        {
            Destroy(gameObject);
            return;
        }
        Instance = this;
        DontDestroyOnLoad(gameObject);
    }
}
```

**Use sparingly.** Prefer ScriptableObject events or dependency injection for decoupling.

### Object Pool Pattern

```csharp
public class ObjectPool<T> where T : Component
{
    private readonly Queue<T> _pool = new();
    private readonly T _prefab;
    private readonly Transform _parent;

    public ObjectPool(T prefab, int initialSize, Transform parent = null)
    {
        _prefab = prefab;
        _parent = parent;
        for (int i = 0; i < initialSize; i++)
            _pool.Enqueue(CreateInstance());
    }

    public T Get()
    {
        T obj = _pool.Count > 0 ? _pool.Dequeue() : CreateInstance();
        obj.gameObject.SetActive(true);
        return obj;
    }

    public void Return(T obj)
    {
        obj.gameObject.SetActive(false);
        _pool.Enqueue(obj);
    }

    private T CreateInstance()
    {
        T obj = Object.Instantiate(_prefab, _parent);
        obj.gameObject.SetActive(false);
        return obj;
    }
}
```

### Bootstrap Scene Pattern

Use a lightweight entry scene that initializes systems before loading gameplay:

```csharp
// Boot.unity scene with single BootLoader GameObject
public class BootLoader : MonoBehaviour
{
    [SerializeField] private string firstSceneName = "MainMenu";

    private async void Start()
    {
        // Initialize systems
        await InitializeSaveSystem();
        await InitializeAudio();
        InitializeAnalytics();

        // Load first real scene
        SceneManager.LoadScene(firstSceneName);
    }
}
```

## Rendering Pipeline Selection

| Pipeline | Use Case | Trade-offs |
|---|---|---|
| **URP** | Most projects, mobile, 2D, stylized 3D | Good performance, extensible, growing feature set |
| **HDRP** | High-fidelity 3D, PC/console | Best visuals, highest hardware requirements |
| **Built-in** | Legacy projects only | Deprecated for new projects, limited future support |

**Default recommendation:** URP for all new projects unless targeting AAA-quality visuals on high-end hardware.

## Licensing and Cost

Unity uses a **subscription model** with tiers based on revenue:

- **Unity Personal** — Free for revenue/funding under $200K in the last 12 months. Includes "Made with Unity" splash screen
- **Unity Pro** — Required above $200K revenue. Per-seat subscription fee
- **Unity Enterprise** — For larger organizations, custom pricing

**Important cost considerations:**

- **Unity Runtime Fee** (introduced 2023, revised 2024) — Unity has restructured its pricing model. Verify current terms via WebSearch before making engine commitments, as this has changed multiple times
- No royalty on game revenue (unlike Unreal's 5% above $1M)
- Asset Store purchases are separate costs
- Some Unity packages (Havok Physics, Sentis) have their own licensing terms

## When NOT to Use Unity

Unity may be the wrong choice when:

- **Cutting-edge 3D visuals are the primary selling point** — HDRP is capable but Unreal's Nanite/Lumen pipeline is ahead for photorealistic rendering
- **Extremely large open worlds** — Unity's scene management and streaming require significant custom work compared to Unreal's World Partition
- **Budget is zero and must remain zero** — While Personal is free, many essential workflow packages (Odin, Rewired) are paid, and the ecosystem assumes some Asset Store spending
- **Team strongly prefers open-source** — Unity's source is available via license but it is not open-source; Godot is if this matters
- **Pricing stability is critical** — Unity's licensing terms have changed significantly in recent years; evaluate current terms carefully

## Platform-Specific Caveats

- **iOS:** IL2CPP is mandatory for App Store submission. IL2CPP code stripping can remove code used via reflection — configure `link.xml` to preserve types. Metal shader compilation adds to build time
- **Android:** Minimum API level requirements change yearly. Vulkan support varies by device — always test on low-end Android hardware. App Bundle size limits (200MB AAB) may require Play Asset Delivery
- **WebGL:** No threading support (no `Task.Run`). Memory is limited by browser tab. Many .NET APIs unavailable. Build sizes can be large — enable compression
- **Consoles:** Require platform-specific SDK access (PlayStation Partners, Nintendo Developer, ID@Xbox). Built-in console export but certification testing is extensive. Each platform has unique TRC/TFR requirements
- **Steam Deck:** Essentially Linux — test with Proton compatibility. Input must work with Steam Input API

## Architecture Decision Checklist

When choosing Unity for a project, verify:

- [ ] Rendering pipeline selected (URP recommended for most projects)
- [ ] Input System: new Input System package vs legacy Input Manager
- [ ] Assembly Definition structure planned for code organization
- [ ] ScriptableObject data architecture for game data
- [ ] Networking solution if multiplayer (Netcode, Photon, Mirror, FishNet)
- [ ] Audio approach: native Unity Audio, FMOD, or Wwise
- [ ] Asset loading strategy: Resources, Addressables, or Asset Bundles
- [ ] DI approach: manual, VContainer, or Zenject
- [ ] Key Asset Store packages identified and license-compatible
- [ ] Target Unity version selected (prefer latest LTS)
