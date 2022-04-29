---
title: 'Writing a Wayland wallpaper daemon in Rust'
date: "2021-10-15T13:30:00"
categories: ["workshop"]
tags: ["supervision", "c"]
---
class: center, middle
name: start

# Writing a Wayland </br> wallpaper daemon </br> in Rust ![:i](fab fa-rust)

.footer[29th April 2022</br>_Friday Talks_ @ ![:i](fab fa-suse)]
---
class: left, middle

# Danilo Spinella

### _Software Engineer in Packaging_

### **SUSE** ![:i](fab fa-suse)

---
class: middle, left

## ![:i](fas fa-list) Contents

Introduction to Wayland

The Wayland protocol

Implementation

---
class: left, middle

# ![:i](fas fa-chevron-right) Introduction to Wayland

---
class: left, middle

## What is Wayland ![:i](fas fa-question)

---
class: left, middle

## Wayland is...

...a communication protocol that specifies the communication between a display server and its clients

---
class: left, middle

## Wayland

Initial release in 2008

Aim to be X11 replacement

Designed to be secure and efficient

---
class: center, middle

![](https://wayland.freedesktop.org/x-architecture.png)

---
class: center, middle

![](https://wayland.freedesktop.org/wayland-architecture.png)

---
class: left, middle

The Wayland architecture integrates the display server, window manager and compositor into one process

---
class: left, middle

## What is wrong with X ![:i](fas fa-question) <sup>[^1](https://wayland.freedesktop.org/faq.html#heading_toc_j_6)</sup>

When you're an X server there's a tremendous amount of functionality that you must support to claim to speak the X protocol, yet nobody will ever use this

---
class: left, middle

# The Wayland protocol

---
class: left, middle

The Wayland protocol works by issuing *requests* and *events* that act on *objects*.

---
class: left, middle

Each object has an *interface* which defines what requests and events are possible, and the *signature* of each. 

---
class: left, middle

## Event / Request

Object ID

Message length

Request/event opcode

Arguments

---
class: left, middle

## Event example: wl_surface::event

```
0000000A    Object ID (10)
000C0000    Message length (12) and event opcode (0)
00000005    Output (object ID): 5
```

---
class: left, middle

## Components

wayland.xml

wayland-scanner

libwayland

---
class: left, middle

## `wl_surface`

```xml
<interface name="wl_surface" version="4">
  <request name="damage">
    <arg name="x" type="int" />
    <arg name="y" type="int" />
    <arg name="width" type="int" />
    <arg name="height" type="int" />
  </request>

  <event name="enter">
    <arg name="output" type="object" interface="wl_output" />
  </event>
</interface>
```

---
class: left, middle

## Protocol design patterns

Atomicity

Resource lifetimes

Versioning

---
class: left, middle

The protocol works over *named unix socket*, because they allow passing file descriptor

---
class: left, middle

# Implementation

---
class: left, middle

## Why Rust ![:i](fas fa-question)

*Insert generic Rust motives*

It has its own libwayland implementation

---
class: left, middle

## Crates

wayland-client

smithay-client-toolkit

image

---
class: left, middle

## Wayland display

Created when the connection is established

Object ID 1

---
class: left, middle

## Globals

*Globals* objects provide information, functionality and work as broker for additional objects

---
class: left, middle

## Registry

Bind the *globals* to the client

The server will emit the *global* event for each global available on the server

---
class: left, middle

## `wl_compositor`

Allow the creation of surfaces and regions

---
class: left, middle

## Wayland surface

Wayland *surface* has a rectangular area which may be displayed on zero or more outputs, present
buffers, receive user input, and define a local coordinate system.

---
class: left, middle

## Shared memory buffers

Allows you to transfer a file descriptor for the compositor to mmap with MAP_SHARED, then share pixel buffers out of this pool

---
class: left, middle

## `wlr-layer-shell`

Assign a surface to a "layer" of the output and render it with a defined z-depth respective to each other

---
class: left, middle

## XDG output

Describe outputs and their information

---
class: left, middle

An **event loop** waits for events and dispatches requests

---
class: left, middle

# Disclaimer ![:i](fas fa-exclamation)

This is not all code, but the most important part for showing as it works

---
class: left, middle

Holding the globals:

```rust
struct Env {
    compositor: SimpleGlobal<WlCompositor>,
    outputs: OutputHandler,
    shm: ShmHandler,
    xdg_output: XdgOutputHandler,
    layer_shell: SimpleGlobal<zwlr_layer_shell_v1::ZwlrLayerShellV1>,
}
```
---
class: left, middle

Information about a surface:

```rust
struct Surface {
    surface: wl_surface::WlSurface,
    layer_surface: Main<zwlr_layer_surface_v1::ZwlrLayerSurfaceV1>,
    pool: AutoMemPool,
    buffer: Option<wl_buffer::WlBuffer>,
    dimensions: (u32, u32),
}
```

---
class: left, middle

```rust
// Connect to the wayland server
// Creates the wayland display with object ID 1
let display = Display::connect_to_env().unwrap();
let mut queue = display.create_event_queue();
```

---
class: left, middle

```rust
let (outputs, xdg_output) =
    smithay_client_toolkit::output::XdgOutputHandler::new_output_handlers();

let env = Environment::new(
    &display.attach(queue.token()),
    &mut queue,
    Env {
        compositor: SimpleGlobal::new(),
        outputs,
        shm: ShmHandler::new(),
        xdg_output,
        layer_shell: SimpleGlobal::new(),
    }
);
```
---
class: left, middle

```rust
let layer_shell = env
        .require_global::<zwlr_layer_shell_v1::ZwlrLayerShellV1>();
let mut surfaces = Vec::new();
for output in env.get_all_outputs() {
    let surface = env.create_surface().detach();
    let pool = env.create_auto_pool().unwrap();
    surfaces.push(Surface::new(&output, surface, &layer_shell, pool));
}
```

---
class: left, middle

```rust
impl Surface {
    pub fn new(
        wl_output: &wl_output::WlOutput,
        surface: wl_surface::WlSurface,
        layer_shell: &Attached<zwlr_layer_shell_v1::ZwlrLayerShellV1>,
        pool: AutoMemPool,
    ) -> Self {
        let layer_surface = layer_shell.get_layer_surface(
            &surface,
            Some(wl_output),
            zwlr_layer_shell_v1::Layer::Background,
            "wallpaper".to_owned(),
        );
```

---
class: left, middle

```rust
        // Set proprieties of the layer_shell
        layer_surface.set_size(0, 0);
        layer_surface.set_anchor(
            zwlr_layer_surface_v1::Anchor::Top
                | zwlr_layer_surface_v1::Anchor::Left
                | zwlr_layer_surface_v1::Anchor::Right
                | zwlr_layer_surface_v1::Anchor::Bottom,
        );
        // If set to -1, the surface indicates that it would not like to be
        // moved to accommodate for other surfaces, and the compositor
        // should extend it all the way to the edges it is anchored to.
        layer_surface.set_exclusive_zone(-1);

        // Handle layer_shell events
        // Removed for simplicity
```

---
class: left, middle

```rust
        // Commit so that the server adds our layer_shell to the surface
        surface.commit();

        Self {
            surface,
            layer_surface,
            next_render_event,
            info,
            pool,
            buffer: None,
        }
    }
}
```

---
class: left, middle

```rust
// in the main
loop {
    for surface in surfaces {
        surfaces.draw();
    }
    
    queue.dispatch((), |_| {});
}
```

---
class: left, middle

```rust
impl Surface {
    pub fn draw(&mut self) {
        let stride = 4 * self.dimensions.0;
        let width = self.dimensions.0;
        let height = self.dimensions.1;

        let image = image::open("/path/to/wallpaper").unwrap();
                .resize_to_fill(width, height, FilterType::Lanczos3)
                .into_rgba8();
```

---
class: left, middle

`rgb(red, green, blue)`

`rgba(red, green, blue, alpha)`

`bgr(blue, green, red)`

`bgra(blue, green, red, alpha)`

---
class: left, middle

```rust
        self.buffer = Some(self.pool.try_draw::<_, _>(
            width,
            height,
            stride,
            wl_shm::Format::Abgr8888,
            |canvas: &mut [u8]| {
                let mut writer = BufWriter::new(canvas);
                writer
                    .write_all(image.as_raw())?
                writer.flush()?;

                Ok(())
            },
        ).unwrap()));
```

---
class: left, middle

Damaging a surface indicate to the compositor that it needs to be redrawn

---
class: left, middle

```rust
        // Attach the buffer to the surface
        self.surface
            .attach(Some(self.buffer.as_ref().unwrap()), 0, 0);

        // Mark the entire surface as damaged
        self.surface
            .damage_buffer(0, 0, width as i32, height as i32);

        // Finally, commit the surface
        self.surface.commit();
    }
}
```

---
class: middle, left

## Useful resources

[Wayland official page](https://wayland.freedesktop.org)

[Wayland book](https://wayland-book.com)

[Wayland protocols explorer](https://wayland.app/protocols/)

[wlroots implementation](https://gitlab.freedesktop.org/wlroots/wlroots)

---
class: center, middle

# Questions ![:i](fas fa-question)

---
class: center, middle

#![:i](fab fa-creative-commons) ![:i](fab fa-creative-commons-by) ![:i](fab fa-creative-commons-sa)

## This work is licensed under

## _Attribution-ShareAlike 4.0 International</br> (CC BY-SA 4.0)_

.footer[Icons from _www.onlinewebfonts.com_, which are licensed under CC BY 3.0]

---
template: start

