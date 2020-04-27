# Otway

### GUI toolkit library which aims to continue the simplicity of Reclutch

## Counter Example

```rust
type CounterState = i32;

struct IncrementEvent(i32);
struct DecrementEvent(i32);

fn counter<T: 'static>(parent: ui::CommonRef, aux: &mut ui::Aux<T>) -> view::View<T, CounterState, CounterEvent> {
    let mut view = view::View::new(parent, /* CounterState: */ 0);

    let layout = view.vstack(aux);

    // powerful mix-in functions allow for elegant widget construction.
    let incr = view.lay_button_ext("Increment", layout, None, aux)
        .press(|view, aux, _event| {
            // `set_state` forwards the return type.
            aux.queue.emit_owned(IncrementEvent(view.set_state(|x| { *x += 1; x })));
        })
        .inner();

    let decr = view.lay_button_ext("Decrement", layout, None, aux)
        .press(|view, aux, _event| {
            aux.queue.emit_owned(DecrementEvent(view.set_state(|x| { *x -= 1; x })));
        })
        .inner();

    let label = view.lay_label_ext("", layout, None, aux)
        .size(42.0) // make the text bigger
        .inner();

    // Callback whenever `set_state` is invoked.
    view.state_changed(|view| {
        view.get(label).set_text(format!("Count: {}", view.state()));
    });

    // Invoke state_changed to initialize label.
    view.set_state(|_| {});

    view
}
```

### Event Queue Synchronization

Through much exploration, a conclusion was reached wherein some global object is required to synchronize event queues. This idea was simplified further into a global heterogenous queue.
The implementation used is `reclutch-nursery/uniq`, which is a heterogenous adapter on top of `reclutch/event`.

Given that there is a single queue, out-of-order events are impossible. Further, a thread-safe variant has been implemented, which can be used for multi-threaded UI applications.

### Parallelism

At some point, Otway may internally move from `sinq` to [`revenq`](https://github.com/YZITE/revenq), or parallel queue updating may be implemented in `sinq`.
Either way, hopefully there will be some multi-threading introduced to the update mechanisms.

There are no plans for moving rendering code to a separate thread, given that `winit` schedules repaints excellently already.

### `View` or `Widget`?

If you need custom rendering or custom input handling, use `Widget`.

If you want to compose widgets to make a larger UI, use `View`.

### `Widget`s have no Middleman

`View`s, by their very nature, simplify creating a UI by acting as a proxy interface, and thus require handles to reference children.

`Widget`s, on the other hand, handle everything themselves. They can access their children directly.

### Full and Extensible Theming

Widgets are 100% rendered by themes. Further than that, themes are stringly-typed and composable, meaning you can extend an existing theme to also cover your own custom widgets.

### Open-ended Windowing and Rendering

The only standard interface relating to OS interactions is the window event type, which defines events for things such as clicking, typing, cursor movements, etc.

Everything else is up to the implementor; any windowing API can be used and any graphics backend can be used as long as it implements `reclutch::display::GraphicsDisplay`.

## License

Otway is licensed under either

- [Apache 2.0](https://www.apache.org/licenses/LICENSE-2.0)
- [MIT](https://opensource.org/licenses/MIT)

at your choosing.
