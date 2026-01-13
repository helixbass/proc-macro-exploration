# Without proc-macros: hash-map from iter

If you dislike having to manually `.insert()` things into our `HashMap` and
`HashSet`, the most idiomatic way I am aware of to avoid that is to use
`FromIterator`/`.collect()`/`::from_iter()`

This is actually some cool Rust stuff that you may not have your head wrapped
around. Let's look at how these work and how to express these initializations
"inline"/"as a single expression" using these facilities

## `HashSet` initialization

Ok so the reference point here which is kind of ugly because it initializes the
`HashSet` via a sequence of statements rather than a single
expression[^expression] is:
```
let mut valid_children_presentational_modes: HashSet<PresentationalMode> = HashSet::new();
valid_children_presentational_modes.insert(PresentationalMode::Vertical);
valid_children_presentational_modes.insert(PresentationalMode::Circular);
```

[^expression]: If you need convincing that it is stylish coding to know how to
    build up things like function bodies using idioms that are more
    "expression-oriented" than "statement-oriented" I'd encourage you to read
    eg some things from "Lisp-world". Classic "functional programming" (for
    lack of a better name) style grasps the elegance of code that does things
    like nest silly depths of expressions inside each other for no apparent
    reason

## Oh wait let's talk about Rust blocks

Before proceeding to `.collect()` etc this makes me think of another Rust idiom
that I'm not convinced enough people are familiar with: YOU CAN MAKE ANY SET OF
RUST STATEMENTS INTO AN EXPRESSION BY ENCLOSING THEM IN A BLOCK

So I often will do things like group a set of statements "inline" inside a
completely structurally-unnecessary Rust block just because I find it
expressive and "expression-oriented"

So here we can instantiate `valid_children_presentational_modes` in technically
a single expression by wrapping its initialization in a Rust block:
```
{
    let mut valid_children_presentational_modes: HashSet<PresentationalMode> = HashSet::new();
    valid_children_presentational_modes.insert(PresentationalMode::Vertical);
    valid_children_presentational_modes.insert(PresentationalMode::Circular);
    valid_children_presentational_modes
}
```
Wow ok. Didn't know you could use Rust blocks that way

This relies on the gorgeous Rust feature (that maybe Coffeescript helped
inspire?) that the last expression in a block is implicitly "returned"

So now that block "expression" can be used in "expression position"
grammatically/syntactically eg:
```
PresentationalModeOptions {
    immediate: true,
    valid_children_presentational_modes: Some({
        let mut valid_children_presentational_modes: HashSet<PresentationalMode> = HashSet::new();
        valid_children_presentational_modes.insert(PresentationalMode::Vertical);
        valid_children_presentational_modes.insert(PresentationalMode::Circular);
        valid_children_presentational_modes
    }),
    last_usage_hints: None,
    height: None,
    max_height: None,
    width: None,
    max_width: None,
}
```
To a far less stylish coder than myself this may look ugly. Well you're wrong.
Don't under-value expression-oriented code style

For one thing now we have less "spread of local variable scope". Our
`valid_children_presentational_modes` variable is confined scope-wise to this
enclosing Rust block. If you are a proper programmer and your brain insists on
tracking all the execution-time semantics of a piece of code you're looking at,
you agree with people like the venerable Kent Beck etc who would lecture you
that in properly factored code you don't want a lot of local variable state
(which may or may not be mutable) floating around to keep track of the
possible changes/interplays of

NOTICE THAT WHEN YOU USE EXPRESSION-ORIENTED CODE STYLE YOU HAVE WAY LESS
LOCAL VARIABLES THAN IN STATEMENT-ORIENTED CODE STYLE

### A typical stylish case: `move`

A place where it really reads nicely to add a wholly unnecessary Rust block to
your code is surrounding a `move` closure or `async` block

