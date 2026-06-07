# js-tuple

A high-performance JavaScript library for managed nested maps, nested set and creating immutable, cached tuples that can be safely used as Map keys.
The Nested classes are also high-performant and strong tree structures that can be used for high configurable traversals

[![Maintainability](https://qlty.sh/gh/codibre/projects/js-utils/maintainability.svg)](https://qlty.sh/gh/codibre/projects/js-utils)
[![Code Coverage](https://qlty.sh/gh/codibre/projects/js-utils/coverage.svg)](https://qlty.sh/gh/codibre/projects/js-utils)

## Why js-tuple?

In JavaScript, arrays are compared by reference, not by value. This means `[1, 2, 3] !== [1, 2, 3]`, which makes arrays unsuitable as Map keys when you want value-based equality. **js-tuple** solves this by providing cached, immutable arrays where identical element sequences always return the same reference.
It also offers the classes **NestMap** and **NestSet**, which deals with array keys internally giving the same tuple experience but without the need of calling tuple every time.

## Installation

```bash
npm install js-tuple
```

## Quick Start

```typescript
import { tuple, NestedMap, NestedSet } from 'js-tuple';

// RECOMMENDED: NestedMap and NestedSet
// NestedMap: value-based equality for array keys
const map = new NestedMap<[number, string], string>();
map.set([1, 'a'], 'foo');
map.set([1, 'a'], 'bar'); // Overwrites previous value
map.set([2, 'b'], 'baz');
console.log(map.get([1, 'a'])); // 'bar'
console.log(map.size); // 2

// NestedSet: value-based equality for array keys
const set = new NestedSet<[number, string]>();
set.add([1, 'a']);
set.add([1, 'a']); // No duplicate
set.add([2, 'b']);
console.log(set.has([1, 'a'])); // true
console.log(set.size); // 2


// Another option if you can't use NestedMap or NestedSet: tuple

// Same elements = same reference
const t1 = tuple([1, 2, 3]);
const t2 = tuple([1, 2, 3]);
console.log(t1 === t2); // true ✅

// Perfect for Map keys
const cache = new Map();
cache.set(tuple(['user', 123]), { name: 'John' });
cache.set(tuple(['post', 456]), { title: 'Hello' });

// Retrieve values using the same tuple
console.log(cache.get(tuple(['user', 123]))); // { name: 'John' } ✅
```

## Features

- **🔒 Immutable**: All returned tuples are frozen and cannot be modified
- **⚡ High Performance**: Optimized tree-based caching outperforms string serialization
- **🧠 Memory Efficient**: Uses WeakRefs and WeakMaps for automatic garbage collection
- **🎯 Type Safe**: Full TypeScript support with proper type inference
- **🔄 Reference Equality**: Same elements always return the same instance
- **🗑️ Auto Cleanup**: Unused tuples are automatically garbage collected

## Performance

Latest Benchmark Results:

| Method                        | Creation + Map Set         | Map Lookup                |
|-------------------------------|----------------------------|---------------------------|
| **Tuple - Create key + Map set**      | 650,280 ops/sec ±7.50%   | 1,093,434 ops/sec ±3.63%    |
| JSON.stringify                | 414,829 ops/sec ±4.52%     | 471,930 ops/sec ±5.91%    |
| JSON.stringify + MD5          | 61,155 ops/sec ±9.86%      | 57,899 ops/sec ±9.35%     |
| Manual stringify              | 564,227 ops/sec ±4.57%     | 497,562 ops/sec ±9.05%    |
| Manual stringify + MD5        | 63,409 ops/sec ±6.32%      | 59,595 ops/sec ±12.22%    |
| **Nested Map - Direct array** | **700,708 ops/sec ±4.09%**     | **2,386,124 ops/sec** ±4.27% |

**Fastest option overall:** Nested Map is the best option in all cases

Both tuple and nested strategies are much faster than any stringification approach.

## API Reference

### `tuple<T>(elements: T): Readonly<T>`

Creates a cached, immutable tuple from an array of elements.

**Parameters:**
- `elements`: Array of elements to create a tuple from

**Returns:**
- Frozen array instance that maintains reference equality for identical element sequences

**Example:**
```typescript
const coordinates = tuple([10, 20]);
const sameCoordinates = tuple([10, 20]);
console.log(coordinates === sameCoordinates); // true
```

### `getRef(value: unknown): Reference`

Gets an object reference for any value, enabling it to be used as a WeakMap key. This is primarily used internally but exported for advanced use cases.

**Parameters:**
- `value`: Any value to get a reference for

**Returns:**
- Object that can be used as a WeakMap key

### `getOrSet(key: K, factory: () => V): V`

Retrieves the value for a key if it exists; otherwise, creates it using the provided factory function and stores it. Eliminates manual `if (!cache.get(key)) { cache.set(key, ...) }` boilerplate. Ideal for caching expensive computations or grouping items by composite keys.

**Parameters:**
- `key`: The array key to look up (e.g., `[userId, 'profile']`)
- `factory`: A function that returns the default value if the key is missing

**Returns:**
- The existing value or the newly created one

**Example:**
```typescript
const cache = new NestedMap<[string, string], UserData>();

// Instead of:
let user = cache.get(['user', '123']);
if (!user) { user = fetchUser('123'); cache.set(['user', '123'], user); }

// Use getOrSet:
const user = cache.getOrSet(['user', '123'], () => fetchUser('123'));
```

## Usage Examples

### Map Keys

With NestedMap:

```typescript
import { tuple } from 'js-tuple';

const userCache = new NestedMap();

// Store data using tuple keys
userCache.set(['user', 'profile', 123], { name: 'Alice' });
userCache.set(['user', 'settings', 123], { theme: 'dark' });

// Retrieve data using the same tuple structure
const profile = userCache.get(['user', 'profile', 123]);
console.log(profile); // { name: 'Alice' }
```

With tuple function:

```typescript
import { tuple } from 'js-tuple';

const userCache = new Map();

// Store data using tuple keys
userCache.set(tuple(['user', 'profile', 123]), { name: 'Alice' });
userCache.set(tuple(['user', 'settings', 123]), { theme: 'dark' });

// Retrieve data using the same tuple structure
const profile = userCache.get(tuple(['user', 'profile', 123]));
console.log(profile); // { name: 'Alice' }
```

### Set Operations

With NestedSet:

```typescript
import { tuple } from 'js-tuple';

const coordinates = new NestedSet();

coordinates.add([0, 0]);
coordinates.add([1, 1]);
coordinates.add([0, 0]); // Won't add duplicate

console.log(coordinates.size); // 2
console.log(coordinates.has([0, 0])); // true
```

With tuple function:

```typescript
import { tuple } from 'js-tuple';

const coordinates = new Set();

coordinates.add(tuple([0, 0]));
coordinates.add(tuple([1, 1]));
coordinates.add(tuple([0, 0])); // Won't add duplicate

console.log(coordinates.size); // 2
console.log(coordinates.has(tuple([0, 0]))); // true
```

## Caching & Grouping Patterns

`js-tuple` shines when you need to cache results or group items by composite keys (multiple attributes). `NestedMap.getOrSet()` eliminates the boilerplate of manual initialization.

### Caching Expensive Computations

Avoid re-computing values for the same input combination:

```typescript
import { NestedMap } from 'js-tuple';

const expensiveCache = new NestedMap<[string, number], string>();

function computeResult(type: string, id: number): string {
  // Simulate heavy computation
  return `Processed ${type}#${id}`;
}

// Before: manual boilerplate
let result1 = expensiveCache.get(['user', 123]);
if (!result1) {
  result1 = computeResult('user', 123);
  expensiveCache.set(['user', 123], result1);
}

// After: clean one-liner
const result2 = expensiveCache.getOrSet(['user', 123], () => computeResult('user', 123));
```

### Grouping / Bucketing by Composite Key

Group items without string concatenation or manual `if (!group) { group = []; map.set(key, group); }` boilerplate:

```typescript
import { NestedMap } from 'js-tuple';

interface Particle {
  color: string;
  alpha: number;
  draw(): void;
}

const groups = new NestedMap<[string, number], Array<() => void>>();

function addParticle(particle: Particle): void {
  // Group by [color, alpha] — no string concat needed!
  const group = groups.getOrSet([particle.color, particle.alpha], () => []);
  group.push(particle.draw);
}

// Usage in a render loop:
for (const p of particles) {
  addParticle(p);
}

groups.forEach((drawFns, [color, alpha]) => {
  ctx.fillStyle = color;
  ctx.globalAlpha = alpha;
  drawFns.forEach(fn => fn());
});
```

## Mixed Types Support

tuple works with any combination of primitive values and objects:

```typescript
const obj = { id: 1 };
const symbol = Symbol('test');

const mixed1 = tuple([1, 'hello', obj, symbol, null]);
const mixed2 = tuple([1, 'hello', obj, symbol, null]);

console.log(mixed1 === mixed2); // true - same references
```

## TypeScript Support

Full TypeScript support with proper type inference:

```typescript
// Type is inferred as Readonly<[number, string, boolean]>
const typed = tuple([42, 'hello', true] as const);

// Works with generic functions
function createKey<T extends readonly unknown[]>(elements: T): Readonly<T> {
  return tuple(elements);
}
```

## Limitations

- **Primitive Caching**: Wrapper objects for primitives are cached with strong references (may accumulate memory usage in very dynamic scenarios). This is not a problem for **NestedMap** and **NestedSet**, as they don't use the tuple strategy to control array keys, but a nested one.
- **Object Identity**: Objects are compared by reference, not deep equality
- **Modern Environments**: Requires WeakRef support (Node.js 14.6+, modern browsers)

## Advanced Usage

For complex scenarios—such as custom traversal orders, subtree iteration, or post-order cleanup—js-tuple provides highly flexible traversal APIs for both `NestedMap` and `NestedSet`. You can choose between depth-first and breadth-first traversal, and between pre-order and post-order yielding, to match your algorithm's needs.


- **NestedMap:** See [Advanced NestedMap.entries](https://github.com/codibre/js-utils/blob/main/libs/js-tuple/docs/nestedmap-entries-advanced.md) for details on traversal modes, yield order, edge cases, and performance considerations.
- **NestedSet:** See [Advanced NestedSet](https://github.com/codibre/js-utils/blob/main/libs/js-tuple/docs/nestedset-advanced.md) for set-specific traversal, subtree operations, and advanced patterns.
- **Traversal Performance:** See [Traversal Performance](https://github.com/codibre/js-utils/blob/main/libs/js-tuple/docs/traversal-performance.md) for detailed benchmarks, memory insights, and practical guidance on choosing the best traversal mode for your use case.
- **SubMaps and Cloning:** See [Methods to Obtain Subtrees in NestedMap](https://github.com/codibre/js-utils/blob/main/libs/js-tuple/docs/advanced-getsubmap-example.md) for good insights about working with submaps (which can perform incredibly good for bulk operations) or deep clones of maps and submaps.

These guides cover:
- How to use `basePath` for partial/subtree traversal
- Differences between DFS/BFS and pre/post order
- Edge cases (missing values, empty subtrees)
- Performance and memory tradeoffs

If you need to implement advanced algorithms, process hierarchical data, or optimize traversal, start with these docs!


## Browser Support

- **Node.js**: 14.6.0+
- **Chrome**: 84+
- **Firefox**: 79+
- **Safari**: 14.1+
- **Edge**: 84+

## Contributing

Contributions are welcome! Please read our contributing guidelines and submit pull requests to our repository.

## License

MIT License - see LICENSE file for details.

## Which Should I Use?

### NestedMap & NestedSet

- **Value-based equality**: Arrays with the same values are treated as the same key, even if they are different instances.
- **Deep key support**: Works for any array of values, including objects and primitives (but no deep comparison in objects is done).
- **Convenient API**: No need to call a tuple function, just use arrays directly.
- **Fastest option**: Writing and reading performance are better than tuple-based Map/Set.
- **Best for general usage**: NestedMap & NestedSet are recommended for most use cases.

### Map/Set + tuple function

- **Fastest for writing**: Tuple-based Map/Set is faster for key creation and insertion.
- **Reference equality**: Only identical element arrays return the same reference, so you must always use the tuple function to create keys.
- **Fallback option**: Use this if, for any reason, NestedMap and NestedSet can't be used.

## Summary

- Both **NestedMap/NestedSet** and **Map/Set + tuple** are much better than key stringification for both writing and reading, although the nested variants surpass tuple usage in performance;
- For most applications, **NestedMap & NestedSet** are recommended, as tuple primitive caching may use additional memory over time.
