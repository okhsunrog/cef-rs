# Custom CEF 154 Wayland ABI

This branch targets the CEF fork at commit
`edaadb655607aea65355ab46605b235e0928de38` (Chromium 154.0.8037.98),
including the port of chromiumembedded/cef#4233. Linux x86_64 bindings are
regenerated from that fork's generated distribution headers. Other Linux
architectures need regeneration before use.

Linux uses `CEF_API_VERSION_EXPERIMENTAL` (999999), exposed as
`sys::CEF_API_VERSION_DEFAULT`. Declare it through `cef::api_hash` before the
first CEF call. Tauri additionally compares the generated platform hash
`sys::CEF_API_HASH_999999` with the loaded library before accessing vtables.
Other platforms retain the upstream versioned API selection.

`CEF_PATH` must point at the matching custom SDK. Linux builds refuse a missing
or stock SDK instead of silently downloading one. The directory must contain
`include/`, runtime shared objects/resources and `locales/`; the smoke app's
`install-sdk.sh` adapts CEF's usual Release/Resources layout using symlinks.

Regenerate after changing public CEF headers:

```sh
CEF_PATH_linux_x86_64=/absolute/path/to/custom/sdk \
  cargo run -p update-bindings -- --bindgen --target x86_64-unknown-linux-gnu
rustfmt --edition 2021 cef/src/bindings/x86_64_unknown_linux_gnu.rs \
  cef/src/resources/x86_64_unknown_linux_gnu.rs
```

Use the official generator rather than editing the generated bindings. CEF's
`cef_make_headers` target can generate distribution headers ahead of the full
browser build. Copy source includes, overlay `out/Release_GN_x64/gen/cef/include`
and apply the distribution's replacement of `cef_net_error_list.h` with
Chromium's `net/base/net_error_list.h` before binding generation.

The new APIs are Wayland display adoption, `WindowInfo::parent_xdg_surface`,
and `BrowserHost::set_window_bounds`. Bindings compile with `dox` without the
CEF library; this checks Rust types only and is not runtime acceptance.
