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
