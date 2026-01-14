# PulseThread SIS (Community Edition)

PulseThread SIS is a **synchronous-first execution and coordination framework**
for Minecraft servers.

It exists to solve a specific problem:  
when multiple heavy systems (AI, generation, structures, datapacks, physics)
compete for main-thread or region-thread time, performance collapses in
unpredictable ways.

PulseThread provides a **shared execution pipeline** that cooperative plugins
stage work into, instead of each plugin trying to manage load independently.

This is not a one-click optimizer and not a universal async engine.

PulseThread SIS is currently in active development. Behavior, defaults, and
limits may continue to evolve as additional real-world testing is performed.

---

## What PulseThread Does

PulseThread centralizes:
- task staging
- batching
- rate limiting
- fairness
- safety enforcement
- recovery under pressure

Instead of plugins deciding *when* to run heavy work, PulseThread decides
**if and how much** work may run at any given moment.

At runtime, the system continuously answers:

> How much work can safely execute right now without harming gameplay?

---

## Execution Model (Sync-First)

PulseThread is intentionally **synchronous-first**.

Heavy computation is performed asynchronously, but **all world mutation is
applied synchronously or on region-safe threads**, routed through a single
execution authority.

There are no unsafe async Bukkit mutation paths.

---

## Architecture Overview

PulseThread SIS is structured as a strict pipeline:

- **PulseHook** — discovery and staging only  
- **PulseRuntime** — policy and safety gating  
- **PulseThreadCore** — execution authority and throughput control  

Responsibilities are intentionally separated and not collapsed.

---

## Constraints (By Design)

PulseThread enforces hard safety boundaries:

- No async world mutation
- No collapsing of Hook, Runtime, or Core roles
- No unsafe Folia region bypass
- No unbounded backlog drain
- No forced execution under unsafe conditions

These constraints are enforced automatically by runtime gates and governor logic.
Plugin integrations declare intent; PulseThread decides if and when execution is
permitted.

---

## Repository Scope

This repository contains **public reference material** for the
PulseThread SIS Community Edition, including:

- Command reference (`COMMANDS.txt`)
- Public API and integration notes
- Licensing terms and schedules
- Legal notices

No source code, compiled binaries, or build artifacts are provided here.

PulseThread SIS software is distributed separately via official release channels.

---

## Notes for Developers

This project is intentionally constrained and architecture-locked.

Developers are encouraged to:
- review the execution model
- evaluate the constraints
- test integrations
- provide feedback after real-world usage

Internal design notes and evolving project state are documented in:

- `NOTES.md`

---

## Legal

PulseThread SIS is authored and maintained by
**PerfectPriceProjectsLLC**, distributed under the trade name **WhiskeyMC**.

Use of the software is governed by the applicable license terms and schedules
included in this repository and with the software distribution.

See `NOTICE` for copyright and trademark information.
