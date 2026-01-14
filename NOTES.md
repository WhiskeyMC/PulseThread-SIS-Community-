PulseThread SIS (Community)

PulseThread SIS is a workload coordination and execution framework for Minecraft servers. It is designed to be hooked into by other plugins to safely stage, govern, and execute heavy or disruptive work without overwhelming the main thread or region threads.

Rather than acting as a standalone optimizer, PulseThread provides a shared execution and policy layer that cooperative plugins can integrate with. It centralizes scheduling, batching, rate limiting, and safety enforcement so that participating plugins operate under consistent rules instead of competing independently for server time.

PulseThread SIS is designed as a coordinated system rather than a single plugin. Most real functionality is delivered through companion plugins that integrate into the shared execution pipeline.
SIS Plugin Architecture Overview

PulseThread SIS is composed of a core execution pipeline and a set of optional domain modules. Not all modules are included in every edition. Availability, limits, and behavior are governed by the applicable license Schedule A.
Core Runtime Plugins (All Editions)

These plugins form the mandatory execution pipeline and are always present in operational deployments:

    PulseThreadCore
    Final execution authority. Owns thread pools, batching, rate limiting, governor modes, apply queues, watchdogs, and operator visibility.

    PulseRuntime
    Policy and safety enforcement layer. Validates task intent, world state, MSPT signals, and reverse-async safety before execution is permitted.

    PulseHook
    Discovery and staging layer. Integrates with cooperative plugins, enforces quotas and ownership, and stages work into Runtime without executing it directly.

Optional SIS Modules (Edition-Dependent)

The following companion plugins extend the core runtime and are enabled based on license tier and deployment requirements:

Community / Pro (limited or policy-gated in Community)

    PulseGen
    Offloading and coordination for generation-heavy workloads, structure placement, and chunk-adjacent computation.

    PulseAI
    Entity, AI, and behavior task coordination with batching, rate limiting, and governor-controlled throughput.

Enterprise Only

    PulseWorld
    World-level health, lifecycle control, and integrity management, including recovery and enforcement mechanisms.

    PulseHealth
    Long-term performance analytics, pressure forecasting, and predictive health signals.

    Advanced Policy Extensions
    Custom governors, override hooks, and integration allowances beyond public configurations.

Not all modules listed above are included in the Community edition. Redistribution, commercial use, and module availability are governed by the applicable license and Schedule A.
Editions
Community Edition

The Community edition provides the full core architecture and safety model of PulseThread SIS for non-commercial use. It includes the execution framework, governor logic, safety gates, and companion plugins required for real operation.

This edition is intended for evaluation, learning, and non-monetized servers.
Pro Edition

The Pro edition expands on the Community foundation with support for monetized and commercial servers, along with additional tuning flexibility and operational allowances.

Details for Pro licensing and distribution will be published separately.
Enterprise Edition

Enterprise deployments are handled on a per-organization basis. Enterprise licensing may include expanded modules, higher limits, custom integration allowances, and support agreements.

Enterprise terms are negotiated individually and are not publicly documented at this time.
PulseThread — Technical Overview (v9.0)

Status: FEATURE-COMPLETE / ARCHITECTURE-LOCKED
Phase: ALPHA 9.0
Target: Publish-ready pending final security + documentation pass

Note: Core architecture and governor systems were finalized in v6.0 and carried forward unchanged into v9.0.
Executive Summary

PulseThread is a performance, stability, and fairness control layer for Minecraft servers.

It provides:

    A controlled execution gateway for heavy workloads
    Adaptive backpressure to prevent runaway load
    Fairness across competing sources of work
    Policy-driven recovery under stress
    Deterministic operator control over throughput behavior

PulseThread is not a magic compatibility layer and does not attempt to fix non-cooperative plugins. It focuses on safe control of cooperative workloads and server-side pressure management.

This document reflects the consolidated implementation carried forward into v9.0 and represents the current production-ready architecture.
Design Goals
Primary goals

    Reduce main-thread and region-thread pressure safely
    Maintain predictable player experience under load
    Prevent backlog drain storms and cascading stalls
    Provide clear policy knobs instead of hard-coded behavior
    Recover automatically after extreme spikes

Non-goals

    Universal async conversion of Bukkit API calls
    Automatic correction of unsafe third-party behavior
    Guaranteeing Folia compatibility for all plugins

Architecture (Locked — Do Not Collapse)

Pipeline:
PulseHook → PulseRuntime → PulseThreadCore

This separation is intentional:

    Hook discovers and stages work
    Runtime enforces safety and policy gates
    Core executes and governs throughput

