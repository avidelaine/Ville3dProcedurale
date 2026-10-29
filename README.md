# Procedural 3D city — Rust / Bevy

A procedural city generator written in Rust with the [Bevy](https://bevyengine.org/) engine. Streets, parks and buildings are laid out from Perlin noise, generated and unloaded by chunks around the camera, and driven in real time by an egui panel: a *chaos* slider reshapes the city, a *resolution* slider trades render quality for frame rate.

Built in spring 2025 as a Rust course project at ESIEE Paris, with Alexandre Grosdidier and Théo Lyonnet.

![Overview](./screenshots/vue_generale.png)

## Run it

```bash
rustup install stable
git clone https://github.com/avidelaine/Ville3dProcedurale && cd Ville3dProcedurale
cargo run --release
```

Arrow keys pan the camera; the egui panel exposes chaos, zoom, resolution and a *regenerate* button.

## How it works

| Module | Role |
|---|---|
| `city.rs` | Tile generation: a Perlin field decides, per cell, whether it becomes road, park or building; the chaos value scales the noise so the city goes from a regular grid to an irregular sprawl |
| `chunk.rs` | Keeps only the chunks around the camera alive; spawns new ones as the camera moves and despawns the far ones. A change of chaos (detected with `is_changed()`) clears and regenerates everything |
| `camera.rs` | 3D camera with pan and zoom (height and depth only, so the viewing angle stays stable), plus the render-to-texture path used for downscaling |
| `ui.rs` | `bevy_egui` panel; changes are applied on mouse release rather than on every frame to avoid regenerating the world continuously |
| `trees.rs`, `lighting.rs` | Decoration and a simple directional light |

### Design notes

- **From a fixed grid to noise.** The first version was a static 31×31 grid with hard-coded tiles. Perlin noise replaced it so that every launch gives a coherent but different city, and the chaos slider became a single scalar that controls the whole layout.
- **Chunking was a performance necessity.** Rendering the whole city at once was too heavy; generating only the chunks near the camera brought the frame rate back and made the world effectively unbounded.
- **Two cameras.** The egui overlay needs a 2D camera, the city a 3D one. They are distinguished by a `Main3dCamera` marker component so that systems address the right one.
- **Render downscaling has a hard limit.** The scene is rendered to a low-resolution texture that is then stretched over a full-screen quad. This reduces fragment work, but Bevy's pipeline still runs every ECS system, entity update and light calculation at full cost — the gain is purely visual. A custom render graph would be needed to go further.

![Downscaling](./screenshots/scale.png)

## Screenshots

| Full resolution | Medium downscale | Strong downscale |
|---|---|---|
| ![](./screenshots/high_res.png) | ![](./screenshots/mid_res.png) | ![](./screenshots/low_res.png) |

| Zoomed out | Zoomed in |
|---|---|
| ![](./screenshots/zoom_out.png) | ![](./screenshots/zoom_in.png) |

| Low chaos | High chaos |
|---|---|
| ![](./screenshots/low_chaos.png) | ![](./screenshots/high_chaos.png) |

## Stack

Rust 2021 · Bevy 0.13 · bevy_egui 0.25 · noise 0.8 · rand 0.8
