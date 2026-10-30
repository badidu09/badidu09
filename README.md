<img src="assets/hero.svg" width="100%" alt="Vincent Ballandi, game server and backend developer" />

<p align="center">
  <a href="#bpw-pvp"><img src="assets/tile-bpw.svg" width="65%" alt="BPW PvP: 31K+ lines, 29 modules, hand-written MySQL client" /></a>
  <a href="#envaudit"><img src="assets/tile-envaudit.svg" width="33.7%" alt="envaudit: zero-dependency CLI that keeps env files in sync" /></a>
</p>
<p align="center">
  <a href="#sudoku-vision"><img src="assets/tile-sudoku.svg" width="33.7%" alt="Sudoku Vision: 97.6% of sudokus solved on unseen photos" /></a>
  <a href="#14k-pvp"><img src="assets/tile-14k.svg" width="65%" alt="14K PVP: virtual economy with transactions and idempotency" /></a>
</p>
<p align="center">
  <a href="#action_logger"><img src="assets/tile-logger.svg" width="65%" alt="action_logger: an audit pipeline that survives outages" /></a>
  <a href="#theory-i-actually-use"><img src="assets/tile-theory.svg" width="33.7%" alt="Theory I actually use" /></a>
</p>

I like understanding what sits under the abstraction, then building it myself when that is the better call. My main playground is **multiplayer game servers on FiveM**: real players, real concurrency, and clients that cannot be trusted. At **Epitech**, I work on the theory side: computer vision, machine learning, probability and statistics.

<sub>↑ tap a card to jump to the project · ▸ open a section for the technical deep dive</sub>

---

## bpw pvp

**A competitive game server built from scratch.** Most FiveM servers assemble an existing framework (ESX, QBCore) with third-party scripts. I wrote my own instead, so that every layer is understood, and nothing runs that the game mode does not need.

`~31,000 lines of Lua, JS, CSS and SQL` `29 gameplay modules` `no game framework` `no npm dependency in the database layer`

<details open>
<summary><b>▸ architecture & what makes it interesting</b></summary>
<br/>

```mermaid
flowchart LR
    subgraph Client["Game client (untrusted)"]
        UI["Module UIs<br/>HTML / CSS / JS"]
        CL["Client modules<br/>Lua"]
    end
    subgraph Server["Game server (authoritative)"]
        SV["Gameplay modules<br/>match · ranked · lobby · shop · admin"]
        CORE["Core<br/>config · shared lib · players · bans"]
        DRV["MySQL client written from scratch<br/>binary protocol · auth · pool"]
    end
    DB[("MariaDB / MySQL")]
    UI <--> CL
    CL <-->|events| SV
    SV --> CORE --> DRV -->|TCP| DB
```

- **A MySQL client written by hand, on raw TCP (688 lines).** It implements the wire protocol directly from the specification using only Node's `net` and `crypto`: handshake v10, binary packet parsing, multiple result sets, connection pooling and client-side parameter escaping.
- **Real authentication, not just queries.** It supports `mysql_native_password` and `caching_sha2_password`, including the fast path and the full RSA exchange, so it works against both MySQL and MariaDB.
- **A dependency-ordered module system.** Each feature is a self-contained module that declares its dependencies and optional UI. The boot sequence starts modules in dependency order, so adding a feature does not mean touching the core.
- **Matchmaking as a state machine.** Lobbies move through `IDLE → QUEUE → FOUND → MATCH`, with a ready check and a return path if someone declines. Queued lobbies are assembled into complete teams for ELO-ranked 4v4, and players can also queue for 1v1 duels.
- **Don't trust the client.** Sensitive actions are checked on the server. For example, a client reporting a death it did not suffer during a ranked match gets logged as a cheating attempt.
- **Versioned schema.** Database changes go through numbered SQL migrations applied by the core.

The same architecture now also powers **Five Battle Royal**, a 5v5 battle royale on Cayo Perico, which shows the design holds up beyond a single game mode.

</details>

## envaudit

**A zero-dependency CLI that keeps your code, `.env.example` and `.env` in sync, and catches secrets before they leak.** It reads your source code, compares it with your env files, and tells you what drifted, without ever printing the values of your variables.