So instead of writing eg this:
```
let (sender, receiver) = channel();
let contents = vec!["Hello", "World"];

do_some_other_stuff();

let sender = sender.clone();
let contents_ref = &contents;
do_something_with(async move {
    sender.send(contents_ref[0].to_lowercase()).await.unwrap();
});
```
you may often want to use an as-narrowly-placed-as-possible Rust block to
"segregate" the code related to the `move`:
```
let (sender, receiver) = channel();
let contents = vec!["Hello", "World"];

do_some_other_stuff();

do_something_with({
    let sender = sender.clone();
    let contents_ref = &contents;
    async move {
        sender.send(contents_ref[0].to_lowercase()).await.unwrap();
    }
});
```
I personally do this consistently because it "muddies up" less. The top-level
function body reads more easily and has less local-variable-state floating
around

## `::from_iter()`

One way of expressing `HashSet`/`HashMap` initialization in a single expression
is using eg `HashSet::from_iter()`. This method is from the
[`FromIterator`](https://doc.rust-lang.org/stable/std/iter/trait.FromIterator.html)
trait

As you can see from `HashSet`'s
[docs](https://doc.rust-lang.org/stable/std/collections/struct.HashSet.html#impl-FromIterator%3CT%3E-for-HashSet%3CT,+S%3E),
`HashSet::from_iter()` expects to be passed an `Iterator`/`IntoIterator` whose
`Item` type is the `HashSet` itself's item type (aka `T`)

So in our case we could say:
```
HashSet::from_iter([
    PresentationalMode::Vertical,
    PresentationalMode::Circular,
])
```
If you're not intimately familiar with `Iterator`/`IntoIterator`, the specific
mechanics of this line of code is:

The type of the argument passed to `HashSet::from_iter()` is I believe
`[PresentationalMode; 2]` (aka an array (rather than `Vec`) of
`PresentationalMode`'s with length 2)

Arrays implement `IntoIterator` over their item type. So clearly under the hood
`HashSet::from_iter()` is calling `.into_iter()` on this array and then
initializing itself as it consumes the resulting
`Iterator<Item = PresentationalMode>`'s items

Cool that is nicer I'd say

## `.collect()`

We've all seen `.collect()` being used to "accumulate" the results of an
iterator into a `Vec`

But I don't think everyone knows that `.collect()` is built on top of
`FromIterator`!

(So when we call `.collect()` to accumulate an iterator into a `Vec`, we can
safely surmise that `Vec<T>` implements `FromIterator` for
`Iterator`/`IntoIterator`'s whose `Item` type is `T`)

Ok mind-blowing insight into very standard `.collect()` Rust code if you didn't
already know that

Well so same principle at work, `HashSet`'s as we just saw above also implement
`FromIterator` for its item type

So in our case we can also create our `HashSet` this way:
```
[
    PresentationalMode::Vertical,
    PresentationalMode::Circular,
].collect()
```

Cute.

#### and `.collect()` for `HashMap`'s

`HashMap`'s also implement `FromIterator`. In their case they expect key-value
tuples (`(K, V)`) as the iterated item type (this is similar to how iterating
over a `HashMap` yields key-value tuples, no?)

So similarly to initialize our `HashMap` we can say:
```
[(
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
)].collect()
```

To dissect this if the syntax is a bit dense/overwhelming:

`[(a, b)]` is a single-item array whose item type is a two-item tuple

So our `[...]`'s type is `[(PresentationalMode, PresentationalModeOptions); 1]`

Ok so this array type implements `IntoIterator` with `Item = (PresentationalMode, PresentationalModeOptions)`

And in fact that `Item` type is the correct one to initialize a
`HashMap<PresentationalMode, PresentationalModeOptions>` via `.collect()`
(aka via `HashMap::from_iter()`/`FromIterator`)

Brilliant. We've made our initialization code prettier I'd say

At this point the whole thing would look like:
```
StylingOptions {
    colors: vec![
        Color::Red,
        Color::White,
        Color::Blue,
    ],
    presentational_modes: [(
        PresentationalMode::Vertical,
        PresentationalModeOptions {
            immediate: true,
            valid_children_presentational_modes: Some([
                PresentationalMode::Vertical,
                PresentationalMode::Circular,
            ].collect()),
            last_usage_hints: None,
            height: None,
            max_height: None,
            width: None,
            max_width: None,
        },
    )].collect(),
}
```

Next we can use typical `::new()` methods to arguably make our initialization
code more clean, idiomatic and perhaps flexible
