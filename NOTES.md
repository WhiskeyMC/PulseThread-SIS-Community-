**PulseThread SIS (Community)**

PulseThread SIS is a workload coordination and execution framework for Minecraft servers. It is designed to be hooked into by other plugins to safely stage, govern, and execute heavy or disruptive work without overwhelming the main thread or region threads.

Rather than acting as a standalone optimizer, PulseThread provides a shared execution and policy layer that cooperative plugins can integrate with. It centralizes scheduling, batching, rate limiting, and safety enforcement so that participating plugins operate under consistent rules instead of competing independently for server time.

PulseThread SIS is designed as a coordinated system rather than a single plugin. Most real functionality is delivered through companion plugins that integrate into the shared execution pipeline. This allows complex or expensive systems to operate under a unified execution model instead of fragmenting server resources.


**SIS Plugin Architecture Overview**

PulseThread SIS is composed of a core execution pipeline and a set of optional domain modules. Not all modules are included in every edition. Availability, limits, and behavior are governed by the applicable license Schedule A.

At a high level, all editions share the same execution and safety foundation. Higher editions primarily expand limits, policy flexibility, and optional modules rather than changing core behavior.


**Core Runtime Plugins (All Editions)**

These plugins form the mandatory execution pipeline and are always present in operational deployments:

PulseThreadCore  
Final execution authority. Owns thread pools, batching, rate limiting, governor modes, apply queues, watchdogs, and operator visibility.

PulseRuntime  
Policy and safety enforcement layer. Validates task intent, world state, MSPT signals, and reverse-async safety before execution is permitted.

PulseHook  
Discovery and staging layer. Integrates with cooperative plugins, enforces quotas and ownership, and stages work into Runtime without executing it directly.


**Optional SIS Modules (Edition-Dependent)**

The following companion plugins extend the core runtime and are enabled based on license tier and deployment requirements:


**Community / Pro (limited or policy-gated in Community)**

PulseGen  
Offloading and coordination for generation-heavy workloads, structure placement, and chunk-adjacent computation.

PulseAI  
Entity, AI, and behavior task coordination with batching, rate limiting, and governor-controlled throughput.

PulseAI stages entity processing, perception, and ray-based queries as first-class workload units within the shared execution pipeline. AI and perception workloads are validated by Runtime, batched and rate-limited by Core, and applied synchronously under normal execution gates. This avoids ad-hoc AI execution paths and ensures consistent behavior under mixed load.


**Client-Side Execution Modules (Optional)**

PulseFabric Client  
Standalone Fabric client-side execution surface for PulseThread SIS.

PulseFabric Client provides an optional client execution and offload surface that allows PulseThread SIS servers to delegate selected perception and analysis workloads to participating clients. The client acts purely as an execution target and never makes authoritative gameplay decisions.

Client-side computation is advisory and fully server-controlled. The server advertises capabilities, issues bounded work requests, validates returned results, and may fall back to server-side execution at any time. Absence or failure of client participation does not impact correctness.

PulseFabric Client is designed to integrate into the same shared execution and policy model as server-side modules. Ray tracing and perception workloads routed to clients are treated as first-class workload units, governed by the same batching, rate limiting, and safety guarantees enforced by Runtime and Core.

Installation of the Fabric client is optional. When no compatible server requests offloaded work, the client remains idle and does not alter gameplay.


**Enterprise Only**

PulseWorld  
World-level health, lifecycle control, and integrity management, including recovery and enforcement mechanisms.

PulseHealth  
Long-term performance analytics, pressure forecasting, and predictive health signals.

Advanced Policy Extensions  
Custom governors, override hooks, and integration allowances beyond public configurations.

Not all modules listed above are included in the Community edition. Redistribution, commercial use, and module availability are governed by the applicable license and Schedule A.

**Editions**

Community Edition

The Community edition provides the full core architecture and safety model of PulseThread SIS for non-commercial use. It includes the execution framework, governor logic, safety gates, and the companion modules required for real operation, with conservative limits and policy defaults.

This edition is intended for evaluation, learning, and non-monetized servers.

Pro Edition

