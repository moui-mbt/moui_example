# MoUI Quick Example

MoUI Quick Example is the minimal standalone counter app that demonstrates
the core MoUI patterns in the smallest possible surface: a single
`Model { count }`, a `Msg` enum (`Increment` / `Decrement` / `Reset`), a
pure `update`, a declarative `view`, and a `@moui.Program::simple`
construction. It is the recommended starting template for new MoUI apps and
the repo referenced by the [MoUI README](https://github.com/wzzc-dev/MoUI)
[Quick Start](https://github.com/wzzc-dev/MoUI#quick-start).

This is a standalone repository that pins `wzzc-dev/moui@0.1.6` from the
MoonBit package registry; it is **not** a workspace member of the main MoUI
mono-repo and is not built by the MoUI `dev-check.sh` baseline.

## Package Shape

- `app/` — shared counter app logic. `program()` builds the
  `@moui.Program[Model, Msg]` from `Model::new()`, `update`, and `view`.
  The view composes `@views.{ center, card, column, row, text, button }`
  into a centered card with a `+` / `Reset` / `-` row. Supported on both
  `native` and `wasm-gc` targets.
- `macos_skia/` — macOS native entrypoint using the Skia raster provider.
  Runs the app at 520×360 with the title "MoUI Quick Example".
- `linux_skia/` — Linux native entrypoint using the Skia raster provider
  with the Wayland `wl_shm` presenter. Runs the app at 520×360.
- `windows_skia/` — Windows native entrypoint using the Skia raster provider
  with the Win32 GDI presenter. Runs the app at 520×360.
- `web_wasm/` — Web wasm-gc entrypoint using the browser WebGPU host import
  path. Imports `wzzc-dev/moui/backend/web` and exports the
  `web_dispatch_event` / `web_complete_async_*` bridge functions needed by
  the browser host.

## Prerequisites

- [MoonBit](https://www.moonbitlang.com/) toolchain (`moon` on PATH)
- For macOS Skia: a native macOS host (Skia is vendored by `moui_skia`)
- For Web wasm-gc: a static file server (`python3 -m http.server` is fine)
  served over HTTP so the wasm-gc + WebGPU bootstrap runs.

## Dependencies

```toml
import {
  "wzzc-dev/moui@0.1.6",
}
```

The macOS entrypoint additionally imports `wzzc-dev/moui/backend/macos/skia`
and `wzzc-dev/moui/runtime`; the Web entrypoint imports
`wzzc-dev/moui/backend/web` and `wzzc-dev/moui/runtime`. Both platform
entrypoints render the same shared `app/` logic, which is the motivating
property of MoUI: one typed app package, multiple native and Web hosts.

## Running

After cloning, refresh the package cache:

```sh
moon update
```

### macOS Skia

```sh
moon run macos_skia --target native
```

A 520×360 AppKit window titled "MoUI Quick Example" opens, renders a centered
card with the current count, and dispatches `+` / `Reset` / `-` button
clicks through the Skia raster mainline.

### Linux Skia

Requires a Wayland compositor and Skia link flags. See the [MoUI platform notes](https://github.com/wzzc-dev/MoUI/blob/main/docs/platform-notes.md#linux-native) for full setup.

```sh
moon run linux_skia --target native
```

### Windows Skia

Requires Visual Studio C++ build tools and vcpkg `zlib:x64-windows`. See the [MoUI platform notes](https://github.com/wzzc-dev/MoUI/blob/main/docs/platform-notes.md#windows-native) for MSVC setup.

```powershell
powershell -ExecutionPolicy Bypass -Command "& { . .\scripts\windows\msvc_env.ps1; moon run windows_skia --target native }"
```

### Web wasm-gc

```sh
moon build web_wasm --target wasm-gc
python3 -m http.server 8080 --bind 127.0.0.1
```

Open the index in a WebGPU-capable browser:

```text
http://127.0.0.1:8080/web_wasm/index.html
```

## Tests

The app package ships a counter-app runtime whitebox smoke test:

```sh
moon test app --target native
```

## Structure

```text
examples/moui_example
├── moon.mod           ← standalone module (pin moui@0.1.6)
├── app
│   ├── app.mbt        ← Model / Msg / update / view / program
│   └── counter_app_test.mbt
├── macos_skia         ← native Skia raster entrypoint
├── linux_skia         ← native Wayland Skia entrypoint
├── windows_skia       ← native Win32 Skia entrypoint
└── web_wasm           ← Web wasm-gc + browser WebGPU host entrypoint
```

## Scope

The app deliberately avoids routing, forms, data tables, navigation, rich
text, theming editors, image loaders, and any non-render component helpers.
For any of those, see the [MoUI mono-repo examples](https://github.com/wzzc-dev/MoUI/tree/main/examples)
— `showcase` (visual component catalog), `markdown_editor` (WYSIWYG
editor), `mo_workbench` (desktop agent dogfood), and `excel` (workbook
renderer) are the featured richer apps.

## License

Apache-2.0, matching MoUI.
