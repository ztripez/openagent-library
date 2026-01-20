---
name: bevy-0-18
description: Bevy 0.18 migration guide - key breaking changes and new features from 0.15 to 0.18
license: MIT
compatibility: opencode
metadata:
  audience: bevy-developers
  version: "0.18"
---

# Bevy 0.18 Migration Guide

Quick reference for breaking changes and new features from Bevy 0.15 to 0.18. For detailed patterns, see: `bevy-ecs`, `bevy-ui`, `bevy-rendering`, `bevy-components`.

## Breaking Changes Summary

### Bundles Deprecated (0.15)
Use Required Components instead:
```rust
// Old: #[derive(Bundle)] struct PlayerBundle { ... }
// New:
#[derive(Component)]
#[require(Sprite, Team)]
struct Player;
commands.spawn(Player);
```

### Style Merged into Node (0.15)
```rust
// Old: commands.spawn((Node::default(), Style { width: Val::Px(100.) }));
// New:
commands.spawn(Node { width: Val::Px(100.), ..default() });
```

### Handle<T> Not a Component (0.15)
```rust
// Old: commands.spawn(mesh_handle);
// New:
commands.spawn(Mesh3d(mesh_handle));
commands.spawn(Sprite { image: image_handle, ..default() });
commands.spawn(AudioPlayer(audio_handle));
commands.spawn(SceneRoot(scene_handle));
```

### Observer API Changes (0.17)
```rust
// Old: world.add_observer(|trigger: Trigger<MyEvent>| { ... });
// New:
world.add_observer(|event: On<MyEvent>| { ... });

// Old: OnAdd, OnRemove, OnInsert
// New: Add, Remove, Insert
app.add_observer(|add: On<Add, Player>| { ... });
```

### Event vs Message (0.17)
```rust
// Events are observed
#[derive(Event)]
struct GameOver;

// Messages are buffered (formerly EventReader/EventWriter)
#[derive(Message)]
struct ChatMessage { text: String }
fn send(mut w: MessageWriter<ChatMessage>) { w.write(...); }
fn recv(mut r: MessageReader<ChatMessage>) { for m in r.read() { ... } }
```

### EntityEvent (0.17)
```rust
// Old: world.trigger_targets(MyEvent, entity);
// New:
#[derive(EntityEvent)]
struct Click { entity: Entity }
world.trigger(Click { entity });
```

### UiImage -> ImageNode (0.15)
```rust
// Old: commands.spawn(UiImage { ... });
// New:
commands.spawn(ImageNode { image: handle, ..default() });
```

## New Features by Version

### 0.15
- Required Components (`#[require(...)]`)
- Entity Picking (`bevy_picking`)
- Box Shadows, UI Scrolling
- Cosmic Text (better i18n)
- Volumetric Fog for Point/Spot lights
- Curves API, Easing Functions

### 0.16
- ECS Relationships (`#[relationship]`, `ChildOf`, `Children`)
- `children!` and `related!` macros
- GPU-Driven Rendering (auto-enabled)
- Procedural Atmosphere
- Decals (Forward and Clustered)
- Error Handling (`-> Result` in systems)
- Entity Disabling, Cloning, Immutable Components
- `no_std` support

### 0.17
- Observer API overhaul (`On<T>`, `EntityEvent`, `Message`)
- Reflection Auto-Registration
- Headless UI Widgets (experimental)
- Bevy Feathers (experimental)
- Solari Raytracing (experimental)
- Hot Patching Systems
- DLSS Support
- Tilemap Chunks
- ViewportNode
- UI Gradients, Text Shadows

### 0.18
- Atmosphere Occlusion and PBR Shading
- Generalized Scattering Media
- Fullscreen Materials
- Camera Controllers (FreeCamera, PanCamera)
- Cargo Feature Collections (`2d`, `3d`, `ui`)
- Font Weights, Strikethrough, Underline, OpenType Features
- Auto Directional Navigation
- Popover/Menu widgets
- Easy Screenshots/Recording

## Feature Flags (0.18)

```toml
# High-level feature collections
bevy = { version = "0.18", default-features = false, features = ["2d"] }
bevy = { version = "0.18", default-features = false, features = ["3d"] }
bevy = { version = "0.18", default-features = false, features = ["ui"] }
```

## Migration Resources

- [0.14 to 0.15 Guide](https://bevyengine.org/learn/migration-guides/0-14-to-0-15/)
- [0.15 to 0.16 Guide](https://bevyengine.org/learn/migration-guides/0-15-to-0-16/)
- [0.16 to 0.17 Guide](https://bevyengine.org/learn/migration-guides/0-16-to-0-17/)
- [0.17 to 0.18 Guide](https://bevyengine.org/learn/migration-guides/0-17-to-0-18/)