The Pro edition expands on the Community foundation with support for monetized and commercial servers. It primarily increases operational allowances, tuning flexibility, and sustained throughput limits without changing the underlying execution model.

Enterprise Edition

Enterprise deployments are handled on a per-organization basis. Enterprise licensing may include expanded modules, higher limits, custom integration allowances, and support agreements.

Enterprise terms are negotiated individually and are not publicly documented at this time.



**PulseThread — Technical Overview (v9.0)**

Status: FEATURE-COMPLETE / ARCHITECTURE-LOCKED  
Phase: ALPHA 9.0  
Target: Publish-ready pending final security + documentation pass

Note: Core architecture and governor systems were finalized in v6.0 and carried forward unchanged into v9.0.

Version Context (v9.0–v9.2)

Version 9.0 represents the architecture lock point for PulseThread SIS. All core execution semantics, safety guarantees, and responsibility boundaries were finalized prior to this release and carried forward unchanged.

Version 9.1 existed as an internal stabilization and consolidation pass. Work during this period focused on cache correctness, lockout behavior, safety hardening, and cleanup of partially wired execution paths. No new execution capabilities were introduced, and this version was not treated as a public milestone.

Version 9.2 completes the remaining planned feature surface area on top of the locked v9.0 architecture. Changes in this release are the result of feature activation, wiring, and integration rather than architectural change. Behavioral differences reflect improved signal quality, reduced latency, and more complete utilization of the existing execution pipeline.

**Executive Summary**

PulseThread is a performance, stability, and fairness control layer for Minecraft servers.

It provides a shared execution framework that cooperative plugins can offload work into, allowing heavy or disruptive tasks to be staged, governed, and executed safely under a unified policy model rather than competing directly on the main thread or region threads.

It provides:
- A controlled execution gateway for heavy workloads
- Adaptive backpressure to prevent runaway load
- Fairness across competing sources of work
- Policy-driven recovery under stress
- Deterministic operator control over throughput behavior

PulseThread is explicitly designed to be offloaded into through its Hook and Runtime integration layers. Cooperative plugins may stage work into the shared execution pipeline, where it is validated, batched, rate-limited, and executed according to active governor and safety policies.

PulseThread is not a magic compatibility layer and does not attempt to fix non-cooperative plugins or unsafe direct world mutation. It focuses on safe control and coordination of cooperative workloads and server-side pressure management, preserving correctness and predictability under load.

This document reflects the consolidated implementation carried forward into v9.0 and extended through subsequent feature completion.


**Design Goals**

Primary goals
- Reduce main-thread and region-thread pressure safely
- Enable cooperative plugins to offload heavy work into a shared execution pipeline
- Maintain predictable player experience under load
- Prevent backlog drain storms and cascading stalls
- Provide clear policy knobs instead of hard-coded behavior
- Recover automatically after extreme spikes

Non-goals
- Universal async conversion of Bukkit API calls
- Automatic correction of unsafe third-party behavior
- Forcing all plugins to become Folia-compatible
- Bypassing platform safety or thread ownership rules


Architecture (Locked — Do Not Collapse)

Pipeline:  
PulseHook → PulseRuntime → PulseThreadCore

This separation is intentional:

- Hook discovers and stages work  
- Runtime enforces safety and policy gates  
- Core executes and governs throughput  

No responsibilities are collapsed.  
No async Bukkit mutation paths are introduced.

PulseHook — Discovery and Staging

Role
- Discovers cooperative plugins and provides integration points
- Enforces ownership, quotas, and opt-in boundaries
- Stages work into Runtime (never executes directly)
- Exposes integration points for perception, ray tracing, and analysis workloads without executing them

Key constraints
- Maintains lightweight caches only (UUIDs, coordinates, identifiers)
- No hard references to Chunk, Block, or World objects
- Must remain execution-free to preserve isolation
- Does not perform ray tracing or perception work directly


PulseRuntime — Policy and Safety Gates

Role
- Policy layer that decides whether work may proceed
- Performs safety validation before execution is allowed
- Acts as the authoritative gate for perception and ray-based workloads

