---
name: bevy-rendering
description: Bevy 0.18 rendering - cameras, lights, materials, atmosphere, GPU-driven rendering, and post-processing
license: MIT
compatibility: opencode
metadata:
  audience: bevy-developers
  version: "0.18"
---

# Bevy 0.18 Rendering

## Cameras

```rust
commands.spawn(Camera2d);

commands.spawn((
    Camera3d::default(),
    Camera { hdr: true, ..default() },
    Transform::from_xyz(0., 5., 10.).looking_at(Vec3::ZERO, Vec3::Y),
));

// Camera features auto-require dependencies
commands.spawn((
    Camera3d::default(),
    MotionBlur,  // Requires DepthPrepass, MotionVectorPrepass
    ScreenSpaceAmbientOcclusion::default(),
));
```

### Camera Controllers (0.18+)
```rust
app.add_plugins(FreeCameraPlugin);
commands.spawn((Camera3d::default(), FreeCamera::default()));

app.add_plugins(PanCameraPlugin);
commands.spawn((Camera2d::default(), PanCamera::default()));
```

## Lights

```rust
commands.spawn(PointLight { intensity: 1000.0, range: 50.0, ..default() });
commands.spawn(SpotLight { intensity: 5000.0, outer_angle: 0.8, ..default() });
commands.spawn(DirectionalLight::default());

// Volumetric fog lights (0.15+)
commands.spawn((PointLight { ... }, VolumetricLight));
```

## Meshes

```rust
// 3D
commands.spawn((Mesh3d(mesh_handle), MeshMaterial3d(material_handle)));

// 2D
commands.spawn((Mesh2d(mesh_handle), MeshMaterial2d(material_handle)));

// Virtual geometry (experimental)
commands.spawn((MeshletMesh3d(mesh), MeshMaterial3d(material)));
```

## Sprites

```rust
commands.spawn(Sprite {
    image: assets.load("player.png"),
    texture_atlas: Some(TextureAtlas { layout: atlas_layout, index: 0 }),
    image_mode: SpriteImageMode::Sliced(slicer),  // 9-slice
    ..default()
});
```

## Materials

```rust
// Standard PBR
let material = materials.add(StandardMaterial {
    base_color: Color::srgb(0.8, 0.2, 0.2),
    metallic: 0.5,
    perceptual_roughness: 0.3,
    ..default()
});

// GPU-driven rendering (0.16+) - for custom materials
#[derive(AsBindGroup)]
#[bindless]
struct MyMaterial { ... }
```

## Atmosphere (0.16+)

```rust
commands.spawn((Camera3d::default(), Atmosphere::EARTH));

// Raymarched (0.17+)
commands.spawn((
    Camera3d::default(),
    Atmosphere::default(),
    AtmosphereSettings { rendering_method: AtmosphereMode::Raymarched, ..default() },
));

// Custom atmosphere (0.18+)
let medium = media.add(ScatteringMedium::new(256, 256, [
    ScatteringTerm { absorption: Vec3::ZERO, scattering: Vec3::new(5.8e-6, 13.5e-6, 33.1e-6), ... },
]));
commands.spawn((Camera3d::default(), Atmosphere::earthlike(medium)));
```

## Fog

```rust
// Distance fog
commands.spawn((Camera3d::default(), DistanceFog { color: Color::WHITE, falloff: FogFalloff::Linear { start: 10.0, end: 100.0 }, ..default() }));

// Volumetric fog (0.15+)
commands.spawn((Camera3d::default(), VolumetricFog { ambient_intensity: 0.1, ..default() }));
commands.spawn(FogVolume { density_factor: 0.2, ..default() });
```

## Post-Processing

```rust
// Built-in effects
commands.spawn((Camera3d::default(), Bloom::default(), ChromaticAberration::default()));

// Fullscreen materials (0.18+)
impl FullscreenMaterial for MyEffect {
    fn fragment_shader() -> ShaderRef { "effect.wgsl".into() }
    fn node_edges() -> Vec<InternedRenderLabel> {
        vec![Node3d::Tonemapping.intern(), Self::node_label().intern(), Node3d::EndMainPassPostProcessing.intern()]
    }
}
```

## Decals (0.16+)

```rust
commands.spawn((ForwardDecal, ForwardDecalMaterial { ... }));
commands.spawn(ClusteredDecal { image: decal_texture, ..default() });
```

## Picking (0.15+)

```rust
commands.spawn(Sprite::from_image(image))
    .observe(|_: On<Pointer<Click>>| info!("Clicked!"))
    .observe(|drag: On<Pointer<Drag>>, mut q: Query<&mut Transform>| {
        q.get_mut(drag.entity()).unwrap().translation.x += drag.delta.x;
    });

// Mesh picking requires MeshPickingPlugin
```

## Solari Raytracing (0.17+ experimental)

```rust
// Reference pathtracer
cargo run --example solari --features bevy_solari -- --pathtracer

// Realtime with DLSS
cargo run --example solari --features bevy_solari,dlss
```

## Screenshots (0.18+)

```rust
app.add_plugins(EasyScreenshotPlugin::default());  // PrintScreen key
app.add_plugins(EasyScreenRecordPlugin::default());  // Space to toggle recording
```
