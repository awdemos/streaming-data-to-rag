# Agent Notes: Streaming Data to RAG

NVIDIA AI Blueprint developer example that turns live RF/audio streams into a searchable, real-time RAG index. The stack uses Docker Compose, NVIDIA NIMs, Holoscan SDR, and a context-aware RAG backend.

## Repository Layout

- `src/software-defined-radio/` — FM-radio ingestion pipeline and Holoscan SDR service.
- `src/file-replay/` — File-replay service that simulates SDR baseband I/Q samples from audio files (no hardware required).
- `external/` — Git submodules for NeMo Agent Toolkit UI and Context-Aware RAG.
- `deploy/docker-compose.yaml` — Main compose file for the replay workflow.
- `notebooks/` — Jupyter walkthroughs, including `quickstart.ipynb`.
- `docs/` — Additional documentation.
- `api-integration.md`, `hardware-integration.md`, `troubleshooting.md` — Integration guides.

## Prerequisites

- Docker & Docker Compose.
- NVIDIA GPU with CUDA drivers and the NVIDIA Container Toolkit.
- NVIDIA API key from [build.nvidia.com](https://build.nvidia.com).
- (Optional) Physical SDR hardware; otherwise use file-replay mode.

## Initial Setup

```bash
git submodule update --init --recursive

export NVIDIA_API_KEY=<your-api-key>
export MODEL_DIRECTORY=$HOME/.cache/nim   # optional NIM cache path
```

For file-replay mode (no SDR hardware):

```bash
export REPLAY_FILES="sample_files/ai_gtc_1.mp3,sample_files/ai_gtc_2.mp3"
export REPLAY_TIME=3600
export REPLAY_MAX_FILE_SIZE=50
```

Replay files must live under `src/file-replay/files/`.

## Build

```bash
# Context-aware RAG services
docker compose -f external/context-aware-rag/docker/deploy/compose.yaml build

# FM-radio ingestion workflow (file-replay profile)
docker compose -f deploy/docker-compose.yaml --profile replay build
```

## Deploy

```bash
# 1. Start context-aware RAG
docker compose -f external/context-aware-rag/docker/deploy/compose.yaml up -d

# 2. Start the FM-radio ingestion workflow
docker compose -f deploy/docker-compose.yaml --profile replay up -d
```

## Verify

Check service health:

```bash
docker compose -f deploy/docker-compose.yaml ps
```

Open the NeMo Agent Toolkit UI and try a question such as _"What was discussed in the latest AI Podcast segment?"_.

## Key Conventions

- Always run `git submodule update --init --recursive` after cloning.
- Keep replay audio files inside `src/file-replay/files/` or the `file-replay` build context will not include them.
- NIM model weights are cached under `MODEL_DIRECTORY` (default `~/.cache/nim`).

## Common Issues

- **Submodules empty**: re-run `git submodule update --init --recursive`.
- **NIM container fails**: confirm `NVIDIA_API_KEY` is exported and the GPU has enough VRAM.
- **No audio in replay**: verify `REPLAY_FILES` paths are relative to `src/file-replay/files/` and the files are inside that directory.
- **Compose profile errors**: use `--profile replay` when starting the main ingestion services.

## License

See `LICENSE` and `SECURITY.md`.
