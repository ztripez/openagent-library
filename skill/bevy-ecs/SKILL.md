---
name: bevy-ecs
description: Bevy 0.18 ECS patterns - Required Components, Relationships, Observers, Events, and entity management
license: MIT
compatibility: opencode
metadata:
  audience: bevy-developers
  version: "0.18"
---

# Bevy 0.18 ECS Patterns

## Required Components (0.15+)

Required Components replaced Bundles as the idiomatic way to spawn entities.

```rust
// DON'T: Bundles are deprecated
#[derive(Bundle)]
struct PlayerBundle { player: Player, sprite: Sprite, ... }

// DO: Use Required Components
#[derive(Component, Default)]
#[require(Team, Sprite)]  // Sprite requires Transform, Visibility, etc.
struct Player { name: String }

// Simple spawn - required components auto-inserted
commands.spawn(Player::default());

// Override specific required components
commands.spawn((Player { name: "Bob".into() }, Team::Blue));
```

### Custom Defaults
```rust
#[derive(Component)]
#[require(Team(|| Team::Blue))]  // Closure
struct Player;

#[derive(Component)]
#[require(Team(blue_team))]  // Function
struct Player;
fn blue_team() -> Team { Team::Blue }
```

## ECS Relationships (0.16+)

Bidirectional entity linking with automatic cleanup.

```rust
#[derive(Component)]
#[relationship(relationship_target = LikedBy)]
struct Likes(pub Entity);

#[derive(Component, Deref)]
#[relationship_target(relationship = Likes)]
struct LikedBy(Vec<Entity>);
```

### Parent-Child Hierarchy
```rust
commands.spawn(ChildOf(parent_entity));  // Add child
commands.entity(child).insert(ChildOf(new_parent));  // Reparent
```

### Spawning Hierarchies
```rust
commands.spawn((
    Player,
    children![
        (RightHand, children![Glove, Sword]),
        (LeftHand, children![Glove, Shield]),
    ],
));

// Custom relationships
commands.spawn((
    Name::new("Monica"),
    related!(LikedBy[Name::new("Naomi"), Name::new("Dwight")]),
));
```

## Observers and Events (0.17+)

### Event Types
```rust
#[derive(Event)]
struct GameOver { score: u32 }  // Global event

#[derive(EntityEvent)]
struct Click { entity: Entity }  // Entity event

#[derive(EntityEvent)]
#[entity_event(propagate)]
struct UiClick { entity: Entity }  // Bubbles up hierarchy
```

### Observer Syntax
```rust
world.add_observer(|game_over: On<GameOver>| {
    info!("Score: {}", game_over.score);
});

commands.entity(button).observe(|click: On<Click>| { ... });

// Propagation control
world.add_observer(|mut click: On<UiClick>| {
    click.propagate(false);
});
```

### Component Lifecycle
```rust
app.add_observer(|add: On<Add, Player>| {
    info!("Player {} added", add.entity);
});
```

### Messages (buffered, not observed)
```rust
#[derive(Message)]
struct ChatMessage { text: String }

fn send(mut writer: MessageWriter<ChatMessage>) {
    writer.write(ChatMessage { text: "Hello".into() });
}

fn receive(mut reader: MessageReader<ChatMessage>) {
    for msg in reader.read() { info!("{}", msg.text); }
}
```

## Error Handling (0.16+)

```rust
fn move_player(mut query: Query<&mut Transform, With<Player>>) -> Result {
    let mut transform = query.single_mut()?;
    transform.translation.x += 1.0;
    Ok(())
}
```

## Entity Management

### Disabling (0.16+)
```rust
commands.entity(e).insert(Disabled);  // Excluded from default queries
fn system(query: Query<&Transform, With<Disabled>>) { ... }
```

### Cloning (0.16+)
```rust
let clone = commands.entity(original).clone();
let clone = commands.entity(original).clone_with_hierarchy();
```

### Immutable Components (0.16+)
```rust
#[derive(Component)]
#[component(immutable)]
struct Id(u64);  // Only removed/replaced, never mutated
```

## Reflection (0.17+)

```rust
#[derive(Reflect)]
struct MyData { ... }  // Auto-registered, no app.register_type needed

app.register_type::<Container<Item>>()  // Generics need manual registration

#[derive(Reflect)]
#[reflect(no_auto_register)]
struct ManualOnly { ... }
```
