# PulseThread SIS (Community Edition)

PulseThread SIS is a **synchronous-first execution and coordination framework**
for Minecraft servers.

It is designed to be integrated into by other plugins to safely stage,
govern, and execute heavy or disruptive workloads without overwhelming
the main thread or region threads.

PulseThread SIS is not a one-click optimizer and not a universal async engine.
It provides a shared execution and policy layer so cooperative plugins operate
under consistent rules instead of competing independently for server time.

PulseThread SIS is currently in active development. Behavior, defaults, and
limits may continue to evolve as additional real-world testing is performed.

---

## Repository Purpose

This repository contains **public reference material** for the
**PulseThread SIS Community Edition**.

It exists to document:
- command behavior for server operators
- public API surfaces and execution model for developers
- architectural constraints and guarantees
- Community Edition licensing terms

This repository is **not** the full source repository and **does not**
contain buildable or complete implementation code.

---

## Repository Contents

The following files are authoritative:

- **README.md**  
  High-level overview and repository scope

- **COMMANDS.txt**  
  Operator-facing command reference for PulseThread SIS

- **NOTES.md**  
  Public API notes, execution model, architectural constraints,
  and internal behavior documentation

- **SCHEDULE_A_COMMUNITY.txt**  
  Community Edition license terms and usage restrictions

Reference code snippets may appear in documentation files, but no
complete plugin implementations or build artifacts are provided here.

---

## Software Distribution

PulseThread SIS binaries are distributed separately via official
release channels (for example, Modrinth).

Refer to the distribution source for:
- installation instructions
- runtime configuration
- plugin JAR downloads

---

## Audience

This repository is intended for:
- server operators reviewing commands and behavior
- plugin developers evaluating integration with PulseThread
- reviewers assessing architectural and safety constraints

It is not intended as a quickstart or end-user installation guide.

---

## Legal

PulseThread SIS is authored and maintained by
**PerfectPriceProjectsLLC**, distributed under the trade name **WhiskeyMC**.

Use of the software is governed exclusively by the applicable license
terms and schedules included in this repository and with the software
distribution.

See `NOTICE` for copyright and trademark information.
