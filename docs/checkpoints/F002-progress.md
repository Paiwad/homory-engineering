# HOM-F002 Progress

## Checkpoint 4 — 2026-10-11

Status: in progress

- [x] Model, Room entity/DAO/migration, private storage foundation
- [x] MIME signature and maximum-size validation
- [x] Generic save/remove repository foundation
- [x] Atomic replacement and delete cleanup queue
- [x] Retry pending file cleanup at app start
- [x] Domain read use case and attachment ViewModel state
- [x] Item Detail attachment UI
- [x] Photo Picker, camera, document picker, FileProvider, viewer
- [ ] Unit, Room, repository, UI, and device tests

Verification so far: `:app:compileDebugKotlin` and `:feature:item:compileDebugKotlin` pass.

Remaining: add coverage for attachment DAO/repository and run device flow. The UI uses an
external viewer for stored image/PDF files; it reports an unavailable file only through the
current generic error path.
