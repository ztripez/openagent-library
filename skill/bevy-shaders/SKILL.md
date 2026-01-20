---
name: bevy-shaders
description: Bevy shader bindings - AsBindGroup, uniforms, textures, storage buffers, and WGSL patterns
license: MIT
compatibility: opencode
metadata:
  audience: bevy-developers
  version: "0.15+"
---

# Bevy Shader Bindings

## Bind Group Layout

Bevy reserves groups 0-1 for internal use. Materials use **group 2** (via `#{MATERIAL_BIND_GROUP}` placeholder in WGSL or hardcoded `@group(2)`).

| Group | Purpose |
|-------|---------|
| 0 | View data (camera, globals) |
| 1 | Mesh data (transforms) |
| 2 | Material data (your bindings) |

Compute shaders use **group 0** since they don't share Bevy's render pipeline.

## AsBindGroup Derive (Rust Side)

```rust
#[derive(Asset, TypePath, AsBindGroup, Clone)]
pub struct MyMaterial {
    // Uniform buffer at binding 0
    #[uniform(0)]
    pub config: MyConfig,
    
    // Read-only storage buffer at binding 1
    #[storage(1, read_only)]
    pub data_buffer: Handle<ShaderStorageBuffer>,
    
    // Read-write storage buffer
    #[storage(2, read_write)]
    pub output_buffer: Handle<ShaderStorageBuffer>,
    
    // 2D texture at binding 3, sampler at binding 4
    #[texture(3)]
    #[sampler(4)]
    pub color_texture: Handle<Image>,
    
    // 2D array texture (cubemap faces, atlases)
    #[texture(5, dimension = "2d_array")]
    #[sampler(6)]
    pub array_texture: Handle<Image>,
    
    // Unsigned integer texture (index lookups)
    #[texture(7, sample_type = "u_int")]
    pub index_texture: Handle<Image>,
    
    // Cube texture
    #[texture(8, dimension = "cube")]
    #[sampler(9)]
    pub env_map: Handle<Image>,
}
```

### Attribute Reference

| Attribute | WGSL Type | Notes |
|-----------|-----------|-------|
| `#[uniform(N)]` | `var<uniform>` | For small, frequently-read data |
| `#[storage(N, read_only)]` | `var<storage, read>` | Large arrays, read-only |
| `#[storage(N, read_write)]` | `var<storage, read_write>` | Mutable GPU data |
| `#[texture(N)]` | `texture_2d<f32>` | Default 2D float texture |
| `#[texture(N, dimension = "2d_array")]` | `texture_2d_array<f32>` | Texture arrays |
| `#[texture(N, dimension = "cube")]` | `texture_cube<f32>` | Cubemaps |
| `#[texture(N, sample_type = "u_int")]` | `texture_2d<u32>` | Integer textures |
| `#[texture(N, sample_type = "f_int")]` | `texture_2d<i32>` | Signed int textures |
| `#[sampler(N)]` | `sampler` | Usually follows texture |

## WGSL Side Declarations

### Material Shader (Group 2)
```wgsl
// Use placeholder for material group
@group(#{MATERIAL_BIND_GROUP}) @binding(0)
var<uniform> config: MyConfig;

@group(#{MATERIAL_BIND_GROUP}) @binding(1)
var<storage, read> data: array<MyStruct>;

@group(#{MATERIAL_BIND_GROUP}) @binding(3)
var color_tex: texture_2d<f32>;

@group(#{MATERIAL_BIND_GROUP}) @binding(4)
var color_sampler: sampler;

@group(#{MATERIAL_BIND_GROUP}) @binding(5)
var array_tex: texture_2d_array<f32>;

@group(#{MATERIAL_BIND_GROUP}) @binding(7)
var index_tex: texture_2d<u32>;
```

### Compute Shader (Group 0)
```wgsl
@group(0) @binding(0)
var<uniform> params: ComputeParams;

@group(0) @binding(1)
var<storage, read> input: array<f32>;

@group(0) @binding(2)
var<storage, read_write> output: array<f32>;

// Storage texture for writing
@group(0) @binding(3)
var out_tex: texture_storage_2d_array<rgba16float, write>;
```

## Struct Alignment (Critical!)

Rust structs must match WGSL layout exactly. Use proper derives and padding:

```rust
#[derive(Clone, Copy, ShaderType, bytemuck::Pod, bytemuck::Zeroable)]
#[repr(C)]
pub struct MyConfig {
    pub value: f32,
    pub count: u32,
    pub _pad0: f32,  // Align next vec4
    pub _pad1: f32,
    pub color: Vec4,  // vec4 needs 16-byte alignment
    pub matrix: Mat4, // mat4 needs 16-byte alignment
}
```

```wgsl
struct MyConfig {
    value: f32,
    count: u32,
    _pad0: f32,
    _pad1: f32,
    color: vec4<f32>,
    matrix: mat4x4<f32>,
}
```

