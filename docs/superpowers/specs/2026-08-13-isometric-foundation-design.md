# Isometric foundation design

## Scope

Build the first playable Unity foundation only: a Bootstrap entry scene, a
small TestChamber, a CharacterController player, fixed orthographic camera,
screen-space mouse/gamepad aiming, aim marker, and HDRP flashlight. Keep
OutdoorsScene as untouched reference content. Do not add caves, combat,
projectiles, drones, inventory, extraction, multiplayer, cutaway rendering,
or final downscaling.

## Approach

Use one idempotent editor setup entry point, `ProjectSetup.Configure`, for
serialized project authoring. It will create missing folders, input actions,
layers, collision exceptions, scenes, prefab references, and Build Settings,
then save and refresh through Unity APIs. Re-running it finds existing objects
by stable names and replaces only the assets it owns. Existing template assets,
scenes, packages, and unrelated worktree edits are not touched.

The editor entry point lives at `Assets/_Game/Editor/ProjectSetup.cs`. Runtime
scripts live under `Assets/_Game/Scripts/Core`, `Player`, and `Camera`.

Use small runtime components with direct Unity APIs:

- `BootstrapLoader` loads `TestChamber` when the application starts.
- `PlayerMovement` reads the `Move` action and moves a CharacterController in
  camera-relative screen space with acceleration, deceleration, gravity, and
  collision handled by `CharacterController.Move`.
- `IsometricCameraFollow` owns a fixed orthographic camera, smooth follow, and
  a modest aim look-ahead hook.
- `PlayerAiming` intersects a camera ray with the floor plane for mouse aim or
  projects the gamepad right stick into camera-relative world space. It exposes
  one aim target and direction for other components.
- `FlashlightController` follows that shared aim direction and owns one HDRP
  spot light with shadows.

Cinemachine remains installed and verified, but a custom fixed camera is the
smallest correct implementation for this milestone. A Cinemachine rig can be
introduced when blends, shake, or multiple camera states actually exist.

## Assets and scenes

Create the requested `_Game` subfolders, including missing `Art/Shaders`,
`Prefabs/Gameplay`, and `Scripts/Environment` folders. Create
`Assets/_Game/Settings/PlayerControls.inputactions` with a `Player` map and
`Move`, `Aim`, `Fire`, `Interact`, `Reload`, and `Dodge` actions. Only Move and
Aim are consumed by runtime code.

Build Settings order becomes:

1. `Assets/_Game/Scenes/Bootstrap.unity`
2. `Assets/_Game/Scenes/TestChamber.unity`
3. `Assets/OutdoorsScene.unity`

Bootstrap contains only a loader object. TestChamber contains a dark HDRP
development volume, directional and fill lighting, a large floor, outer
walls, obstacles, a broad room, a tighter passage, the Player prefab instance,
and the fixed camera. OutdoorsScene is not opened, edited, or loaded by the
normal flow.

The Player prefab hierarchy is:

```text
Player
├── Visual
│   └── Capsule
├── AimPivot
│   └── WeaponPlaceholder
├── FlashlightPivot
│   └── Flashlight
└── GroundCheck
```

The temporary aim marker is an additional disabled-by-default child controlled
by `PlayerAiming.showDebugMarker`, so it can be enabled without changing aim
logic.

## Project settings

Use the existing HDRP 17.5.0 asset, global settings, default volume profile,
and quality profiles. Do not change quality targets beyond development-safe
defaults already provided by HDRP. Set user layers 8 through 13 to Player,
Environment, Enemy, Projectile, Interactable, and CutawayGeometry. Keep the
cutaway layer physically collidable; only future rendering code may hide it.
Ignore only Player-to-Projectile and Projectile-to-Projectile collisions for
the current foundation. Leave environment interactions enabled.

The existing Input System setting is already active. The setup validates it
and does not add a second input configuration. The project’s existing package
versions are retained; no networking, procedural-generation, AI, inventory,
or random asset-store package is added.

## Verification

After authoring, wait for compilation and domain reload, confirm Pipeline
`editor_status` is ready, read the console error log, inspect Build Settings,
layers, graphics settings, open scene hierarchies, and capture the TestChamber
Game view at 1280x720. Run Unity batchmode validation against the same project
only when the live editor is not holding the project lock; otherwise use the
connected Pipeline commands. Treat new compiler, missing-script, package,
serialization, or shader errors as failures. Keep the known AI Assistant
network error separate because it predates this work.
