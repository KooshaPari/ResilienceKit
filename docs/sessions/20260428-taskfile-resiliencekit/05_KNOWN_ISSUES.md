# Known Issues

- The repository layout includes Go and Python directories, but the current task runner targets the Rust workspace that exists in this checkout.
- The Rust workspace was narrowed to the two crates present in this clone so `cargo`-based tasks can run without missing local path dependencies.
