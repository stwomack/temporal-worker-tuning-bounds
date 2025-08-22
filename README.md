# Temporal Worker Tuning Reference

| Area        | Definition                               | Upper Bound                         | Lower Bound / Starting Point | Key Metrics / Notes                                  |
|-------------|------------------------------------------|-------------------------------------|-----------------------------|------------------------------------------------------|
| **Pollers** | Threads fetching tasks from Task Queue   | ~20k per Task Queue (total)         | 4 (default partition count) | Scale until poll success rate stable, latency low    |
| **Task Slots** | Concurrency for workflow/activity tasks | CPU-bound (thread pool size)        | Fully utilize slots         | Monitor queue backlog, target ~70–80% CPU           |
| **Stickiness** | In-memory workflow state cache        | Memory-based (LRU eviction)         | ~200 if autoscaling         | Low = higher replay latency; High = less horiz. scaling |
| **Scaling** | How to grow worker capacity             | Vertical (CPU/mem/slots) preferred  | N/A                          | Horizontal only helps new/uncached workflows        |

## Quick Tuning Rules

- Keep pollers high enough to avoid starvation, but watch CPU.  
- Size task slots to match CPU headroom; avoid overcommitting threads.  
- Lower stickiness for frequent autoscaling; raise it if replay latency is a problem.  
- Use metrics (poll success rate, sticky hit rate, queue latency, CPU) to drive changes, not guesswork.  


# Temporal Worker Tuning Flow

## Step 1: Pollers
- Start with 4 pollers (default partition count).
- Monitor:
  - **Poll success rate**: Should be high and stable.
  - **Task queue latency**: Should be low.
- Increase pollers until these metrics stabilize.
- Hard cap: ~20k pollers per Task Queue.

## Step 2: Task Slots
- Set workflow and activity task slots to fully utilize CPU without saturation.
- Monitor:
  - **Queue backlog**: Avoid long waits.
  - **CPU utilization**: Target ~70–80%.
- Adjust:
  - Increase slots if CPU is under target and backlog exists.
  - Decrease if CPU is overcommitted.

## Step 3: Stickiness (Workflow Cache)
- Start with:
  - **Default**: 1000 (Go SDK).
  - **Autoscaling**: ~200 to allow for scaling headroom.
- Monitor:
  - **Sticky cache hit rate**: Higher is better for latency.
  - **Memory usage**: Ensure no cache thrashing.
- Adjust:
  - Lower if autoscaling frequently (avoid stale cache).
  - Increase if replay latency is causing SLA issues.

## Step 4: Scaling Strategy
- Prefer **vertical scaling** (CPU, memory, slots) for existing workflows.
- Use **horizontal scaling** primarily for:
  - New workflows entering the Task Queue.
  - Load spikes requiring more pollers.
- Always base scaling decisions on:
  - **Poll success rate**
  - **Sticky cache hit rate**
  - **Queue latency**
  - **CPU utilization**

---

## Quick Rules of Thumb
- Don’t starve pollers; don’t oversubscribe threads.
- Tune stickiness with autoscaling in mind.
- Treat metrics as source of truth—avoid guesswork.

