## Go / Google Cloud / Generative AI

English | [日本語](README.ja.md)

I build **17 public Go libraries** on my own, and the generative applications that combine them on
**Cloud Run + Cloud Tasks**. On the side, I run an AI music project, **Digital Armor Style**, whose music videos
are drawn from the audio by code.

I spend more time recording **where the boundaries are drawn** than on what I build: what each library does not
take on, why it sits on that layer, which options were rejected. Everything that cannot be written from inside a
single repository is collected in [**public-docs**](https://github.com/shouni/public-docs).

* 🗺️ [**Library reference**](https://github.com/shouni/public-docs/blob/main/docs/libraries.md) — a map of the 17 libraries' boundaries (scope, lines with neighbours, layer)
* 🚀 [**Application reference**](https://github.com/shouni/public-docs/blob/main/docs/applications.md) — what each app keeps and what it delegates
* 📐 [**Conventions**](https://github.com/shouni/public-docs#documents) — cross-repository rules for URL naming, workers and READMEs
* 📝 [**Zenn @snknsk**](https://zenn.dev/snknsk) — records of design decisions

New here? Start with [**the overview of the ecosystem**](https://zenn.dev/snknsk/articles/shouni-go-ecosystem-overview).

> public-docs and the articles are written in Japanese. The Python tools below have English READMEs.

---

### 🏗️ Core libraries

The layer with no AI at all. Usable as is by applications that generate nothing.

| | Scope |
|---|---|
| [**netarmor**](https://github.com/shouni/netarmor) | Safety of outbound traffic only. Knows nothing of AI or cloud. Empty `require` |
| [**go-utils**](https://github.com/shouni/go-utils) | `jobid` / `jst` / `paging` / `slogctx` / `strlist`. Admission rule: zero dependencies, no I/O, used by two or more repositories |
| [**go-serve-kit**](https://github.com/shouni/go-serve-kit) | The routine of answering requests (defensive headers, content negotiation, role declaration). No server of its own |
| [**gcp-kit**](https://github.com/shouni/gcp-kit) | Cloud Run web and Cloud Tasks workers. `worker.Lifecycle` owns the order of a job's life |
| [**go-http-kit**](https://github.com/shouni/go-http-kit) | A `net/http`-compatible client with SSRF protection and exponential backoff built in |
| [**go-remote-io**](https://github.com/shouni/go-remote-io) | GCS / S3 / local behind a single `Open(ctx, path)`. Where artifacts are stored |
| [**go-web-reader**](https://github.com/shouni/go-web-reader) | Reads `https://` / `gs://` / `s3://` through one entry point. Where sources are fetched |
| [**audio**](https://github.com/shouni/audio) | Audio binaries and Japanese readings only |
| [**go-job-kit**](https://github.com/shouni/go-job-kit) | Submitting, recording and listing jobs. Stays out of the artifacts' domain |
| [**go-notify**](https://github.com/shouni/go-notify) | Only delivering results to people. No CLI |

### 🤖 AI abstraction libraries

The layer that keeps the `genai` SDK out of public APIs.

| | Scope |
|---|---|
| [**go-prompt-kit**](https://github.com/shouni/go-prompt-kit) | Prompt management, and turning Markdown / JSON responses into HTML |
| [**genai-kit**](https://github.com/shouni/genai-kit) | Text, image, music and video generation with call guards. Reference images are `gs://`, resolved by the model side |
| [**go-character-kit**](https://github.com/shouni/go-character-kit) | Loading and validating character definitions only |
| [**go-voicevox**](https://github.com/shouni/go-voicevox) | `[]ScriptLine` → one joined WAV. The caller decides where it is stored |

### 🎨 Generation orchestrators

The layer that owns how the steps are assembled. Storage and notification are the caller's implementations.

| | Scope |
|---|---|
| [**go-comic-kit**](https://github.com/shouni/go-comic-kit) | Builds comics step by step while keeping characters consistent |
| [**go-veo-orchestrator**](https://github.com/shouni/go-veo-orchestrator) | Structures a song's composition sheet into video cuts and hands them to Veo |
| [**go-review-kit**](https://github.com/shouni/go-review-kit) | Git diff → AI review → publish → notify as one pipeline |

---

### 🚀 Applications

Running on Cloud Run / Cloud Tasks. Authentication, sessions, CSRF and OIDC are not implemented in the apps;
all of it lives in `gcp-kit/auth`.

| | What it does | Key design point |
|---|---|---|
| [**ap-mv**](https://github.com/shouni/ap-mv) | Generates music videos with Veo from a composition sheet | **Generates one cut at a time and the worker resubmits itself**, getting past Cloud Tasks' time limit and resuming midway |
| [**ap-story**](https://github.com/shouni/ap-story) | Generates chapters, storyboards, panels and pages from a manuscript | **Script gate**: no image generation before the panel count is known |
| [**ap-voice**](https://github.com/shouni/ap-voice) | Turns articles and documents into narration with chosen speakers | Script and audio are separate steps, so **synthesis can be redone without regenerating** |
| [**adk-review**](https://github.com/shouni/adk-review) | Has an AI agent review a Git diff | Three read-only tools let it **look outside the diff on its own**. Stops by count, not time |
| [**ap-music-poc**](https://github.com/shouni/ap-music-poc) (archived) | Music generation with Lyria (PoC) | Quality control simplified so the async skeleton is readable. Frozen for reference |

### 🔌 MCP servers

| | |
|---|---|
| **ap-mcp** (private) | A gateway that gathers five services' tools behind one endpoint. Descriptions live in `{service}/{tool}.md` rather than Go literals, annotations are declared rather than guessed, and **the confirm gate is the preview itself** (design in [applications.md](https://github.com/shouni/public-docs/blob/main/docs/applications.md)) |
| [**ap-mcp-slack**](https://github.com/shouni/ap-mcp-slack) | A Slack MCP that runs over stdio with no server. Which tools get registered changes with the credentials |

### 🐍 Python tools

Run locally on their own, not on Cloud Run. The existing assets for video and audio analysis are on the Python side.

| | |
|---|---|
| [**lyric-video-maker**](https://github.com/shouni/lyric-video-maker) | Analyses a song and **draws the music video from the audio, frame by frame** (kick, snare, bass and lyrics; beatmv). English songs get word-timed karaoke with a Japanese translation, Japanese songs an English one. Can also build karaoke MP4s from a keyframe ZIP ([article](https://zenn.dev/snknsk/articles/lyric-video-maker), in Japanese) |
| [**ap-audio-probe**](https://github.com/shouni/ap-audio-probe) | Checks whether a generated song **plays as its recipe instructs**. Measures after removing the backing with Demucs, with thresholds relative to each song. Musical quality is out of scope |

---

### 📝 Design decisions (articles in Japanese)

| Article | What it covers |
|---|---|
| [The whole personal-development ecosystem](https://zenn.dev/snknsk/articles/shouni-go-ecosystem-overview) | How the layers relate, and the criteria for splitting libraries |
| [Agents read outside the diff](https://zenn.dev/snknsk/articles/adk-go-agent-review-practice) | Rebuilding code review as an agent loop with ADK for Go |
| [An MCP server is more than an API wrapper](https://zenn.dev/snknsk/articles/mcp-server-is-not-api-wrapper) | The confirm gate, and why there are no preview-only tools |
| [No image generation before the panel count is known](https://zenn.dev/snknsk/articles/79fa0d1bbb57e1) | Splitting a process in two for cost |
| [One Cloud Run image, split into web and worker](https://zenn.dev/snknsk/articles/cloudrun-web-worker-split) | Switching the dependency graph and permissions by role |
| [A full app build for every model name change](https://zenn.dev/snknsk/articles/cloudrun-config-terraform-import) | Moving Cloud Run settings into Terraform |
| [The four pillars of NetArmor](https://zenn.dev/snknsk/articles/netarmor-four-pillars) | Why SSRF protection sits on the lowest layer |
| [No room for the AI to misread](https://zenn.dev/snknsk/articles/go-kagome-prompt-defense) | Transpiling prompts into a safe "reading" form |

All 17 articles are at [zenn.dev/snknsk](https://zenn.dev/snknsk).

---

<sub>The boundary map and conventions live in [public-docs](https://github.com/shouni/public-docs), usage in each repository's README, APIs on [pkg.go.dev](https://pkg.go.dev/search?q=github.com%2Fshouni), and internal design decisions in each repository's CLAUDE.md.</sub>