[**→ view the repo**](https://github.com/badidu09/envaudit) · open source (MIT)

```sh
npx github:badidu09/envaudit
```

<details>
<summary><b>▸ what it does & how it is built</b></summary>
<br/>

- **Six rules, split by severity.** Errors: `undocumented`, `secret`, `exposed-env`. Warnings: `unused`, `missing-local`, `duplicate`.
- **Works across stacks.** JavaScript, TypeScript, Vue, Svelte, Astro, Python, Go, Rust, Ruby, PHP, Java, Kotlin, C# and `docker-compose` files.
- **Secret detection.** Known formats (Stripe, GitHub, GitLab, Slack, AWS, Google and more), private keys, JWTs, URLs with embedded passwords, and high-entropy values in sensitive variables, while ignoring placeholders and local URLs.
- **Made for automation.** A GitHub Action, `--json` output, `--fix` to document missing variables, and clear exit codes (`0` clean, `1` problems, `2` invalid usage).
- **Honest about its limits.** It matches source text rather than executing it, and the README documents what that cannot catch, such as dynamic reads like `process.env[name]`.

</details>

## sudoku vision

**From a phone photo to a solved grid.** Team project with three people at Epitech. **My role: models and data**, meaning the datasets, YOLO training and experiment tracking in MLflow.

`97.6% sudokus solved (40/41 unseen photos)` `99.51% accuracy on 405 real cells` `0% wrong solutions on the test set` `0.16 s per photo`

<details>
<summary><b>▸ pipeline & method</b></summary>
<br/>

<p>
  <img src="https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white" alt="Python" />
  <img src="https://img.shields.io/badge/YOLO11-111F68?style=flat-square&logo=ultralytics&logoColor=white" alt="YOLO11" />
  <img src="https://img.shields.io/badge/OpenCV-5C3EE8?style=flat-square&logo=opencv&logoColor=white" alt="OpenCV" />
  <img src="https://img.shields.io/badge/MLflow-0194E2?style=flat-square&logo=mlflow&logoColor=white" alt="MLflow" />
  <img src="https://img.shields.io/badge/FastAPI-009688?style=flat-square&logo=fastapi&logoColor=white" alt="FastAPI" />
  <img src="https://img.shields.io/badge/Streamlit-FF4B4B?style=flat-square&logo=streamlit&logoColor=white" alt="Streamlit" />
</p>

```mermaid
flowchart LR
    A[Photo] --> B[YOLO<br/>grid detection]
    B --> C[OpenCV<br/>homography and warp]
    C --> D[81 cells]
    D --> E[YOLO<br/>cell classifier]
    E --> F[Grid validation<br/>and repair]
    F --> G[Backtracking<br/>with MRV]
    G --> H[Solution drawn<br/>on the photo]
```

**The method matters more than the score:**

- **Data leakage prevented.** The photos include many shots of the same grid, so a naive split would inflate the results. Splits are made per grid, with an automatic check that no grid appears in two sets.
- **Models trained on the real failure modes.** Synthetic cells simulate imperfect warps, leftover grid lines and show-through from the back of the page. Augmentations that would change a digit's meaning are turned off.
- **99.5% per cell isn't enough.** Over 81 cells, small errors add up to a wrong grid. The sudoku rules therefore act as a quality check: when a grid is invalid, the least confident cells get repaired instead of a wrong answer being returned.
- **Every pipeline stage measured.** An ablation study quantifies what each stage adds, taking the solve rate from 85.4% with OpenCV alone to 97.6% with the full pipeline.

</details>

## 14k pvp

**A virtual economy that has to stay correct.** A case-opening website whose database is shared with a live game server. It handles virtual currency, so race conditions and double spending are the real problems to solve.

<details>
<summary><b>▸ problems & solutions</b></summary>
<br/>

<p>
  <img src="https://img.shields.io/badge/Next.js_16-000000?style=flat-square&logo=nextdotjs&logoColor=white" alt="Next.js 16" />
  <img src="https://img.shields.io/badge/React_19-20232A?style=flat-square&logo=react&logoColor=white" alt="React 19" />
  <img src="https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white" alt="TypeScript" />
  <img src="https://img.shields.io/badge/Drizzle_ORM-4B7A00?style=flat-square&logo=drizzle&logoColor=white" alt="Drizzle ORM" />
  <img src="https://img.shields.io/badge/MariaDB-003545?style=flat-square&logo=mariadb&logoColor=white" alt="MariaDB" />
</p>

| Problem | Solution |
|---|---|
| Two simultaneous requests spend the same balance | Database transactions with row-level locking (`SELECT … FOR UPDATE`) on the wallet |
| A deposit from the game is delivered twice | Idempotent operations keyed on a unique `ref` |
| Balance history must be auditable | Append-only ledger of every currency movement |
| Draw results must not be predictable | Weighted draw on the server with a CSPRNG (`crypto.randomInt`) |
| Website and game must not corrupt each other | The site never writes to game tables; all exchanges go through a documented API |

</details>

## action_logger

**An audit pipeline that survives outages.** A full logging platform for game servers, from the Lua resource to the web dashboard. Node / Express · SQLite · React · Discord OAuth2.

<details>
<summary><b>▸ how it holds up</b></summary>
<br/>

- **Built for failure:** bounded queue, batching and retries with exponential backoff when the API is unavailable.
- **Secure by design:** the API key exists only on the server, and everything sent from a client is validated again and rate limited.
- **Framework-agnostic:** works with ESX, QBCore, Qbox or standalone servers.
- **Usable:** role-based access, filters, per-player timeline, statistics and a live feed.

</details>

## also on fivem

- **[bd_uno](https://github.com/badidu09/bd_uno)** — a multiplayer UNO table in the game world, with a full ruleset and several concurrent tables. *(open source)*
- **Large-scale map streaming** — a sky arena of 2.2 M triangles split into 92 tiles, with simplified collision meshes and chained texture dictionaries.
- **Skydrop** — a floating battle-royale lobby with a scripted drop sequence, 1,000 m above Cayo Perico.
- **nomercy_climb** — ledge detection using raycasts: checks surface angle, flatness and clearance before climbing.
- **Release tooling** — a Python build script for Cfx Asset Escrow that strips secrets and sources from the shipped package.

---

## theory i actually use

| Field | Concepts | Where |
|---|---|---|
| **Networking & protocols** | TCP, binary framing, handshakes, client-server authority | MySQL client, game server |
| **Cryptography** | Hash-based challenge-response, RSA, CSPRNG | MySQL auth, draw system |
| **Databases** | ACID transactions, row locking, idempotency, migrations | 14K PVP, BPW core |
| **Distributed reliability** | Retries, exponential backoff, batching, bounded queues | action_logger |
| **Algorithms** | Backtracking, MRV heuristic, state machines, dependency ordering | Sudoku solver, matchmaking, module boot |
| **Static analysis** | Pattern-based source scanning, secret detection, entropy heuristics | envaudit |
| **Machine learning** | Object detection, classification, data leakage, augmentation, mAP, ablation studies | Sudoku Vision |
| **Computer vision** | Homography, perspective transforms, contour analysis, interpolation | Sudoku Vision |
| **Probability & statistics** | Probability laws, regression, hypothesis testing, confidence intervals | [Epitech math projects](https://github.com/badidu09/MATH_PROJECT): all validated, 16/20 average |

## tools

<p align="center">
  <img src="https://skillicons.dev/icons?i=lua,ts,js,py,c,cpp,haskell&theme=dark" /><br/>
  <img src="https://skillicons.dev/icons?i=nextjs,react,nodejs,express,tailwind,opencv&theme=dark" /><br/>
  <img src="https://skillicons.dev/icons?i=mysql,sqlite,mongodb,nginx,linux,git,blender&theme=dark" />
</p>

<p align="center">
  <img width="100%" src="https://ghchart.rshah.org/7C3AED/badidu09" alt="GitHub contributions" />
</p>

<p align="center">
  <sub>Most of my game server code lives in private repositories. Happy to walk through it in an interview.</sub><br/><br/>
  <img src="https://img.shields.io/badge/Discord-badi______-5865F2?style=for-the-badge&logo=discord&logoColor=white" alt="Discord: badi___" />
</p>
