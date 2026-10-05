# nexus-runner-main-nexus_runner.py
1. EXPLORE │ │ Scan folders recursively (depth 3) │ │ Detect project markers │ │ │ │ 2. DETECT │ │ Pick best project type via priority list │ │ Load package.json / Cargo.toml / pyproject.toml │ │ │ │ 3. PLAN │ │ Map install / build / start commands │ │ Detect language-specific entry points │ │ │ │ 4. EXECUTE │ │ Run install (blocking) │
# NEXUS RUNNER
[![License: NEXUS-OPEN-2.0](https://img.shields.io/badge/License-NEXUS--OPEN--2.0-blue.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-2.0.0-brightgreen.svg)]()
[![Python](https://img.shields.io/badge/python-3.8%2B-blue.svg)]()
[![Platform](https://img.shields.io/badge/platform-linux%20%7C%20macos%20%7C%20ios%20(a--Shell)-lightgrey.svg)]()
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![Zero deps](https://img.shields.io/badge/dependencies-stdlib%20only-success.svg)]()

> **Detect. Install. Build. Run. Any project. One command.**

NEXUS RUNNER is a single-file Python tool that scans your local folders, detects any project type (Node, Python, Rust, Go, Java, Ruby, PHP, Deno, Make, CMake, Docker), and runs the full pipeline: **install → build → start**. It ships with a local web UI on `localhost:8922` and a full CLI, with zero external dependencies.

Author: **Aissa Mohammedi (DGK)**
License: **NEXUS-OPEN-2.0**
Version: **2.0.0**

---

## Why NEXUS RUNNER

Most projects require you to remember:

- Whether to use `npm` / `pnpm` / `yarn` / `bun`
- Whether to run `poetry install` or `pip install -r requirements.txt`
- Whether the build step is `cargo build --release` or `make`
- Where the entry point is (`main.py`? `app.py`? `manage.py`?)

**NEXUS RUNNER does all of that automatically.** You point it at a folder. It detects the project. It shows you the exact commands it will run. You confirm. It runs.

No config file. No YAML. No Docker-compose. Just a Python file you drop into any machine.

---

## Features

| Feature | Description |
|---------|-------------|
| **Auto-detection** | Recognizes 11 project types through 21 marker files |
| **Multi-package-manager** | Handles npm, pnpm, yarn, bun, pip, poetry, pipenv, cargo, go mod, maven, gradle, bundler, composer |
| **Visual folder browser** | Explore your directories from a web UI, projects highlighted in green |
| **Web interface** | Beautiful dark UI on `localhost:8922` |
| **CLI mode** | Full command-line interface for scripts and automation |
| **Live process management** | Start processes in background, view logs, kill them |
| **Dry-run mode** | Preview what will run before anything executes |
| **Skip install / skip build** | Fast iteration when you only want to test the start step |
| **Zero dependencies** | Pure Python 3.8+ stdlib. No pip install. No venv. |
| **a-Shell iOS compatible** | Works on iPhone / iPad through a-Shell |

---

## Supported project types

| Type | Marker files | Install | Build | Start |
|------|-------------|---------|-------|-------|
| Node (npm) | `package.json` | `npm install` | `npm run build` | `npm start` / `npm run dev` |
| Node (pnpm) | `pnpm-lock.yaml` | `pnpm install` | `pnpm build` | `pnpm start` |
| Node (yarn) | `yarn.lock` | `yarn install` | `yarn build` | `yarn start` |
| Node (bun) | `bun.lockb` | `bun install` | `bun run build` | `bun start` |
| Deno | `deno.json` | — | — | `deno task start` |
| Python | `requirements.txt` | `pip3 install -r requirements.txt` | — | `python3 main.py` |
| Python (poetry) | `pyproject.toml` | `poetry install` | — | `poetry run python main.py` |
| Python (pipenv) | `Pipfile` | `pipenv install` | — | `pipenv run python main.py` |
| Rust | `Cargo.toml` | `cargo fetch` | `cargo build --release` | `cargo run` |
| Go | `go.mod` | `go mod download` | `go build ./...` | `go run .` |
| Java (maven) | `pom.xml` | `mvn dependency:resolve` | `mvn package -DskipTests` | `mvn spring-boot:run` |
| Java (gradle) | `build.gradle` | — | `./gradlew build -x test` | `./gradlew bootRun` |
| Ruby | `Gemfile` | `bundle install` | — | `bundle exec ruby main.rb` |
| PHP | `composer.json` | `composer install` | — | `php -S localhost:8000` |
| Make | `Makefile` | — | `make` | `make run` / `./a.out` |
| CMake | `CMakeLists.txt` | `cmake ..` | `cmake --build .` | `./app` |
| Docker | `docker-compose.yml` | — | `docker compose build` | `docker compose up` |

---

## Quick start

```bash
# 1. Download the script
curl -O https://raw.githubusercontent.com/Aissamohammedi88/nexus-runner/main/nexus_runner.py

# 2. Run it
python3 nexus_runner.py

# 3. Open the browser
# → http://localhost:8922/
```

That's it. No installation. No dependencies.

---

## Web interface

The web UI gives you:

- **Explorer tab** — Navigate your folders, projects highlighted in green
- **Process tab** — Live list of running processes, logs, kill buttons
- **System tab** — Detected tools (node, python, cargo, docker, etc.)

<p align="center">
<img src="assets/screenshot-explorer.png" alt="Explorer" width="800">
</p>

---

## CLI mode

NEXUS RUNNER also works entirely in the terminal.

### List projects in a folder

```bash
python3 nexus_runner.py ls ~/Projects
```

Output:

```
Racine : /Users/aissa/Projects

api-gateway [PROJET:node]
frontend [PROJET:node_pnpm]
ml-pipeline
data-loader [PROJET:python]
trainer [PROJET:python_poetry]
rust-service [PROJET:rust]
```

### Run a project

```bash
# Auto (install + build + start)
python3 nexus_runner.py run ~/Projects/api-gateway

# Dry-run (preview only)
python3 nexus_runner.py run ~/Projects/api-gateway dry

# Install + build, no start
python3 nexus_runner.py run ~/Projects/api-gateway install

# Start only (skip install + build)
python3 nexus_runner.py run ~/Projects/rust-service start --skip-install --skip-build
```

Sample output:

```
Type : rust
Files : Cargo.toml

Plan :
install : cargo fetch
build : cargo build --release
start : cargo run

[install] OK (rc=0)
[build] OK (rc=0)
[start] OK (rc=0)
-> proc_id=a3f5c8d91e7b pid=48293
```

---

## How it works

```
┌─────────────────────────────────────────────────────────────┐
│ NEXUS RUNNER │
├─────────────────────────────────────────────────────────────┤
│ │
│ 1. EXPLORE │
│ Scan folders recursively (depth 3) │
│ Detect project markers │
│ │
│ 2. DETECT │
│ Pick best project type via priority list │
│ Load package.json / Cargo.toml / pyproject.toml │
│ │
│ 3. PLAN │
│ Map install / build / start commands │
│ Detect language-specific entry points │
│ │
│ 4. EXECUTE │
│ Run install (blocking) │
│ Run build (blocking) │
│ Run start (background, PID tracked) │
│ │
│ 5. MANAGE │
│ Stream logs │
│ Kill processes │
│ Report status │
│ │
└─────────────────────────────────────────────────────────────┘
```

### Detection priority

When a folder contains multiple project markers (e.g., `package.json` + `Makefile` + `Dockerfile`), NEXUS RUNNER uses this priority:

```
node > node_pnpm > node_yarn > node_bun
> deno
> python_poetry > python_pipenv > python
> rust > go
> java_gradle > java_maven
> make > cmake
> ruby > php
> docker > docker_simple
```

So a folder with both `package.json` and `Makefile` will be treated as a Node project.

---

## Architecture

```
nexus_runner.py
├── Detection
│ ├── MARQUEURS[] — 21 marker files
│ ├── detecter_type() — main detection
│ └── lire_package_json() — parse Node metadata
│
├── Command mapping
│ └── commandes_pour() — install / build / start per type
│
├── Folder navigation
│ └── dossiers_adjoints() — recursive scan, depth 3
│
├── Execution
│ ├── lancer_commande() — blocking with timeout
│ ├── lancer_en_arriere_plan() — background Popen
│ ├── tuer_process() — terminate + kill
│ ├── lire_log_proc() — read stdout/stderr
│ └── statut_process() — is alive?
│
├── Pipeline
│ └── pipeline() — install → build → start
│
├── HTTP server
│ ├── Handler — routing
│ └── Server — ThreadingMixIn HTTPServer
│
└── CLI
├── cli_ls() — list mode
└── cli_run() — run mode
```

---

## Files generated

NEXUS RUNNER stores everything in `~/Documents/nexus_runner/`:

```
nexus_runner/
├── logs/
│ └── runner.log # Main log
└── procs/
├── <id>.log # Stdout per process
└── <id>.err # Stderr per process
```

Nothing is written outside this folder. Your projects stay untouched.

---

## Security

- **Localhost only** — the HTTP server binds to `127.0.0.1`, not `0.0.0.0`
- **Home-restricted** — the folder explorer cannot go outside `$HOME`
- **No shell injection** — commands are pre-defined, not user-supplied
- **Timeout protection** — install and build steps have configurable timeouts (default 600s)
- **No telemetry** — no data leaves your machine
- **No writes outside base** — the tool never touches your project files

---

## Configuration

### Change the port

```bash
# Edit in the source
PORT = 8922
```

### Change the navigation root

```bash
# Edit in the source
RACINE_DEFAUT = os.path.join(HOME, "Documents")
```

### Change the timeout

Timeouts are per-call parameters. In CLI mode, edit `pipeline()` call or extend the CLI parser.

---

## Roadmap

- [ ] Support for `.csproj` (C# / .NET)
- [ ] Support for `mix.exs` (Elixir)
- [ ] Support for `pubspec.yaml` (Flutter / Dart)
- [ ] WebSocket-based live log streaming
- [ ] Desktop notifications when a process crashes
- [ ] Project favorites / recent list
- [ ] Environment variable editor per project
- [ ] Docker auto-detection of `compose` vs `docker-compose`

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

Quick rules:

1. Zero external dependencies — stdlib only
2. Python 3.8+ compatibility
3. No `subprocess` without timeout
4. No shell injection vectors
5. One feature per PR
6. Tests for new detection types

---

## License

**NEXUS-OPEN-2.0** — see [LICENSE](LICENSE).

You are free to use, modify, and distribute this software. Attribution required.

---

## Author

**Aissa Mohammedi (DGK)**
Systems Architect · Quebec, Canada

- GitHub: [@Aissamohammedi88](https://github.com/Aissamohammedi88)
- LinkedIn: [linkedin.com/in/aissa-mohammedi-2308743b2](https://linkedin.com/in/aissa-mohammedi-2308743b2)

---

## Acknowledgments

- Inspired by the pain of `cd`-ing into 40 projects and trying to remember the exact command
- Built for developers who work on heterogeneous stacks
- No framework, no build system, no dependency hell

---

<p align="center">
<strong>NEXUS RUNNER</strong><br>
<em>Detect. Install. Build. Run.</em><br>
<sub>Built by Aissa Mohammedi (DGK) — NEXUS-OPEN-2.0</sub>
</p>
