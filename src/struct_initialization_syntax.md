# Without proc-macros: struct initialization syntax

The most already-included-with-Rust way to instantiate our example dream
`StylingOptions` syntax is:
```
let mut presentational_modes: HashMap<PresentationalMode, PresentationalModeOptions> =
    HashMap::new();
let mut valid_children_presentational_modes: HashSet<PresentationalMode> = HashSet::new();
valid_children_presentational_modes.insert(PresentationalMode::Vertical);
valid_children_presentational_modes.insert(PresentationalMode::Circular);
presentational_modes.insert(
    PresentationalMode::Vertical,
    PresentationalModeOptions {
        immediate: true,
        valid_children_presentational_modes: Some(valid_children_presentational_modes),
        last_usage_hints: None,
        height: None,
        max_height: None,
        width: None,
        max_width: None,
    }
);
StylingOptions {
    colors: vec![
        Color::Red,
        Color::White,
        Color::Blue,
    ],
    presentational_modes,
}
```

I would call this "struct-initialization" syntax. Where you just specify all
of the fields in the desired struct instance directly
