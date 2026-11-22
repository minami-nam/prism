<!-- logo: create docs/media/prism-lockup.svg (+ dark variant), then uncomment this block -->
<!--
<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="docs/media/prism-lockup-dark.svg">
    <img src="docs/media/prism-lockup.svg" alt="Prism" width="340">
  </picture>
</p>
-->

<h1 align="center">Prism</h1>

<h3 align="center">Put several AIs on the same table — and make them check each other.</h3>

<p align="center">
  <a href="README.md">English</a> · <a href="README.ko.md">한국어</a>
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/github/license/minami-nam/prism?style=flat" alt="License: Apache-2.0"></a>
  <img src="https://img.shields.io/badge/python-3.11+-blue?style=flat" alt="Python">
  <img src="https://img.shields.io/badge/UI-React-61dafb?style=flat" alt="React">
  <img src="https://img.shields.io/badge/DB-SQLite-003b57?style=flat" alt="SQLite">
</p>

**Prism is a personal workbench where several AI agents review the same target and are held
to what real tools report.** Independent review → group and vote on findings → verify with
real tools → debate → a human approves each patch.

It also keeps your own notes as searchable knowledge, exposes spreadsheets and datasets to
queries, and lets agents act on your own machines through MCP — all on one instance you run
yourself.

<p align="center">
  <a href="#cross-validation">Cross-validation</a> · <a href="#solo--multi">Solo &amp; Multi</a> ·
  <a href="#knowledge">Knowledge</a> · <a href="#personas">Personas</a> · <a href="#tools">Tools</a> ·
  <a href="#architecture">Architecture</a> · <a href="#quick-start">Quick start</a>
</p>

<!-- screenshot: the single most effective asset. Capture a finished run report and replace
     this block with: <p align="center"><img src="docs/media/screenshot-run.png" width="820"></p> -->

---

## Why

An AI review that says "this looks wrong" and a tool that actually ran the code are not the
same kind of statement. Prism keeps them **separately recorded**, so a claim either has an
execution behind it or it visibly does not.

## Features

| Area | What you can do |
| --- | --- |
| **Solo chat** | Pick provider, model and effort **per message** — switch from a local model to a hosted one mid-conversation |
| **Multi chat** | Agents draft, critique each other's drafts, then one synthesis is produced; a different persona per agent |
| **Knowledge** | Markdown notes are the source of truth; a local model extracts entities and relations into a graph, relevant sections get attached to answers |
| **Tables & data** | Spreadsheets (xlsx) and public datasets (SQLite) are queryable through tools during an answer |
| **Tools (MCP)** | Read and write notes, read project code, web search and browser, local files and commands, smart-home devices |
| **Personas** | Reusable presets for tone, judgement criteria, notes to always read, and which knowledge collections an answer may use |
| **Audit trail** | Every answer shows which tools were actually executed, whether each call was approved, and any error |



## Solo & Multi
### Solo — one AI, one conversation. The provider is chosen above the input box.

Provider	Notes
local	an OpenAI-compatible llama-server you run yourself. Model and thinking on/off are set per message
claude / codex / antigravity	hosted coding agents. Model and effort per message. They consume your existing subscription, no separate API billing
internet	toggled per conversation. When on, the agent reads the pages it finds, not just search snippets
Long conversations are summarised automatically so a session can keep going past the context
window. Images and files can be attached to a message.

## Multi — several agents answer the same question, critique each other's drafts, then one
agent synthesises the result. Giving each agent a different persona puts a strict reviewer and
a pragmatic reviewer on the same question.

Tool execution logs are rendered under every answer: tool name, approval state, errors.
If no log appears, no tool was used. "I ran it" becomes something you can check instead
of something you have to trust.

## Knowledge
notes/ Markdown is the original. One topic per note, indexed per ## section
a local model turns sections into entities and relations, an embedding model makes them
retrievable. Answers cite the sections they used
relations are stored as sentences — top instantiates fifo, handling department: Civil Affairs Division — so the graph stays readable
the graph is editable: entities and relations can be added, changed and deleted directly
ambiguous merges, reclassifications and relations go to a review queue instead of being
silently applied
notes, tables, data and conversations are searchable from one box (Ctrl+K)
crossval backup snapshots notes, the database and configuration
Every knowledge section carries a confidence marker, so the reader knows what they are being
asked to trust:

## Marker	Meaning
[OBSERVED]	measured or confirmed inside Prism
[USER_CONFIRMED]	reported by the owner, not externally verified
[INFERRED]	estimate or interpretation
[REPORTED]	a claim, not re-verified when written
[OPEN]	unresolved inconsistency
## Personas
A persona is a Markdown preset that sets tone, judgement criteria, notes to always read, and
which knowledge collections an answer may touch. A review persona limited to code knowledge and
a daily-assistant persona limited to personal notes can answer the same question with different
boundaries. Personas apply per agent in Multi conversations.

## Tools
MCP servers are wired in per server, and each tool is set to read (runs immediately),
write (requires approval) or off.

## Server	Purpose
notes	search, read, create and edit knowledge notes — writes go through an approval card
code	read project files, list validation targets, read the design document
web / browser	search the web and read the result pages
local	read and write files, run commands

## Architecture

graph LR
  UI[React UI] --> API[FastAPI API + SSE]
  API --> WK[worker]
  WK --> AG[agents: local / Claude / Codex / Antigravity]
  WK --> ORC[oracles: Verilator Yosys ruff pytest mypy]
  WK --> DB[(SQLite)]
  N[notes/ Markdown] --> WK
  WK --> LLM[OpenAI-compatible llama-server]
  WK --> MCP[MCP servers]
web serves the API, SSE stream and UI; it does not run validations itself
worker runs validation runs, chat answers and knowledge collection
knowledge extraction and embedding run against OpenAI-compatible llama-server endpoints
local models are optional for chat and tools; retrieval and graph extraction need them
Quick start
Personal tool, run from source — there is no packaged release or installer.

## How to Start
```
git clone https://github.com/minami-nam/prism.git prism
cd prism
python -m venv .venv
.venv/bin/pip install -r requirements.txt   # TODO: requirements.txt is not published yet
```


# TODO: web server command
Configuration lives outside the repository. Provider keys, MCP tokens and model endpoints belong
in local files and are never committed.

# Roadmap
In use — RTL and Python cross-validation, Solo and Multi conversations, knowledge graph and
retrieval, MCP tool wiring, personas, backup.

Not yet — packaged release and installer, published dependency list, multi-user isolation,
full documentation.

# License
Apache License 2.0 — see LICENSE.

<details> <summary><b>한국어 요약</b></summary>
Prism은 여러 AI agent가 같은 대상(RTL·Python)을 독립적으로 검토하고, 실제 도구(Verilator·Yosys·
ruff·pytest·mypy)가 보고한 결과와 맞추도록 만드는 개인 작업 도구입니다. 지적을 묶고 투표한 뒤
도구로 확인하고, 최종 수정안만 사람이 승인합니다. Markdown 노트에서 개체·관계를 뽑아 graph로
만들고 답변에 관련 절을 붙이며, MCP로 로컬 파일·웹·집 기기를 도구로 씁니다. 모든 답변 아래에는
실제로 실행한 도구 기록이 그대로 남습니다.

</details> ```
