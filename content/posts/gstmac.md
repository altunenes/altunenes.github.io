+++
title = "How We Fixed macOS GStreamer Library Path Issues in Rust Releases"
date = "2025-09-02"
[taxonomies]
tags=["GStreamer", "macOS", "Rust"]
+++

## <span style="color:orange;">How We Fixed macOS GStreamer Library Path Issues in Rust Releases</span>

I spent way too many hours debugging why our Rust app worked fine in development but kept crashing on macOS release builds due to GStreamer library conflicts. Users would see this error when double-clicking the app:

```bash
  objc[43583]: Class GstCocoaApplicationDelegate is implemented in both
  /Library/Frameworks/GStreamer.framework/Versions/1.0/lib/libgstreamer-1.0.0.dylib
  and /Users/user/app/gstreamer/lib/libgstreamer-1.0.0.dylib
```

### <span style="color:orange;">The Problem</span>

Turns out our GitHub Actions script was trying to fix `@rpath` entries with `install_name_tool`, but those commands were just failing silently.


Rust binaries don't have enough header padding by default for `install_name_tool` to actually work.

### <span style="color:orange;">The Solution</span>

Add this linker flag to your macOS Rust builds:

```bash
export RUSTFLAGS="$RUSTFLAGS -C link-arg=-Wl,-headerpad_max_install_names"
```

This simple flag reserves enough space in the binary header so the tool can actually do its job.

- **Before:** Binary had unfixable `@rpath` entries → loaded both system and bundled GStreamer → conflicts
- **After:** Binary library paths properly fixed → loads only bundled GStreamer → works perfectly


So always verify that your `install_name_tool` commands actually succeed. Silent failures can waste hours of debugging!