### Alignment Rules
| Type | Size | Alignment |
|------|------|-----------|
| `f32`, `u32`, `i32` | 4 | 4 |
| `vec2<f32>` | 8 | 8 |
| `vec3<f32>` | 12 | 16 (!) |
| `vec4<f32>` | 16 | 16 |
| `mat3x3<f32>` | 48 | 16 |
| `mat4x4<f32>` | 64 | 16 |
| `array<T>` | - | align of T |

**vec3 is tricky!** It has 16-byte alignment but only 12 bytes of data. Use vec4 or explicit padding.

## Material Implementation

```rust
impl Material for MyMaterial {
    fn fragment_shader() -> ShaderRef {
        "shaders/my_material.wgsl".into()
    }
    
    fn vertex_shader() -> ShaderRef {
        "shaders/my_material.wgsl".into()  // Optional custom vertex
    }
    
    fn alpha_mode(&self) -> AlphaMode {
        AlphaMode::Blend  // or Opaque, Mask, etc.
    }
    
    fn specialize(
        _pipeline: &MaterialPipeline<Self>,
        descriptor: &mut RenderPipelineDescriptor,
        _layout: &MeshVertexBufferLayoutRef,
        _key: MaterialPipelineKey<Self>,
    ) -> Result<(), SpecializedMeshPipelineError> {
        // Custom pipeline configuration
        descriptor.primitive.cull_mode = None;
        Ok(())
    }
}
```

## Creating Storage Buffers

```rust
fn setup(
    mut commands: Commands,
    mut buffers: ResMut<Assets<ShaderStorageBuffer>>,
    mut materials: ResMut<Assets<MyMaterial>>,
) {
    // Create data
    let data: Vec<MyStruct> = generate_data();
    
    // Create storage buffer
    let mut buffer = ShaderStorageBuffer::from(bytemuck::cast_slice(&data));
    buffer.buffer_description.usage |= BufferUsages::STORAGE;
    let buffer_handle = buffers.add(buffer);
    
    // Create material with buffer
    let material = materials.add(MyMaterial {
        data_buffer: buffer_handle,
        ..default()
    });
    
    commands.spawn((Mesh3d(mesh), MeshMaterial3d(material)));
}
```

## Texture Creation

```rust
fn create_texture(mut images: ResMut<Assets<Image>>) -> Handle<Image> {
    let size = Extent3d { width: 256, height: 256, depth_or_array_layers: 6 };
    
    let mut image = Image::new_fill(
        size,
        TextureDimension::D2,  // or D2Array for array textures
        &[0, 0, 0, 255],
        TextureFormat::Rgba8UnormSrgb,
        RenderAssetUsages::RENDER_WORLD,
    );
    
    // For storage textures (compute shader output)
    image.texture_descriptor.usage |= TextureUsages::STORAGE_BINDING;
    
    images.add(image)
}
```

## WGSL Texture Sampling

```wgsl
// Regular texture
let color = textureSample(color_tex, color_sampler, uv);

// Array texture (with layer index)
let color = textureSample(array_tex, array_sampler, uv, layer_index);

// Integer texture (no sampler, use textureLoad)
let index = textureLoad(index_tex, coords, 0).r;

// Storage texture write (compute shaders)
textureStore(out_tex, coords, layer, vec4<f32>(r, g, b, a));
```

## Dynamic Array Length

```wgsl
@group(2) @binding(0)
var<storage, read> items: array<Item>;

fn main() {
    let count = arrayLength(&items);
    for (var i = 0u; i < count; i++) {
        let item = items[i];
    }
}
```

## Common Patterns

### Vertex Pulling (Custom Vertex Data)
```rust
#[derive(AsBindGroup)]
pub struct InstancedMaterial {
    #[storage(0, read_only)]
    pub instances: Handle<ShaderStorageBuffer>,
}
```

```wgsl
@vertex
fn vertex(@builtin(instance_index) instance: u32, @builtin(vertex_index) vertex: u32) -> VertexOutput {
    let inst = instances[instance];
    // Use instance data for transform/color/etc
}
```

### Double Buffering (Animation Keyframes)
```rust
#[derive(AsBindGroup)]
pub struct AnimatedMaterial {
    #[storage(0, read_only)]
    pub prev_frame: Handle<ShaderStorageBuffer>,
    #[storage(1, read_only)]
    pub next_frame: Handle<ShaderStorageBuffer>,
    #[uniform(2)]
    pub blend_factor: f32,
}
```

### Baked Textures (Precomputed Data)
```rust
#[derive(AsBindGroup)]
pub struct BakedMaterial {
    #[texture(0, dimension = "2d_array")]
    #[sampler(1)]
    pub baked_cubemap: Handle<Image>,  // 6 layers for cube faces
}
```
