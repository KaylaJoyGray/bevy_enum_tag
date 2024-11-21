# bevy_enum_tag

This crate provides a macro that makes it easier to filter for a specific variant of an enum component.

## How it works

This crate is similar to [bevy_enum_filter](https://github.com/MrGVSV/bevy_enum_filter), but doesn't rely on change detection or require you to register anything
with the Bevy app.

When you use the `derive_enum_tag macro` on an enum, marker structs are generated for every variant of the enum. The macro also adds 
[component hooks](https://github.com/bevyengine/bevy/pull/14005) that insert/remove the correct markers whenever the enum component is removed.
Because of this feature, you will have to use at least Bevy 0.15 (currently only available as a release candidate).

The marker structs are generated in their own module, which is named using the snake case version of the enum's name. See the examples below for clarification.

## Usage


You can use the `derive_enum_tag` macro on any enum:
```Rust
use bevy_enum_tag::derive_enum_tag;

#[derive_enum_tag]
enum EmptyEnum {}

#[derive_enum_tag]  // derives Component, so TestEnum is now a component
enum TestEnum {
    Variant1,
    Variant2,
    Variant3 {
        some_field: u32,
    }
}
```


The tag is automatically inserted when the enum component is added to an entity:
```Rust
fn spawn_test_enums(mut commands: Commands) {
    commands.spawn(TestEnum::Variant1);
    commands.spawn(TestEnum::Variant2);
}
```


Since the tag is just a marker component, you can use any QueryFilter:
```Rust
use test_enum::Variant1;
use test_enum::Variant2;
use test_enum::Variant3;

fn added_variant_1(query: Query<&TestEnum, Added<Variant1>>) {
    // variant 1 added
}

fn with_variant_2(query: Query<&TestEnum, With<Variant2>>) {
    // with variant 2
}

fn without_variant_3(query: Query<&TestEnum, Without<Variant3>>) {
    // without variant 3
}
```


Tags are automatically removed when their associated component is removed:
```Rust
use test_enum::Variant1;
use test_enum::Variant2;

fn remove_test_enum(mut commands: Commands, query: Query<Entity, With<TestEnum>>) {
    query.iter().for_each(|entity| {
        commands.entity(entity).remove::<TestEnum>();
    });
}

/// run after remove_test_enum
fn check_tags_removed(query1: Query<Entity, With<Variant1>>, query2: Query<Entity, With<Variant2>>) {
    assert!(query1.is_empty());
    assert!(query2.is_empty());
}
```