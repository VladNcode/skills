---
name: zoom-out
description: Tell the agent to zoom out and give broader context or a higher-level perspective. Use when you're unfamiliar with a section of code or need to understand how it fits into the bigger picture.
disable-model-invocation: true
---

`disable-model-invocation` may be honored by the host so this skill is not auto-selected as a model tool; the user can still invoke it explicitly.

## Request

I don't know this area of code well. Go up a layer of abstraction. Give me a **map** of how the touched code fits the system.

## What to produce

Stay bounded: **2–4 levels** above the current files (package → subsystem → product boundary), not an encyclopedia.

Include:

1. **Entry points** — HTTP routes, jobs, CLIs, or public APIs that reach this area
2. **Main modules** — directories or packages that own the flow; one line each on responsibility
3. **Call graph sketch** — who calls whom (arrows or bullets), especially inbound callers of the code in question
4. **Data / dependencies** — notable DBs, queues, external services this area relies on
5. **Gaps** — what you did not verify (e.g. “did not trace mobile client”)

Stop when the map answers “where does this sit and who depends on it?” —not full file-by-file narration.
