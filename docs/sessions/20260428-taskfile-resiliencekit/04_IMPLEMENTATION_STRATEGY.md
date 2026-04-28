# Implementation Strategy

- Use a single root `Taskfile.yml`.
- Detect language from manifests with a simple shell probe.
- Dispatch to repository-native commands instead of inventing wrappers.
- Prefer Rust workspace commands because the repo is Rust-first and the workspace lives under `rust/`.
