# Session Overview

- Goal: add a root `Taskfile.yml` that detects the repo language and exposes common tasks.
- Scope: `build`, `test`, `lint`, and `clean`.
- Primary language detected in this repo: Rust workspace under `rust/`.
- Success criteria: `task detect`, `task build`, `task test`, `task lint`, and `task clean` map to the Rust workspace without manual path selection.
