# gpui-pre-macos (patched for diffz)

This is the published [`gpui-pre-macos`](https://crates.io/crates/gpui-pre-macos) 0.3.3
crate, a [gpui-kit](https://github.com/longbridge/gpui-kit) snapshot of Zed's
`gpui_macos` at `zed@5b055fa789a8b8d38ac951a6e0cde272f66b4495`, with one patch
carried for [diffz](https://github.com/zzwong/diffz) until an upstream gpui-pre
snapshot stops the display link of idle windows:

- **Stop the display link while a window has nothing to draw.** A visible
  window kept its `CVDisplayLink` subscription for as long as it stayed
  visible, so the main thread woke at the display's refresh rate to run a
  frame callback that found nothing dirty. The patch implements GPUI's
  `frame_waker` for macOS, the hook the web platform already uses: the link
  stops after four consecutive frames that neither draw nor present nor ask
  for another frame, and restarts when the window is invalidated, schedules a
  next-frame or animation-frame callback, receives input, or becomes visible
  or changes screens. Continuous animation keeps the link running because
  every frame draws. See [diffz#76](https://github.com/zzwong/diffz/issues/76).

Before the patch, diffz measured about 0.3–0.7% CPU and 300 context switches a
second in a visible idle window (release build, 30 s of `top`, macOS 15.7);
occluded windows were already idle. The patch only changes visible windows, so
the effect is the difference between those two states.

`v0.3.3-upstream` tags the pristine published crate, so `git diff
v0.3.3-upstream` is the full carried delta. This repository is temporary and
will be archived once a gpui-pre snapshot ships an equivalent change.

Licensed under Apache-2.0, as the upstream crate is.
