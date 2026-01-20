---
name: bevy-components
description: Bevy 0.18 component patterns - modern spawning for cameras, sprites, meshes, text, audio, and scenes
license: MIT
compatibility: opencode
metadata:
  audience: bevy-developers
  version: "0.18"
---

# Bevy 0.18 Component Patterns

Modern Bevy uses Required Components instead of Bundles. Components auto-insert their dependencies.

## Cameras

```rust
commands.spawn(Camera2d);

commands.spawn((
    Camera3d::default(),
    Camera { hdr: true, ..default() },
    Transform::from_xyz(0., 5., 10.).looking_at(Vec3::ZERO, Vec3::Y),
));
```

## Sprites

```rust
commands.spawn(Sprite {
    image: assets.load("player.png"),
    texture_atlas: Some(TextureAtlas { layout, index: 0 }),
    image_mode: SpriteImageMode::Sliced(slicer),
    ..default()
});
```

## Meshes

```rust
// 3D mesh
commands.spawn((Mesh3d(mesh_handle), MeshMaterial3d(material_handle)));

// 2D mesh
commands.spawn((Mesh2d(mesh_handle), MeshMaterial2d(material_handle)));
```

## Text

### World-space (Text2d)
```rust
commands.spawn((
    Text2d::new("Hello World"),
    TextFont { font_size: 48.0, ..default() },
    TextColor(Color::WHITE),
));
```

### UI Text
```rust
commands.spawn((
    Text::new("UI Text"),
    TextFont { font_size: 24.0, ..default() },
    TextColor(Color::WHITE),
    Node { ... },
));
```

### Rich Text
```rust
commands.spawn(Text::default())
    .with_child((TextSpan::new("Red "), TextColor(RED.into())))
    .with_child((TextSpan::new("Blue"), TextColor(BLUE.into())));
```

## UI Nodes

```rust
commands.spawn(Node {
    width: Val::Px(100.),
    height: Val::Percent(50.),
    ..default()
});

commands.spawn((Button, Node { ... }));

commands.spawn((
    ImageNode { image: assets.load("icon.png"), ..default() },
    Node { width: Val::Px(64.), ..default() },
));
```

## Lights

```rust
commands.spawn(PointLight { intensity: 1000.0, range: 50.0, ..default() });
commands.spawn(SpotLight { intensity: 5000.0, ..default() });
commands.spawn(DirectionalLight::default());
```

## Audio

```rust
commands.spawn(AudioPlayer(assets.load("music.mp3")));
commands.spawn((AudioPlayer(sound), PlaybackSettings::DESPAWN));
```

## Scenes

```rust
commands.spawn(SceneRoot(scene_handle));
commands.spawn(DynamicSceneRoot(dynamic_scene_handle));
```

## Transforms

Transform requires GlobalTransform. Most spatial components require Transform:

```rust
commands.spawn((
    Sprite { ... },
    Transform::from_xyz(100., 50., 0.).with_scale(Vec3::splat(2.)),
));
```

## Visibility

Visibility requires InheritedVisibility and ViewVisibility. Renderable components require Visibility:

```rust
commands.spawn((
    Sprite { ... },
    Visibility::Hidden,  // Override default visible
));
```

## Common Migrations

### Handle<T> no longer a component
```rust
// Old: commands.spawn(mesh_handle)
// New:
commands.spawn(Mesh3d(mesh_handle));
commands.spawn(Sprite { image: image_handle, ..default() });
commands.spawn(AudioPlayer(audio_handle));
commands.spawn(SceneRoot(scene_handle));
```

### Style merged into Node
```rust
// Old: commands.spawn((Node::default(), Style { width: Val::Px(100.), ... }));
// New:
commands.spawn(Node { width: Val::Px(100.), ..default() });
```

### UiImage renamed to ImageNode
```rust
// Old: commands.spawn(UiImage { ... });
// New:
commands.spawn(ImageNode { image: handle, ..default() });
```

### Camera bundles removed
```rust
// Old: commands.spawn(Camera2dBundle::default());
// New:
commands.spawn(Camera2d);
```
