# OAassist

**OAassist** is an on-prem documentation Q&A service. It indexes a body of documentation on the local machine and answers questions about it using RAG (retrieval-augmented generation) — nothing leaves the machine unless you explicitly configure a cloud LLM.

The intended deployment is alongside the OpenAxes apps (Radar, PuRe, Investigation Assistant): OAassist is installed as a Windows Service, each app calls its REST API (`POST /v1/ask`) for help-aware answers, and Claude Code / Desktop can also connect via MCP for ad-hoc queries.

For the long-form architecture and the build record, see [`docs/`](docs/) — `ARCHITECTURE.md` and `PHASE1_PLAN.md`.

---

## Choose your install path

| You are... | Go to |
|---|---|
| A customer IT admin installing on Windows Server | [Install with the MSI](#install-with-the-msi-windows-server) |
| A developer running OAassist locally | [Run from source](#run-from-source-development) or [Run in Docker](#run-in-docker-development) |

---

## Install with the MSI (Windows Server)

This is the customer-facing path. The MSI installs OAassist as a Windows Service, sets sensible defaults, and survives reboots.

> See [`installer/INSTALL.md`](installer/INSTALL.md) for the build engineer's guide (how to produce `OAassist.msi` from source).

### 1. Install

Silent install (recommended for automated deployment):

```cmd
msiexec /i OAassist.msi /qn
```

Or double-click `OAassist.msi` for the interactive installer.

### 2. Configure

The installer drops a default config at:

```
C:\ProgramData\OAassist\OAassist.env
```

The indispensable variable is **`DATA_PATH`** — absolute path to your documentation root. Without it the service starts but cannot index or answer.

For synthesized answers, also set `LLM_PROVIDER=anthropic` + `ANTHROPIC_API_KEY` (cloud, via Claude) or `LLM_PROVIDER=ollama` (offline, via a local Ollama daemon). Default is `noop` — returns raw chunks. See [Configuration](#configuration) below.

### 3. Drop your documentation

Place documentation files under:

```
C:\ProgramData\OAassist\docs\
├── radar\
├── pure\
└── ia\
```

Files in `radar/` get tagged `app="radar"`, etc. — see [Multi-app documentation](#multi-app-documentation).

### 4. Restart and reindex

The default `OAassist.env` ships `AUTH_ENABLED=true`, so `/v1/*` calls need a bearer service token — mint one with `"C:\Program Files\OAassist\OAassist.exe" manage issue-token <label>` (see [Security](#security)).

```cmd
"C:\Program Files\OAassist\nssm.exe" restart OAassist
curl -X POST http://localhost:8000/v1/reindex -H "Content-Type: application/json" -H "Authorization: Bearer <token>" -d "{}"
```

### 5. Verify

```cmd
curl http://localhost:8000/healthz
```

You should see `{"status":"ready","model_loaded":true,"collection_size":N}`. For service control and logs see [Troubleshooting](#troubleshooting).

---

## Run from source (development)

For developers who want to run OAassist on their own machine.

### 1. Prerequisites

You need **Python 3.11+**, **git**, and **uv** (Python package manager).

```powershell
# Windows PowerShell — install uv
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

```bash
# macOS / Linux / WSL — install uv
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Close and reopen your terminal, then `uv --version` to verify.

### 2. Clone and install

```bash
git clone https://github.com/OpenAxes/OAassist.git
cd OAassist
uv sync
```

### 3. Configure

```powershell
# Windows PowerShell
Copy-Item .env.example .env
```

```bash
# macOS / Linux / WSL
cp .env.example .env
```

Edit `.env`. The indispensable variable is **`DATA_PATH`** — absolute path to your documentation root. Without it the service starts but `/v1/reindex` returns 500 and there is nothing to answer about.

If you want synthesized answers instead of raw chunks, also set `LLM_PROVIDER=anthropic` (+ `ANTHROPIC_API_KEY`) for Claude, or `LLM_PROVIDER=ollama` for a local Ollama daemon (default model `qwen3:4b`). See [Configuration](#configuration) for the full list.

### 4. Index your documents

```bash
uv run ingest.py
```

The first run downloads the embedding model (~90 MB). Re-run whenever the documents change.

### 5. Start the service

```bash
uv run src/main.py
```

You should see `Uvicorn running on http://0.0.0.0:8000`. Leave it running — the same process serves both the REST API and the MCP server.

### 6. Verify

```bash
curl http://localhost:8000/healthz
```

---

## Run in Docker (development)

```bash
docker compose build
docker compose up
```

The container reads your `.env`, mounts your `DATA_PATH` into `/app/docs` read-only, and persists `chroma_db/` to the host. The default embedding model is baked into the image so the container runs fully offline once built.

At minimum your `.env` needs **`DATA_PATH`** (the host folder to mount). For synthesized answers, also set `LLM_PROVIDER` and the matching provider keys — see [Configuration](#configuration). If you pick `ollama`, the compose file already redirects to `host.docker.internal:11434`, so the daemon must be running on your host.

> Docker is provided **only for development and CI**. The customer-facing distribution is the Windows MSI above.

---

## Cheat sheet

Most-used commands once the environment is set up. See the sections above for first-time setup.

### Daily dev (uv)

```bash
uv sync                              # install / refresh deps from uv.lock
uv sync --group dev                  # + PyInstaller (only needed to build the MSI)
uv run src/main.py                   # start service on port 8000 (REST + MCP)
uv run ingest.py                     # re-index DATA_PATH from the CLI
LLM_PROVIDER=noop uv run testing/test_llm.py     # smoke-test the LLM layer
uv run testing/load_test.py --concurrency 10 --duration 60
```

### Docker

```bash
docker compose build
docker compose up
docker compose down
```

### Build the Windows MSI

```powershell
.\scripts\build-msi.ps1                    # full pipeline (~5 min)
.\scripts\build-msi.ps1 -SkipPyInstaller   # reuse dist\OAassist\, iterate on .wxs only
```

Pipeline: `uv sync --group dev` → `pyinstaller build/OAassist.spec` → copy `nssm.exe` into the bundle → `wix build` (compiles `.wxs` → `.msi`; the bundle is harvested inline via the `<Files>` element, no separate `heat` step). Output: `installer\OAassist.msi`. See [`installer/INSTALL.md`](installer/INSTALL.md).

### Service control (MSI install)

```cmd
"C:\Program Files\OAassist\nssm.exe" start OAassist
"C:\Program Files\OAassist\nssm.exe" stop OAassist
"C:\Program Files\OAassist\nssm.exe" restart OAassist
"C:\Program Files\OAassist\nssm.exe" status OAassist
```

### Health and reindex (any install path)

```bash
curl http://localhost:8000/healthz
curl -X POST http://localhost:8000/v1/reindex -H "Content-Type: application/json" -d "{}"
curl -X POST http://localhost:8000/v1/reindex -H "Content-Type: application/json" -d '{"app":"radar"}'
```

MSI installs ship `AUTH_ENABLED=true`: add `-H "Authorization: Bearer <token>"` to the `/v1/*` calls (`/healthz` is unauthenticated). Token minting: [Security](#security).

### MCP registration (Claude Code)

```bash
claude mcp add --transport http --scope user OAassist http://localhost:8000/mcp
claude mcp list
claude mcp remove OAassist --scope user
```

### Cleanup

```bash
make clean       # Python caches only (safe)
make clean-all   # caches + chroma_db (requires re-ingest)
```

---

## Connect Claude to OAassist

OAassist exposes an MCP server at `/mcp` over HTTP. Once the service is running (any install path), register it in Claude Code:

```bash
claude mcp add --transport http --scope user OAassist http://localhost:8000/mcp
claude mcp list   # should show OAassist with ask_documentation and read_document
```

**Claude Desktop needs a stdio-to-HTTP bridge** (it does not connect to HTTP servers directly — pointing its config at the URL is silently ignored). The full setup for both clients, the two tools, and MCP troubleshooting live in **[`docs/MCP.md`](docs/MCP.md)** — the single reference for everything MCP.

To test, ask Claude: *"Use ask_documentation to find [something you know is in your docs]."*

### Claude plugin (OpenAxes marketplace)

OAassist is also packaged as a Claude plugin, distributed through the OpenAxes plugin marketplace (`OpenAxes/OpenAxes.Claude.Marketplace`). Installing the plugin registers the MCP server and a `documentation-qa` skill that teaches Claude how to use the two tools -- no manual `claude mcp add` needed.

By default the plugin connects to the **hosted OAassist server** at `https://docassist.openaxes.com/mcp` (streamable HTTP behind the OpenAxes reverse proxy), so a marketplace user has nothing to install locally. To point the plugin at a different deployment -- a self-hosted MSI install on `localhost`, or another host on the LAN -- set the `OAASSIST_URL` environment variable to that server's `/mcp` URL before launching Claude Code (e.g. `OAASSIST_URL=http://localhost:8000/mcp`). If the target server runs with `AUTH_ENABLED=true`, also set `OAASSIST_TOKEN` to a service token minted by the admin (`OAassist.exe manage issue-token <label>`, one per person -- the label is the identity recorded in the query history); while auth is off the variable can stay unset.

The plugin source lives in this repo (`.claude-plugin/`, `skills/`) and is published to the marketplace by CI on every `v*` tag (`.github/workflows/publish-plugin.yml`) -- never edit the marketplace's `plugins/OAassist/` directory by hand. See [`docs/MARKETPLACE.md`](docs/MARKETPLACE.md) for the full publish flow, versioning, and how to cut a new release.

---

## Configuration

All settings live in `.env` (development) or `C:\ProgramData\OAassist\OAassist.env` (MSI install). They are read once at startup; restart the service to apply changes. One exception: the documents root can also be changed at runtime via `PUT /v1/config` (used by the web complement's UI) — the new path is persisted in `settings.json` next to the database, overrides `DATA_PATH`, and the `PUT` triggers a full reindex. `GET /v1/config` shows the path in effect and where it came from.

| Variable | Default | Description |
|---|---|---|
| `DATA_PATH` | _(required for ingest/reindex)_ | Absolute path to the documentation root. |
| `LLM_PROVIDER` | `noop` | One of `noop`, `anthropic`, `ollama`. See [LLM providers](#llm-providers). |
| `EMBEDDING_MODEL` | `BAAI/bge-small-en-v1.5` | Any [sentence-transformers](https://huggingface.co/sentence-transformers) model. Changing it invalidates existing embeddings — clear `chroma_db/` and re-ingest. |
| `RERANK_ENABLED` | `false` | Re-rank the dense candidate pool with a cross-encoder before returning. Improves precision (a near-duplicate or off-topic chunk in the top results confuses a small model) at the cost of a model load on first query and per-query CPU scoring. |
| `RERANK_MODEL` | `cross-encoder/ms-marco-MiniLM-L-6-v2` | Cross-encoder used when `RERANK_ENABLED=true`. `BAAI/bge-reranker-base` is slower but stronger. |
| `ANTHROPIC_API_KEY` | _(unset)_ | Required when `LLM_PROVIDER=anthropic`. |
| `ANTHROPIC_MODEL` | `claude-haiku-4-5-20251001` | Claude model used for synthesis. |
| `OLLAMA_URL` | `http://localhost:11434` | Ollama daemon URL. Used when `LLM_PROVIDER=ollama`. |
| `OLLAMA_MODEL` | `qwen3:4b` | Ollama model name. Must be `ollama pull`ed in the daemon first. |
| `OLLAMA_NUM_CTX` | `4096` | Context window for the Ollama call. Caps the KV cache (~128 KB/token on qwen3:4b), so it is the main RAM lever. Keep retrieved chunks + `OLLAMA_NUM_PREDICT` under this or Ollama silently drops the earliest tokens. Raise on roomier hardware. |
| `OLLAMA_NUM_PREDICT` | `512` | Max tokens in a synthesized answer. Default keeps CPU-only generation under the request timeout; raise if answers are being cut short. |
| `OLLAMA_NUM_GPU` | _(unset)_ | Number of model layers to offload to the GPU. Leave unset — Ollama auto-detects a GPU and offloads as many layers as fit in VRAM. Set only to override: `0` forces CPU; a small number offloads partially on a VRAM-limited GPU. |
| `AUTH_ENABLED` | `false` | Require a bearer service token on `/v1` + `/mcp`. Keep `true` on any reachable install. See [Security](#security). |
| `DEPLOYMENT_MODE` | `standard` | `on_prem` refuses to start with a non-local LLM provider, guaranteeing document content never leaves the installation. See [docs/LLM_WIKI.md](docs/LLM_WIKI.md). |
| `LLM_WIKI_ENABLED` | `false` | Query-driven synthesized wiki layer. Off = no wiki components, no extra LLM calls. See [docs/LLM_WIKI.md](docs/LLM_WIKI.md). |
| `LLM_WIKI_MAX_RESULTS` | `3` | Max wiki notes added to one answer's context. |
| `LLM_WIKI_RAW_RESULT_WEIGHT` | `1.0` | Authority weight of raw chunks when gating wiki notes. |
| `LLM_WIKI_PAGE_RESULT_WEIGHT` | `0.8` | Authority weight of wiki pages; lower than the raw weight = stricter gate. |
| `LLM_WIKI_REQUIRE_APPROVAL` | `false` | Only human-approved wiki pages enter the answer context. |
| `DATABASE_PATH` | `<ProgramData>\OAassist\oaassist.db` | SQLite file for service tokens and query history. |
| `LOG_LEVEL` | `INFO` | Logging verbosity. |

OAassist validates the config at startup: if `LLM_PROVIDER=anthropic` but no key is set, `LLM_PROVIDER` is an unknown value, or `DEPLOYMENT_MODE=on_prem` is combined with a non-local provider, the service refuses to start with a clear error message.

---

## LLM providers

OAassist supports three modes for how it produces an answer from the retrieved chunks:

| Provider | What it does | When to use it |
|---|---|---|
| `noop` (default) | Returns the raw chunks joined. No synthesis. | When Claude (via MCP) will read the chunks and write the answer itself, or when the calling app has its own LLM. |
| `anthropic` | Calls Claude over the Anthropic API to synthesize an answer. | When apps call `POST /v1/ask` and need a finished answer. Requires `ANTHROPIC_API_KEY`. |
| `ollama` | Calls a local Ollama daemon to synthesize an answer. | Offline / air-gapped deployments. Default model `qwen3:4b`. Requires a running Ollama daemon with the model pulled. |

To switch providers, edit `LLM_PROVIDER` and restart the service.

**Precision (`ollama`).** OAassist calls qwen3:4b with `temperature 0` (deterministic, fact-bound). The context window (`OLLAMA_NUM_CTX`) and answer length (`OLLAMA_NUM_PREDICT`) default to values sized for the smallest target — qwen3:4b on an 8 GB CPU-only box — and are raisable on roomier hardware. Keep the injected chunks plus `OLLAMA_NUM_PREDICT` under `OLLAMA_NUM_CTX` or the prompt is silently truncated. For higher precision at the cost of RAM and speed, build a higher-quantization variant from `ollama/Modelfile` and point `OLLAMA_MODEL` at it. For the daemon-side memory tuning on constrained boxes, see [`installer/INSTALL.md`](installer/INSTALL.md).

---

## Multi-app documentation

A single OAassist install can serve multiple apps, each with multiple versions. Documentation is organized in subfolders by app, then by version:

```
DATA_PATH/
├── radar/
│   ├── 1.4.0/
│   └── 1.5.0/
├── pure/
└── ia/
```

Each chunk is tagged with the first directory segment as its **app** and the second as its **version**. A REST call with `"app": "radar"` filters retrieval to radar chunks; add `"version": "1.4.0"` to narrow to one version. A single level (`radar/` with files directly inside) still works — those chunks just have no version. Files directly under `DATA_PATH` get no app tag.

To re-index one scope, send `{"app": "radar"}` or `{"app": "radar", "version": "1.4.0"}` to `/v1/reindex` — it rebuilds only that scope.

Apps and versions are managed from the separate web complement, which writes into this folder tree and calls `/v1/reindex` — see [`docs/COMPLEMENT.md`](docs/COMPLEMENT.md).

---

## Security

OAassist is the AI engine. The human-facing web — Microsoft sign-in, the admin UI,
users and roles — lives in a **separate .NET/Angular complement** that calls this
API with a service token. See [`docs/COMPLEMENT.md`](docs/COMPLEMENT.md).

With `AUTH_ENABLED=true`, **REST (`/v1/*`) and MCP (`/mcp`)** require a bearer
service token: `Authorization: Bearer <token>`. A `401` means it is missing or
invalid. Mint a token from the CLI (it prints once, on stdout):

```bash
uv run python -m src.manage issue-token radar-backend
# on an MSI install:
"C:\Program Files\OAassist\OAassist.exe" manage issue-token radar-backend
```

The calling app asserts who the human is by passing `user.email` in the `/v1/ask`
body; OAassist records it in the query history.

> **`AUTH_ENABLED=false` is for localhost development only** — it opens `/v1` and
> `/mcp`. Any reachable deployment must run with `AUTH_ENABLED=true`.

---

## API endpoints

All endpoints return JSON. The service listens on port 8000. When `AUTH_ENABLED=true`, add `-H "Authorization: Bearer <token>"` to the `/v1/*` calls below.

> Integrating another app into OAassist via the REST API? See [`docs/API_INTEGRATION_QUICKSTART.md`](docs/API_INTEGRATION_QUICKSTART.md) for the full request/response contract, including authentication, the optional `version` filter, and the `user` / `page` caller-context fields.

### `POST /v1/ask`

```bash
curl -X POST http://localhost:8000/v1/ask \
     -H "Content-Type: application/json" \
     -d '{"query":"How do I close a case?","app":"radar"}'
```

PowerShell equivalent in [`examples/powershell/ask.ps1`](examples/powershell/ask.ps1).

### `POST /v1/reindex`

```bash
# Re-ingest everything
curl -X POST http://localhost:8000/v1/reindex \
     -H "Content-Type: application/json" -d '{}'

# Re-ingest a single app
curl -X POST http://localhost:8000/v1/reindex \
     -H "Content-Type: application/json" -d '{"app":"radar"}'
```

### `GET /healthz`

```bash
curl http://localhost:8000/healthz
```

### Other endpoints

- `POST /v1/suggest` and `POST /v1/suggest-questions` — optional context-driven suggestion plugins; contract in [`docs/API_INTEGRATION_QUICKSTART.md`](docs/API_INTEGRATION_QUICKSTART.md).
- `GET /v1/history` — recent query-log entries.
- Document, app, and config management (`/v1/apps`, `/v1/documents*`, `GET`/`PUT /v1/config`) — used by the web complement; see [`docs/COMPLEMENT.md`](docs/COMPLEMENT.md).

Ready-to-run scripts in [`examples/curl/`](examples/curl/) and [`examples/powershell/`](examples/powershell/).

---

## How OAassist works

```
                      one process, one port (8000)
        ┌─────────────────────────────────────────────┐
   MCP  ──▶  /mcp              ◀── Claude Code/Desktop │
                                                       │
   HTTP ──▶  /v1/ask     ◀── apps (Radar/PuRe/IA)      │
            /v1/reindex                                │
            /healthz                                   │
                       │                               │
                       ▼                               │
                  AskService                           │
                       │                               │
                       ├──▶ RagEngine ──▶ chroma_db    │
                       └──▶ LLM provider (noop/anth.)  │
        └─────────────────────────────────────────────┘
```

- **Local embeddings** with `sentence-transformers` — no API costs.
- **Vector database** with `chromadb` — embedded SQLite, stored locally.
- **MCP server** with the official `mcp` Python SDK over streamable-HTTP.
- **RAG pipeline**: ingestion → chunking (1000 chars, 200 overlap) → embeddings → vector search.
- **Pluggable LLM providers** in `src/llm/` — synthesis is optional. With the default `noop`, the MCP path leverages Claude's own reasoning; the REST path returns the raw chunks for the caller to handle.

Project structure:

```text
OAassist/
├── chroma_db/           # Local vector database (gitignored)
├── src/
│   ├── main.py          # Unified entrypoint (REST + MCP, single process)
│   ├── rest_api.py      # FastAPI routes
│   ├── ask_service.py   # Shared core: orchestrates one Q&A request
│   ├── rag.py           # Embedding + ChromaDB
│   ├── config.py        # Single source of truth for env config
│   ├── llm/             # Pluggable LLM providers (noop, anthropic, ollama)
│   └── plugins/         # Optional endpoints (/v1/suggest, /v1/suggest-questions)
├── ingest.py            # CLI ingestion (alternative to POST /v1/reindex)
├── build/               # PyInstaller spec + runtime hook
├── installer/           # WiX MSI definition + INSTALL.md
├── examples/            # curl, PowerShell, MCP registration
├── scripts/             # build-msi.ps1 (MSI build)
├── testing/             # test/eval tooling: test_llm.py, load_test.py, eval_retrieval.py
└── docs/                # Architecture, integration quickstart, MCP, build record
```

For the full architecture and target design, see [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md). For the staged build plan, [`docs/PHASE1_PLAN.md`](docs/PHASE1_PLAN.md).

---

## Maintenance

**Clean Python cache (safe):**
```bash
make clean
```

**Clean cache + database** (requires re-indexing):
```bash
make clean-all
```

> On native Windows, `make` is not installed by default. Use **Git Bash**, **WSL**, or manually delete the `chroma_db/` folder.

### Load testing

A simple load tester is included:

```bash
uv run testing/load_test.py --concurrency 10 --duration 60
```

It reports throughput and p50 / p95 / p99 latency. See [`docs/QA_CHECKLIST.md`](docs/QA_CHECKLIST.md) for the full release QA matrix.

---

## Troubleshooting

### Service status (MSI install)

```cmd
"C:\Program Files\OAassist\nssm.exe" status OAassist
```

Logs:

```
C:\ProgramData\OAassist\logs\stdout.log
C:\ProgramData\OAassist\logs\stderr.log
```

To restart / stop / start:

```cmd
"C:\Program Files\OAassist\nssm.exe" restart OAassist
"C:\Program Files\OAassist\nssm.exe" stop OAassist
"C:\Program Files\OAassist\nssm.exe" start OAassist
```

### "Address already in use" on port 8000

Another process holds the port (typically a previously-started OAassist or another dev server).

```powershell
# Windows PowerShell — find the holder
Get-NetTCPConnection -LocalPort 8000 -State Listen | Select-Object OwningProcess

# Stop it
Stop-Process -Id <pid> -Force
```

```bash
# macOS / Linux
lsof -i :8000
kill <pid>
```

### Startup error: configuration validation

If you see something like `LLM_PROVIDER=anthropic but ANTHROPIC_API_KEY is not set` or `Invalid LLM_PROVIDER 'xyz'`, edit the env file and restart. OAassist refuses to start with invalid config — by design.

### `uv` is not found after installing

The installer adds `uv` to `PATH` (`%USERPROFILE%\.local\bin` on Windows, `~/.local/bin` on Linux/macOS) but sometimes you need to add it manually:

```powershell
# Windows PowerShell — persistent for your user
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";$env:USERPROFILE\.local\bin", "User")
```

```bash
# macOS / Linux / WSL
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc   # or ~/.zshrc
source ~/.bashrc
```

### `ModuleNotFoundError: No module named 'src'`

Make sure you run the server with `uv run src/main.py` (not `python src/main.py`). The project includes a `sys.path` fix that only works inside the `uv` environment.

### `make` is not recognized (Windows)

`make` isn't included on Windows. Use the manual commands from the Quick Start, or install **Git Bash** / **WSL** if you want the `Makefile` shortcuts.

### MCP problems (failed to connect, tool not used, Desktop issues)

All MCP troubleshooting — connection failures, the stdout-pollution error, Claude Desktop setup — lives in [`docs/MCP.md`](docs/MCP.md). The 30-second version: make sure `curl http://localhost:8000/healthz` returns 200 *before* registering or launching the client.

---

## FAQ

**Where does the embedding model come from?**
It's downloaded automatically the first time you run `uv run ingest.py` (or on first request to the service). It's cached locally (`~/.cache/huggingface/hub/` on Linux/Mac, `C:\Users\<you>\.cache\huggingface\hub\` on Windows). No account or manual setup required. The MSI installer bundles the default model into the install image so customer machines work offline.

**Can I use a different embedding model?**
Yes. Change `EMBEDDING_MODEL` in your env file to any model from [sentence-transformers on HuggingFace](https://huggingface.co/sentence-transformers). The default (`BAAI/bge-small-en-v1.5`, ~130 MB) won the retrieval eval on our corpus (`testing/eval_embeddings.py`). After switching, run `make clean-all` and re-index — vectors from one model aren't compatible with another. Note: bge models need a query-side prefix, already handled in `src/rag.py` (`QUERY_PREFIXES`); add an entry there if your model needs one too.

**What happens to the old embedding model when I switch?**
It stays in the HuggingFace cache taking up disk space until you delete it manually:

```powershell
# Windows PowerShell
Remove-Item -Recurse "$env:USERPROFILE\.cache\huggingface\hub\models--sentence-transformers--all-MiniLM-L6-v2"   # example: the old default
```

```bash
# macOS / Linux / WSL
rm -rf ~/.cache/huggingface/hub/models--sentence-transformers--all-MiniLM-L6-v2
```

**Does OAassist phone home?**
No. With `LLM_PROVIDER=noop` (the default) nothing leaves the host. With `LLM_PROVIDER=anthropic` the question + retrieved chunks go to Anthropic — for answers and, when `LLM_WIKI_ENABLED=true`, for wiki page synthesis too. The embedding model runs locally; documents and embeddings never leave the host. To make locality a hard guarantee, set `DEPLOYMENT_MODE=on_prem`: the service then refuses to start with any non-local provider.

**Why is the docker image so big (~5.7 GB)?**
It's mostly torch + transformers. The PyInstaller MSI bundle is much smaller (~860 MB).
