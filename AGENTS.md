# AGENTS.md

## Cursor Cloud specific instructions

### Overview

`livekit-cli` is the LiveKit command-line tool (`lk`) — a Go project. It manages LiveKit rooms, tokens, egress/ingress, load testing, and an interactive agent console. The console feature builds unconditionally with **cgo**, linking vendored C/C++ (PortAudio + the WebRTC audio-processing module), so this is not a pure-Go build.

### Toolchain requirements (already provisioned in the Cloud VM)

- **Go 1.26+** (see `go.mod`). The default `apt` Go is too old; the VM has Go 1.26 at `/usr/local/go` (symlinked onto `PATH`).
- **cgo build deps**: `libasound2-dev` (ALSA headers) and the `pkg/portaudio/pa_src` git submodule (vendored PortAudio C source). Both are handled by the update script; the submodule is checked out via `git submodule update --init --recursive`.

### Build / run

```bash
make            # builds ./bin/lk (submodule init + CGO_ENABLED=1 go build ./cmd/lk)
./bin/lk --help
./bin/lk --version
```

`make install` copies `lk` to `$GOBIN` and adds a `livekit-cli` alias. The cgo build emits many harmless C++ compiler warnings (`-Wno-*` notes, unused-option notes) — these are expected, not errors. The first build is slow (~2 min) because it compiles the vendored WebRTC APM.

### Lint / test (mirrors `.github/workflows/test.yaml`)

```bash
go test ./...                 # unit tests (must init submodule first, e.g. via make)
golangci-lint run             # v2 config in .golangci.yml (staticcheck); binary not preinstalled
```

### Gotchas

- `TestSessionE2E` (`cmd/lk`) is **opt-in**: it only runs when `LIVEKIT_API_KEY` is set, and additionally needs a prepared Python agent venv (`livekit-agents`) plus live LLM credentials. It is skipped by default. If `LIVEKIT_API_KEY` is exported in your shell (e.g. left over from `lk room` commands), `go test ./...` will try to run it and fail — unset those vars for plain unit-test runs.
- To exercise `lk` against a server without a cloud account, run `livekit-server --dev` (preinstalled) and use `--url ws://localhost:7880 --api-key devkey --api-secret secret`.
