# Expensive Test Runs

Use this reference for tests that start models, GPU contexts, browsers, services, worker processes, Ray clusters, or other environments whose startup and cleanup materially affect feedback time.

## Reuse a healthy instance only with valid evidence

Reuse an instance across normal scenarios only when all of these remain stable:

- source build and loaded process code;
- dependencies and runtime configuration;
- hardware and process topology relevant to the contract;
- requests, transactions, caches, shared state, and allocator state can be reliably reset.

Create a fresh instance after any relevant change, uncertain cleanup, leaked task, poisoned connection, corrupted cache, or process-global mutation.

## Give destructive behavior an isolated lifecycle

Use an independent instance for tests of:

- startup and shutdown;
- partial initialization failure;
- forced process termination;
- timeout and cancellation behavior;
- fault propagation;
- GPU allocator, Ray resources, shared memory, signals, or process-global state.

An isolated instance is also required when the behavior being proved is the lifecycle itself.

## Match the real mechanism

Use the mechanism that carries the contract. IPC behavior needs real process communication. GPU memory behavior needs the relevant device path. Model-output equivalence needs actual model execution under the compared configurations. Service protocol behavior needs the real serialization and transport path.

Performance evidence is required when the change touches an existing performance contract or changes a measured hot path, synchronization frequency, cache behavior, memory layout, serialization volume, copy count, batching, or model-loading path. Run it without competing workloads that would invalidate the measurement.

## Record useful cost evidence

Record the command, source revision or build identity, relevant configuration, elapsed time, restart reason, and result for expensive runs and final acceptance. Ordinary fast checks need only their final result unless their timing is the subject of the work.

The expensive-run branch is complete when every reused instance satisfies the reuse conditions, every destructive case has an isolated lifecycle, and every claim names the real mechanism that supplied its evidence.
