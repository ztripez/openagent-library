---
name: bevy-ui
description: Bevy 0.18 UI system - Node layout, styling, text, widgets, scrolling, and interaction
license: MIT
compatibility: opencode
metadata:
  audience: bevy-developers
  version: "0.18"
---

# Bevy 0.18 UI System

## Node Component

All UI elements use `Node` for layout (Style was merged into Node in 0.15):

```rust
commands.spawn(Node {
    width: Val::Px(200.),
    height: Val::Percent(100.),
    
    // Flexbox
    display: Display::Flex,
    flex_direction: FlexDirection::Column,
    justify_content: JustifyContent::Center,
    align_items: AlignItems::Center,
    flex_grow: 1.0,
    
    // Positioning
    position_type: PositionType::Absolute,
    left: Val::Px(10.),
    top: Val::Px(5.),
    
    // Spacing
    margin: UiRect::all(Val::Px(10.)),
    padding: UiRect::horizontal(Val::Px(5.)),
    
    ..default()
});
```

### Val Units
```rust
Val::Auto, Val::Px(100.), Val::Percent(50.)
Val::Vw(10.), Val::Vh(10.), Val::VMin(5.), Val::VMax(5.)
```

## Styling

```rust
commands.spawn((
    Node { ... },
    BackgroundColor(Color::srgb(0.2, 0.2, 0.8)),
    BorderColor(Color::WHITE),
    BorderRadius::all(Val::Px(10.)),
    BoxShadow {  // 0.15+
        color: Color::srgba(0., 0., 0., 0.5),
        x_offset: Val::Px(5.), y_offset: Val::Px(5.),
        blur_radius: Val::Px(10.), spread_radius: Val::Px(0.),
    },
));

// Gradients (0.17+)
commands.spawn((
    Node { ... },
    BackgroundGradient::linear(LinearGradientDirection::ToBottom, vec![
        GradientColorStop { color: Color::RED, position: Val::Percent(0.) },
        GradientColorStop { color: Color::BLUE, position: Val::Percent(100.) },
    ]),
));
```

## Text

```rust
commands.spawn((
    Text::new("Hello!"),
    TextFont { font_size: 24.0, ..default() },
    TextColor(Color::WHITE),
    TextLayout { justify: JustifyText::Center, ..default() },
));

// Rich text with spans
commands.spawn(Text::default())
    .with_child((TextSpan::new("Hello "), TextColor(RED.into())))
    .with_child((TextSpan::new("World!"), TextColor(BLUE.into())));

// Text effects (0.17-0.18+)
commands.spawn((Text::new("Shadow"), TextShadow { color: Color::BLACK, offset: Vec2::new(2., 2.) }));
commands.spawn((Text::new("Struck"), Strikethrough, StrikethroughColor(Color::RED)));
commands.spawn((Text::new("Under"), Underline));
commands.spawn((Text::new("Bold"), TextFont { weight: FontWeight::BOLD, ..default() }));
```

## Images

```rust
commands.spawn((
    ImageNode { image: assets.load("icon.png"), ..default() },
    Node { width: Val::Px(64.), height: Val::Px(64.), ..default() },
));

// 9-slice
commands.spawn((
    ImageNode {
        image: assets.load("panel.png"),
        image_mode: NodeImageMode::Sliced(TextureSlicer {
            border: BorderRect::square(16.), ..default()
        }),
        ..default()
    },
    Node { ... },
));
```

## Buttons & Interaction

```rust
commands.spawn((Button, Node { ... }, BackgroundColor(Color::GRAY)))
    .with_child(Text::new("Click"))
    .observe(|_: On<Pointer<Click>>| info!("Clicked!"));

// Query-based interaction
fn button_system(query: Query<(&Interaction, &mut BackgroundColor), Changed<Interaction>>) {
    for (interaction, mut bg) in &query {
        *bg = match *interaction {
            Interaction::Pressed => Color::srgb(0.5, 0.5, 0.5).into(),
            Interaction::Hovered => Color::srgb(0.4, 0.4, 0.4).into(),
            Interaction::None => Color::srgb(0.3, 0.3, 0.3).into(),
        };
    }
}
```

## Scrolling

```rust
commands.spawn((
    Node { overflow: Overflow::scroll(), ..default() },
    ScrollPosition::default(),
));

// Sticky headers (0.18+)
commands.spawn((Node { ... }, IgnoreScroll::default()));
```

## Headless Widgets (0.17+ experimental)

Enable `experimental_bevy_ui_widgets`:

```rust
// Slider
commands.spawn(Slider { value: 0.5, min: 0.0, max: 1.0, ..default() })
    .observe(|change: On<ValueChange<f32>>| info!("Value: {}", change.value));

// Checkbox
commands.spawn((Checkbox, Checked(false)))
    .observe(|change: On<ValueChange<bool>>| info!("Checked: {}", change.value));

// Radio buttons
let group = commands.spawn(RadioGroup::default()).id();
commands.spawn(RadioButton { group });

// Widget states: InteractionDisabled, Hovered, Checked, Pressed
```

## Focus & Navigation (0.17+)

```rust
commands.entity(widget).insert(Focused);

// Auto directional navigation (0.18+)
commands.spawn((Button, AutoDirectionalNavigation::default()));

fn nav(mut nav: AutoDirectionalNavigator) {
    nav.navigate(CompassOctant::East);
}
```

## ViewportNode (0.17+)

Render camera to UI:
```rust
commands.spawn(ViewportNode::new(camera_entity));
```

## Z-Index

```rust
commands.spawn((Node { ... }, ZIndex(10)));
commands.spawn((Node { ... }, GlobalZIndex(100)));
```