No responsibilities are collapsed.
No async Bukkit mutation paths are introduced.
PulseHook — Discovery and Staging
Role

    Discovers cooperative plugins and provides integration points
    Enforces ownership, quotas, and opt-in boundaries
    Stages work into Runtime (never executes directly)

Key constraints

    Maintains lightweight caches only (UUIDs, coordinates, identifiers)
    No hard references to Chunk, Block, or World objects
    Must remain execution-free to preserve isolation

PulseRuntime — Policy and Safety Gates
Role

    Policy layer that decides whether work may proceed
    Performs safety validation before execution is allowed

Owns

    Task intent classification (ENTITY, STRUCTURE, READ, WRITE, PHYSICS, TNT)
    World lifecycle gating (unload, write-deny, TTL drop)
    Reverse-async enforcement rules
    MSPT sampling (authoritative signal source)

Does not

    Own thread pools
    Execute tasks

Runtime answers:
Is this work safe and relevant right now?
PulseThreadCore — Authority and Execution
Role

    Final authority that executes work and governs throughput

Owns

    CPU, IO, efficiency, and virtual thread pools
    Batching and apply queues with fairness routing
    Global and per-world rate limiting
    Governor modes and dynamic budgets
    Watchdog, metrics, and operator visibility

Core answers:
How much work should execute right now, and where?
Core Features (Implemented / Stable)
Dynamic Thread Pools

CPU Pool

    Chunk scanning
    Computation-heavy tasks
    Entity AI processing

IO Pool

    Async structure placement and file IO
    Schematics
    Corrupted chunk reloads (where safe)

Efficiency Pool

    Optional
    Activated only by governor or explicit config
    Used for low-priority background work and cache acceleration

Virtual Thread Pool

    Short-lived lightweight tasks
    Never blocks CPU pool

Additional:

    Hardware-aware thread sizing
    Paper and Folia safe execution paths

Governor-Driven Dynamic Batching and Apply Budgeting

    Batching is fully controlled by governor policy
    Modes define base throughput intent
    Runtime signals dynamically scale execution

Signals consumed

    MSPT (fast / slow EMA)
    Apply backlog pressure
    CPU availability

Apply batching

    Main-thread apply batch size set each tick by governor
    Apply budgetMs respected at all times
    No self-tuning inside apply pump

Async batching

    Governor supplies async batch size
    BatchManager executes without policy logic

This eliminates conflicting heuristics and makes batching predictable.
Governor Modes (Finalized)

SAFE

    Emergency recovery mode
    Aggressive clamping of intake and batching
    Automatic exit when signals recover

NORMAL

    Balanced default
    MSPT-governed scaling
    Sustained operation

AGGRESSIVE

    Sustained higher throughput
    Fully MSPT-governed
    No time limit
    Drops automatically to NORMAL or SAFE

TURBO (not a governor mode)

    Per-world, time-limited burn mode
    Credit-based, non-spammable
    Scales with CPU capacity
    Aborts instantly on SAFE
    Biases world scheduling and batching
    Redirects unused headroom to cache work

BOOST remains a legacy alias for TURBO.
Reverse-Async Execution Model

Compute phase (async)

    Runs in CPU, IO, and virtual pools
    Must never mutate Bukkit world state
    Produces intent and data for apply phase

Apply phase (sync / region-safe)

    Performs world mutation on correct thread
    Scheduled via platform-aware routing
    Enforced by runtime and core gates

Safeties include:

    World loaded checks
    Write-deny rules
    TTL and staleness drops
    Rate limiting and fairness enforcement

Apply Queues with Fairness and Separation

Apply work is separated into:

    Player-critical apply (per-player round-robin)
    Global critical apply
    Background apply

Benefits:

    Player actions remain responsive
    Background work cannot starve gameplay
    Fairness enforced under mixed load
    Operator visibility into pressure sources

Rate Limiting (Hardened)

    Global and per-task concurrency caps
    Permit acquisition and release symmetry enforced
    Integrated with governor modes
    Prevents runaway submission from cooperative integrations

TNT Handling (Hardened)

    Windowed detonation model
    Chunk-local grouping
    Deterministic per-chunk ordering
    Compute offloaded safely
    Apply performed synchronously

Prevents backlog tail effects and stall cascades.
Corruption Management

Three-layer replacement:

    Default rules
    Plugin overrides
    Cache fallback

Strict replacement types:

    Blocks to Material
    Entities to EntityType

