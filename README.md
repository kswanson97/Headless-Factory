# The Headless Factory

A private, locally-hosted AI development studio. One Chief of Staff agent today; a team of specialized agents that plan, build, test, deploy, and maintain software autonomously by the end of summer.

Designed around seven goals: free to run, local-first, cloud opt-in only, open-source throughout, scalable, multi-agent, and accessible to non-technical users.

> **Status:** v0.1 — conversational Chief of Staff with persistent memory, swappable cloud backends, and a working CLI. See [`ROADMAP.md`](./ROADMAP.md) for what's next.

---

## Quickstart

**Prerequisites:** Docker and Docker Compose. That's it for cloud backends. GPU + NVIDIA Container Toolkit if you plan to run a local model later.

```bash
# 1. Clone
git clone <your-remote-url> headless-factory
cd headless-factory

# 2. Configure
cp .env.example .env
# Edit .env — at minimum set OPENROUTER_API_KEY (free tier works)

# 3. Launch
docker compose up -d --build

# 4. Verify
./test.sh

# 5. Chat
python3 chat.py
```

The API listens on `http://localhost:3100`. It speaks OpenAI-compatible `/v1/chat/completions`, so any client that talks to OpenAI can point at the Factory.

---

## Repo layout

```
headless-factory/
├── api/                      # The FastAPI service
│   ├── Dockerfile
│   ├── main.py               # API, memory, backend routing
│   └── requirements.txt
├── chat.py                   # Terminal CLI client
├── test.sh                   # Smoke test (6-step E2E)
├── docker-compose.yml        # redis + api (llama-server added in Phase 2)
├── .env.example              # Copy to .env
├── .gitignore
├── BRANCHES.md               # stable / rolling branch model
├── ROADMAP.md                # Phase 0 → 5 plan
└── README.md
```

---

## Configuration

All configuration lives in `.env`. The full list of variables is in [`.env.example`](./.env.example). The ones you'll touch most often:

- **`BACKEND`** — `openrouter`, `gemini`, or `local`
- **`OPENROUTER_API_KEY`** / **`OPENROUTER_MODEL`** — when `BACKEND=openrouter`
- **`COS_NAME`** — what the assistant calls itself (default: `Winston`)
- **`COS_SOUL`** — optional persona prelude prepended to the system prompt

After editing `.env`:

```bash
docker compose restart api
```

---

## Common commands

```bash
docker compose ps            # service status
docker compose logs -f api   # tail API logs
docker compose restart api   # pick up .env changes
docker compose down          # stop everything
./test.sh                    # smoke test
python3 chat.py --help       # CLI options
```

---

## CLI

```bash
python3 chat.py                         # fresh conversation, random ID
python3 chat.py --conv project-alpha    # resume/start a named conversation
python3 chat.py --host 1.2.3.4:3100     # point at a different server
python3 chat.py --no-stream             # block mode, no streaming
```

In-session commands: `/help`, `/new`, `/clear`, `/history`, `/model`, `/exit`. Multiline input with a trailing `\`. Ctrl-C once cancels an in-flight response; twice exits.

---

## API surface (current)

```
GET    /v1/system/health            health + active backend
GET    /v1/models                   OpenAI-compatible model listing
POST   /v1/chat/completions         chat (streaming or blocking)
GET    /v1/memory/{conv_id}         inspect a conversation's hot memory
DELETE /v1/memory/{conv_id}         clear a conversation's hot memory
```

Conversations are keyed by the `user` field in chat requests. Omit it for stateless, ephemeral replies.

---

## Switching to a local model (Phase 2)

When your GPU is ready:

1. Install the [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html).
2. Download a GGUF model into `./models/`.
3. Set `BACKEND=local`, `LLM_URL=http://llama-server:8080/v1`, `LLM_MODEL=<your-model>` in `.env`.
4. Add the `llama-server` service to `docker-compose.yml` (the Phase 5 installer will automate this).
5. `docker compose up -d --build`.

Memory, API shape, and all clients stay identical.

---

## Contributing

See [`BRANCHES.md`](./BRANCHES.md) for the branch model before opening a PR. The short version: feature branches merge into `main`; `rolling` tracks `main`; `stable` is promoted from `rolling` after soak testing.

---

## License

AGPL-3.0 (landing with v1.0 in Phase 5). Until then, this repo is a work in progress and not yet licensed for redistribution.
