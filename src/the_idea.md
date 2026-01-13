# The Idea

Say that I have a brilliant idea for a shorthand/abbreviated syntax for
initializing some data structure that I commonly create instances of

Maybe I'm working on a library/app where we have a type like:
```
use std::{collections::{HashMap, HashSet}, hash::Hash};
use crossterm::style::Color;

struct StylingOptions {
    pub colors: Vec<Color>,
    pub presentational_modes: HashMap<PresentationalMode, PresentationalModeOptions>,
}

#[derive(Copy, Clone, Hash)]
enum PresentationalMode {
    Vertical,
    Horizontal,
    Circular,
}

struct PresentationalModeOptions {
    pub immediate: bool,
    pub valid_children_presentational_modes: Option<HashSet<PresentationalMode>>,
    pub last_usage_hints: Option<Vec<String>>,
    pub height: Option<usize>,
    pub max_height: Option<usize>,
    pub width: Option<usize>,
    pub max_width: Option<usize>,
}
```

And I am tiring of the most elegant/succinct ways I can come up with to express
different instantiations of `StylingOptions` in actual Rust code

So I dream of being able to express myself using syntax like:
```
styling_options!(
    colors => [red, white, blue],
    presentational_modes => [
        vertical => [
            children => [vertical, circular]
        ],
    ],
)
```

Personally I don't shy away from such non-Rust-esque syntax inside a
proc-macro invocation. But feel free to disagree. This book in theory can
educate you about how to achieve such wild syntactic visions while holding
different tastes about how they "sound"

So we'll go with this as an excuse to dissect how (in my limited but
hey-I'm-confident-enough-about-this-to-write-a-book-about-it knowledge) to
turn this dream into a reality

Proc-macros are evil and rad. Don't avoid or fear them. We use them constantly.
Eg unless I'm completely confused such common Rust-ism's as
`#[derive(Debug, PartialEq, Eq)]` are disgusting wanton syntax invoking
horribly-expensive-at-compile-time-supposedly proc-macro's under the hood

And hey you know what's shocking? We all love them because they are so
syntactically succinct, descriptive and declarative. God bless

So with that vision in mind, we'll work towards it by first examining my
understanding of a couple different ways to achieve instantiations of
`StylingOptions` in "plain old Rust". And then try and completely blow them
out of the water with proc-macro gorgeousness
