# 1. Why State Machines

In this book I want to describe a pattern I've started using whenever I write
almost any significant async Rust code. This pattern is inspired by state
machines,  but after awhile writing Rust, I've decided that it's almost always
distracting to try to think state-first, because it makes me want to enumerate
the states of the machine explicitly with enums, which has never worked out for
me. Instead, I want to write about how thinking _event_-first has made it
easier to build the systems I work on. Yes, this is essentially the same thing
as an Actor system, but implemented in plain async functions and probably
compatible with any async runtime, though I've only used it on Tokio.

As an added bonus, this pattern doesn't involve any macros, frameworks, or
magic type-trickery, it's just a particular way of writing async functions
that work together to solve problems. It just-so-happens that this pattern
_also_ leads to highly understandable, highly-maintainable, highly-observable,
and highly-composable code.

This pattern is so straightforward that I'm just going to start by showing a
general template I use, so you have almost a full idea of what I'm going to be
talking about from here on out. If we were building a task called the "Foo Bar"
task, I'd start out writing it like this:

```rust
// src/tasks/foo_bar.rs
pub struct FooBarContext {}
pub struct FoBarHandle {}
pub enum FooBarInput {}

pub async fn try_init_foo_bar_task(/* Resources */) -> Result<FooBarHandle> {}
pub fn try_spawn_foo_bar_task(/* Resources */) -> Result<FooBarHandle> {}
pub async fn foo_bar_loop(task_context: FooBarContext) {}
pub async fn try_foo_bar_stream(
    task_context: &FooBarContext,
) -> Result<impl Stream<Item = FooBarInput> {}
pub async fn try_foo_bar_loop(
    task_context: &FooBarContext,
    input_stream: &mut (impl Stream<Item = FooBarInput> + Unpin),
) {}
pub async fn try_foo_bar_event(
    task_context: &FooBarContext,
    input_event: FooBarInput,
) -> Result<()> {}
pub async fn try_foo_bar_event_a(task_context: &FooBarContext) {}
pub async fn try_foo_bar_event_b(task_context: &FooBarContext) {}

#[cfg(test)]
mod test {
    fn try_spawn_foo_bar_test() -> Result<()> {}
}
```

<!--

, and how this has led to a pattern of
writing code that is 

My state is just the typical data structures involved in the problem I'm
solving, so 

I like to imagine my task as a little worker at an assembly line,
responsible for solving one well-defined problem very very well.
, and I give it the tools and instructions
needed to solve that problem area

Instead, I focus on the _events_ of the
machine, thinking about all the ways in which 
