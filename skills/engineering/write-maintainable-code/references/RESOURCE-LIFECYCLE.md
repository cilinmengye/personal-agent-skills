# Resource and Async Lifecycles

Use this reference only when an implementation acquires resources, starts asynchronous work, shares mutable state, or can overlap startup with shutdown.

## Establish ownership

For every resource or task affected by the change, identify:

- who owns it immediately after acquisition;
- the event that transfers ownership, if any;
- how the owner learns that all users have stopped;
- who initiates release and how completion is confirmed.

Pass narrow ownership handles or immutable facts across collaborators. A whole mutable owner object obscures the release contract.

## Cover reachable lifecycle paths

Trace the paths the change can actually reach:

- partial initialization followed by failure;
- startup overlapping cancellation or shutdown;
- a cancelled waiter while owned work continues;
- repeated or concurrent close requests;
- cleanup failure after an earlier operation has failed.

State the invariant that prevents use after release, double release, leaked work, and creation of new resources after shutdown begins.

## Preserve failure evidence

Keep the originating failure visible according to the repository's error model. Continue independent cleanup operations that remain safe. Report cleanup failures through the same model, and retain resources whose users have not confirmed termination. When termination cannot be confirmed, surface a lifecycle failure and apply the repository's explicit timeout, quarantine, or forced-termination policy. Return the unresolved behavior to the owning workflow when the project has no accepted policy.

Reuse the project's lifecycle primitives and states. Add a new state or coordinator only when an existing mechanism cannot express a reachable transition.

The lifecycle check is complete when every affected acquisition has an owner, every ownership transfer has an event, every release waits for the required stop condition, and every reachable failure path has a defined outcome.