Corruption handling is planned as a background flagging and signal mechanism rather than an automatic repair system. The intent is for invalid or corrupt data to be recorded in a bounded cache so the governor can account for it when making scheduling and backpressure decisions, reducing repeated processing of known-bad state that could otherwise contribute to main-thread pressure. In current releases, portions of this logic may be unintentionally active in limited execution paths; this does not constitute a supported repair feature and is not relied upon for correctness. No automatic world mutation is intended, and a future release (v10.0) will introduce explicit, operator-invoked commands for bounded, synchronous repair after appropriate validation.

Async-safe, retry-capped, fully logged.
Physics and Entity Processing

    Optimized batching
    Virtual-thread submission supported
    Atomic job tracking
    No invalid world access paths

Metrics, Watchdog, Operator UX

Metrics

    Queue sizes (critical and background)
    Apply backlog pressure
    Pool utilization targets
    Task completion counts
    Drops and denials

Watchdog

    Non-blocking
    MSPT-driven stall detection
    Startup grace window
    Safe fallback behavior
    No false SAFE latching

Commands

    /pulsethread status
    /pulsethread mode normal|aggressive
    /pulsethread turbo <world>
    /pulsethread boost (legacy alias)

Real-World Validation Results

Validated under extreme mixed workloads:

    80k TNT
    800+ wardens
    1000+ entities
    EliteMobs worlds
    Large world sizes and spikes

Observed:

    Server survives load instead of stalling
    SAFE triggers and recovers cleanly
    TURBO aborts instantly on stress
    No deadlocks
    No corruption cascades

Testing Coverage Summary

This section documents the types of workloads and environments under which PulseThread SIS has been exercised.

Entries describe observed behavior during testing and do not constitute compatibility guarantees beyond those conditions.
Entity-Heavy Combat Environments

EliteMobs

Tested in active EliteMobs worlds with:

    High hostile mob density
    Frequent combat events
    Concurrent player activity
    Mixed entity AI and spawn pressure

Observed behavior:

    Entity-related workload governed correctly
    No stall cascades during combat spikes
    SAFE mode triggered and recovered as designed under stress
    No corruption or deadlocks observed

Mixed Entity + Structure Load

EliteMobs + Structure Generation

Tested with EliteMobs active alongside structure placement workloads.

Observed behavior:

    Entity processing remained responsive during structure load
    Apply batching and fairness routing behaved as designed
    No starvation of player-critical work observed

World Generation / Structure Placement

Structure-Heavy Worlds

Tested with structure placement workloads and chunk-adjacent computation under live server conditions.

Observed behavior:

    Structure apply throughput governed correctly
    No unsafe async world mutation paths introduced
    SAFE mode and recovery behaved predictably

Burst Block Mutation (Datapacks)

Player-Triggered Block-Breaking Datapacks

Tested under mass block break scenarios initiated by players.

Observed behavior:

    Burst block updates were governed correctly
    No stall cascades or corruption observed
    Apply budgets respected under pressure

Explosives / Physics Stress

Large TNT Detonation Scenarios

Tested with large TNT bursts during live operation.

Observed behavior:

    Windowed detonation prevented backlog tail effects
    Server remained responsive under extreme physics load
    SAFE mode engaged and recovered cleanly when required

Operational Stability

Long-Running Mixed Workloads

Tested during extended uptime with entities, structures, datapacks, and player activity active simultaneously.

Observed behavior:

    No deadlocks observed
    No runaway queue growth
    Governor and backpressure behaved as designed

Testing Scope Disclaimer

Testing reflects observed behavior under the above conditions.

It does not imply:

    Universal compatibility
    Support for unsafe plugin configurations
    Automatic correction of non-cooperative behavior

Remaining Optional Work

    Memory pressure feedback
    Config documentation polish

Safety Requirements (Non-Negotiable)

    No async Bukkit world mutation
    No collapse of Hook, Runtime, or Core
    No unsafe Folia region bypass
    No policy logic hidden in config
    No removal of safety clamps without signal correctness

Mental Model

PulseThread is:

    A governor
    A safety rail
    A fairness layer
    A controlled execution gateway

PulseThread is not:

    A universal async engine
    A compatibility shim
    A plugin auto-fixer

Testing Status / Disclaimer

PulseThread SIS Community has been primarily tested on Minecraft 1.21.8 through 1.21.10 and has undergone extensive stress testing under heavy mixed workloads with no major stability issues observed so far.

This is still an alpha-phase framework, and wider real-world testing is encouraged. Behavior, limits, and defaults may continue to evolve as additional validation and feedback are collected.

Thank you, and much love
WhiskeyMC @ PerfectPriceProjectsLLC
