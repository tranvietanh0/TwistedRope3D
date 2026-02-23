# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UnityTwistedRope3D is a Unity hyper-casual game built with a modular architecture using VContainer for dependency injection. The project follows an MVP (Model-View-Presenter) pattern for UI and uses a state machine for game flow management.

## Build & Development

This is a Unity project. Open directly in Unity Editor (2022.3+ LTS recommended).

- **Solution**: `UnityTwistedRope3D.sln` - Open in Visual Studio or Rider
- **Main scenes**: `Assets/Scenes/0.LoadingScene.unity` (entry point), `Assets/Scenes/1.MainScene.unity`
- **Scenes are loaded via Addressables**, not direct scene references

## Architecture

### Dependency Injection (VContainer)

The project uses VContainer with a hierarchical scope system:
- `GameLifetimeScope` (root) - Registers core services via `RegisterGameFoundation()` and `RegisterUITemplate()`
- `SceneScope` instances per scene - Register scene-specific dependencies

Game states are auto-discovered and registered via reflection in `MainSceneScope`.

### Core Assemblies & Namespaces

| Assembly | Purpose |
|----------|---------|
| `HyperCasualGame.Scripts` | Game-specific code, state machines, screens |
| `GameFoundationCore.Scripts` | Core framework bootstrap |
| `GameFoundationCore.DI` | DI interfaces (`IInitializable`, `ITickable`, etc.) |
| `GameFoundationCore.Signals` | Event system (`SignalBus`) |
| `GameFoundationCore.UIModule` | Screen management, MVP base classes |
| `GameFoundationCore.BlueprintFlow` | Data blueprints and API handling |
| `UITemplate.Scripts` | UI templates and user data management |

### UI System (MVP Pattern)

Screens follow a Presenter-View pattern:
- **View** (`BaseView`): MonoBehaviour attached to UI prefabs
- **Presenter** (`BaseScreenPresenter<TView>`): Logic, injected via VContainer
- **ScreenInfo attribute**: Links presenter to its view - `[ScreenInfo(nameof(ViewClassName))]`

Screen initialization in scene scopes:
```csharp
builder.InitScreenManually<LoadingScreenPresenter>(autoBindData: true);
```

### State Machine

Game flow is managed by `GameStateMachine`:
- States implement `IGameState`
- States with `IHaveStateMachine` get the state machine reference injected
- Transition via `stateMachine.TransitionTo<TargetState>()`

### Signal Bus

Event-driven communication using `SignalBus`:
```csharp
// Declare in VContainer registration
builder.DeclareSignal<MySignal>();

// Subscribe
signalBus.Subscribe<MySignal>(handler);

// Fire
signalBus.Fire(new MySignal());
```

### Assets & Addressables

- `IGameAssets` interface for loading addressables
- Scenes loaded via `gameAssets.LoadSceneAsync("SceneName")`
- Assets organized in Addressables groups

## Key Dependencies

- **VContainer** - Dependency injection
- **UniTask** - Async/await for Unity
- **Addressables** - Asset management
- **DOTween/DOTweenPro** - Animation tweening
- **OSA (Optimized ScrollView Adapter)** - Virtualized lists
- **Odin Inspector** - Editor enhancements (requires license)

## Submodules (Assets/Submodules/)

Shared framework modules used across projects:
- **GameFoundationCore** - DI, Signals, UI framework, Blueprints, Utilities
- **UITemplate** - UI templates, local data persistence
- **Extensions** - Utility extension methods
- **Logging** - Logging infrastructure