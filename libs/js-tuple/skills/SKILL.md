---
name: js-tuple
description: "High-performance nested Maps & Sets for JS — replaces string-concat composite keys and manual getOrSet boilerplate with NestedMap.getOrSet() for caching, grouping, and bucketing"
version: 1.0.0
author: farenheith
license: MIT
platforms: [linux, macos, windows]
metadata:
  tags: [js-tuple, caching, Map, NestedMap, performance, tuple]
---

# js-tuple — Managed Nested Maps & Sets for JS

## Problem
JS arrays compare by reference, not value: `[1,'a'] !== [1,'a']`. Useless as Map keys. String serialization (`[1,'a'].join('|')`) is slow and collision-prone.

Two common anti-patterns emerge from this limitation:

**Anti-pattern 1 — String concatenation for composite keys:**
```ts
const key = `${fillStyle}::${alpha.toFixed(3)}`; // allocates a new string every call
let group = map.get(key);
if (!group) { group = []; map.set(key, group); } // boilerplate
```

**Anti-pattern 2 — Manual `getOrSet` boilerplate:**
```ts
const key = [a, b];
let value = cache.get(key);
if (!value) { value = compute(); cache.set(key, value); }
```

## Solution
`js-tuple` provides cached immutable arrays with reference equality + `NestedMap`/`NestedSet` that handle array keys internally. Eliminates both string concatenation and manual boilerplate.

## Installation
```bash
npm install js-tuple
```

## API

### `tuple<T>(elements: T): Readonly<T>`
Creates a cached, frozen array. Identical element sequences return the same instance.
```ts
import { tuple } from 'js-tuple';
const cache = new Map<Readonly<[number, string]>, number>();
cache.set(tuple([42, 'fish']), 100);
console.log(cache.get(tuple([42, 'fish']))); // 100 ✅
```

### `NestedMap<K extends any[], V>` (RECOMMENDED)
Handles array keys internally — pass arrays directly. ~2x faster than tuple+Map for lookups.
```ts
import { NestedMap } from 'js-tuple';
const cache = new NestedMap<[number, string], number>();
cache.set([42, 'fish'], 100);
console.log(cache.get([42, 'fish'])); // 100 ✅

// getOrSet — eliminates the if/else boilerplate for caching/grouping:
const group = cache.getOrSet([42, 'fish'], () => []);
```

### `NestedSet<K extends any[]>`
Same as NestedMap but for sets.
```ts
import { NestedSet } from 'js-tuple';
const set = new NestedSet<[number, string]>();
set.add([42, 'fish']);
console.log(set.has([42, 'fish'])); // true ✅
```

## Performance (ops/sec)
| Method | Create + Set | Lookup |
|---|---|---|
| `tuple` + `Map` | ~650k | ~1.1M |
| `NestedMap` direct | **~700k** | **~2.4M** |

## Limitations
- Objects compared by reference, not deep equality
- Primitive caching uses strong refs (memory concern in very dynamic scenarios) — NestedMap avoids this for array keys via nested strategy
- Requires `WeakRef` support: Node 14.6+, Chrome 84+, Firefox 79+, Safari 14.1+

## Best Practice
> Use `NestedMap`/`NestedSet` by default. Only use `tuple()` + native `Map/Set` when you need explicit control or API compatibility.

## When to Use This Library (Composite Keys Only)
This library is **only** useful when your lookup requires **2+ dimensions of identity**. If a single key suffices, stick to native `Map`/`Set`.

### ✅ Composite Key Scenarios (Use js-tuple)
- **Grid/Spatial Lookup:** `[tileX, tileY] → Entity[]` — Need both coordinates to identify a cell.
- **Pair/Relationship Cache:** `[entityA, entityB] → CollisionResult` — Need the specific pair, not just one ID.
- **Compound Deduplication:** `[eventType, eventId] → boolean` — Type alone isn't unique; need event ID too.
- **Multi-Dimensional State:** `[roomId, userId] → PresenceData` — Who is in which room at a given time.
- **Grouping/Bucketing by Composite Key:** `[fillStyle, alpha] → DrawFn[]` — Group items by multiple attributes without string concatenation or manual `if (!group) { group = []; map.set(key, group); }` boilerplate. Use `getOrSet([a, b], () => [])`.

### ❌ Simple Key Scenarios (Skip js-tuple)
- **Single Entity Lookup:** `Map<EntityId, Component>` — One ID identifies the component uniquely.
- **Type/Category Sets:** `Set<EventType>` — Just checking if a type exists.
- **Timestamp Indexing:** `Map<Timestamp, Data>` — Single point in time is sufficient.

> Rule of thumb: If your Map/Set question has more than one variable (`"What's the value for X and Y?"`), use js-tuple. If it's just `"What's the value for X?"`, native collections are fine.