Owns
- Task intent classification (ENTITY, STRUCTURE, READ, WRITE, PHYSICS, TNT, RAY, PERCEPTION)
- World lifecycle gating (unload, write-deny, TTL drop)
- Reverse-async enforcement rules
- MSPT sampling (authoritative signal source)
- Validation of ray tracing and perception workloads before execution or apply

Does not
- Own thread pools
- Execute tasks
- Perform ray tracing itself

Runtime answers:
Is this work safe, relevant, and valid right now?


PulseThreadCore — Authority and Execution

Role
- Final authority that executes work and governs throughput
- Executes validated workloads, including ray tracing and perception tasks, under active policy

Owns
- CPU, IO, efficiency, and virtual thread pools
- Batching and apply queues with fairness routing
- Global and per-world rate limiting
- Governor modes and dynamic budgets
- Watchdog, metrics, and operator visibility
- Execution of ray tracing and perception workloads as first-class task types

Core answers:
How much work should execute right now, where should it run, and under what budget?


**Core Features (Implemented / Stable)**

Dynamic Thread Pools

CPU Pool

- Chunk scanning  
- Computation-heavy tasks  
- Entity AI processing  

IO Pool

- Async structure placement and file IO  
- Schematics  
- Corrupted chunk reloads (where safe)  

Efficiency Pool

- Optional  
- Activated only by governor or explicit config  
- Used for low-priority background work and cache acceleration  

Virtual Thread Pool

- Short-lived lightweight tasks  
- Never blocks CPU pool  

Additional Core Capabilities

- Hardware-aware thread sizing based on available CPU resources
- Platform-aware execution paths for both Paper and Folia
- Safe routing to main-thread or region-thread apply paths as required
- No unsafe async world mutation paths introduced


**Additional Systems Completed (v9.1–v9.2)**

The v9.1–v9.2 series completes the remaining planned feature surface on top of the locked v9.0 architecture. Changes during this phase focus on correctness, consistency, and full utilization of the existing execution pipeline rather than architectural expansion.

Completed systems and improvements include:

- More consistent and deterministic Hook → Runtime → Core task flow
- Reduced internal latency between compute and apply stages
- Entity and perception workloads fully governed by batching, rate limiting, and governor policy
- Ray tracing treated as a first-class workload unit within the execution pipeline
- Improved signal consistency and timing for AI and perception-driven decisions
- Cleaner lockout behavior, staleness handling, and cache lifecycle enforcement
- Client ray tracing protocol scaffolding with full server-side validation paths
- Explicit separation between advisory client results and authoritative server apply logic
  

Governor-Driven Dynamic Batching and Apply Budgeting

Batching is governed entirely by active governor policy. Each governor mode defines a baseline throughput intent, and Runtime signals are used to dynamically scale execution within those bounds.

Signals consumed
- MSPT (fast and slow EMA)
- Apply backlog pressure
- CPU availability

Apply batching
- Main-thread apply batch size is determined each tick by the active governor
- Apply budgetMs is strictly enforced at all times
- No adaptive or self-tuning logic exists inside the apply pump

Async batching
- Governor policy supplies the async batch size
- BatchManager executes batches without embedding policy or heuristics

By centralizing all batching decisions in the governor, conflicting heuristics are eliminated and execution behavior remains predictable, observable, and operator-controlled.

Governor Modes (Finalized)

PulseThread execution behavior is governed by explicit governor modes. Governor modes define baseline throughput intent, safety posture, and how aggressively available headroom may be consumed. Mode selection is operator-controlled and enforced uniformly across the execution pipeline.

SAFE
- Prioritizes stability and player experience above throughput
- Aggressively limits batching and concurrency
- Aborts or suppresses non-critical workloads under pressure
- Used automatically during severe MSPT degradation or recovery

NORMAL
- Balanced execution mode intended for typical operation
- Maintains steady throughput while respecting safety margins
- Scales batching and concurrency conservatively based on Runtime signals

AGGRESSIVE
- Throughput-oriented mode for high-capacity environments
- Allows higher batching and concurrency within defined safety bounds
- Still fully governed by Runtime validation and MSPT feedback
- Does not bypass safety gates or authority checks


TURBO (Per-World Burst Mode — Not a Governor Mode)

TURBO is a temporary, per-world burst mode rather than a persistent governor mode.

- Per-world, time-limited execution burst
- Credit-based and non-spammable
- Scales with available CPU capacity
- Aborts immediately if SAFE is entered
- Biases world scheduling and batching toward the target world
- Redirects unused execution headroom into cache warming and background work

BOOST remains a legacy alias for TURBO.


**PulsePolicy Integration**

Governor modes define baseline execution intent, but final behavior is always constrained by PulsePolicy.

PulsePolicy provides higher-level policy enforcement that may:
- Restrict or override governor behavior under specific conditions
- Enforce hard safety limits regardless of selected mode
- Gate execution based on world state, server health, or administrative rules
- Disable or constrain TURBO and AGGRESSIVE behavior when required

PulsePolicy ensures that governor modes remain predictable, auditable, and safe, even under operator-driven or automated mode changes. No governor mode bypasses PulsePolicy enforcement.

Reverse-Async Execution Model

Compute phase (async)

- Runs in CPU, IO, and virtual pools  
- Must never mutate Bukkit world state  
- Produces intent and data for apply phase  

Apply phase (sync / region-safe)

- Performs world mutation on correct thread  
- Scheduled via platform-aware routing  
- Enforced by runtime and core gates  

Safeties include:

- World loaded checks  
- Write-deny rules  
- TTL and staleness drops  
- Rate limiting and fairness enforcement  

Client Ray Tracing and Perception Offload (Fabric)

PulseThread SIS includes optional support for client participation in ray tracing and perception-heavy workloads, primarily targeting Fabric-based clients.

Ray tracing workloads are distributed using the same per-player hybrid execution model used elsewhere in PulseThread. Each player’s perception and ray-based queries are treated as independently governed workloads, allowing rays to be computed in parallel without creating global contention.

Because ray tracing is staged as bounded, async compute and applied only as validated data, it is significantly cheaper than traditional synchronous or per-tick ray tracing approaches. Work is batched, rate-limited, and distributed across available cores, avoiding the cost of repeated main-thread scans.

The server advertises capabilities and budgets, issues bounded ray trace requests, and validates all returned data before use. Client results are advisory only and may be dropped due to timeout, mismatch, or policy constraints.

Clients never mutate world state and never act authoritatively. All apply logic remains server-side. Failure or absence of client participation results in a clean fallback to server-side execution without impacting correctness.

Apply Queues with Fairness and Separation

Apply work is separated into:

- Player-critical apply (per-player round-robin)  
- Global critical apply  
- Background apply  

Benefits:

- Player actions remain responsive  
- Background work cannot starve gameplay  
- Fairness enforced under mixed load  
- Operator visibility into pressure sources  

Rate Limiting (Hardened)

- Global and per-task concurrency caps  
- Permit acquisition and release symmetry enforced  
- Integrated with governor modes  
- Prevents runaway submission from cooperative integrations  

TNT Handling (Hardened)

- Windowed detonation model  
- Chunk-local grouping  
- Deterministic per-chunk ordering  
- Compute offloaded safely  
- Apply performed synchronously  

Prevents backlog tail effects and stall cascades.

Corruption Management

PulseThread includes a bounded corruption handling system designed to safely resolve invalid or corrupted state without introducing uncontrolled world mutation.

Three-layer replacement model
- Default replacement rules defined by PulseThread
- Plugin-provided override rules (when explicitly registered)
- Cache-based fallback when no direct rule is available

Strict replacement guarantees
- Corrupted blocks are replaced by explicit Material types only
- Corrupted entities are replaced by explicit EntityType definitions only
- No arbitrary reconstruction or inference is performed

All corruption handling is recorded, rate-limited, and bounded. No automatic or speculative world mutation occurs outside of explicit replacement rules, and all apply actions are executed through normal safety and policy gates.

Known Issues

- AI prediction and advanced perception may engage less frequently during short or burst encounters due to conservative gating and state reset behavior.
- Client ray tracing and client-side offload paths are present but not yet fully utilized in all deployments or configurations.
- Skeleton entities may dodge less consistently due to different timing characteristics and attack cadence when using ranged (bow) behavior.


Thank you, and much love  
WhiskeyMC @ PerfectPriceProjectsLLC

