# Memory Leak Detection

Finding and fixing memory leaks in applications.

## What Is a Memory Leak?

```
Memory leak: Memory that is allocated but never freed,
causing the application to consume more memory over time.

Normal:                    Memory leak:
Memory ──┬──┬──┬──         Memory ──────────────►
         │  │  │                   ↗ ↗ ↗ ↗ ↗
         └──┴──┴── (GC)            No garbage collection
                                   because references exist
```

## Symptoms of Memory Leaks

```markdown
Warning signs:
- [ ] Memory usage grows over time
- [ ] Application slows down gradually
- [ ] OOM (Out of Memory) kills
- [ ] Increasing GC pause times
- [ ] Performance degrades after running for hours/days
```

## Memory Leak Types

### Event Listener Leaks

```typescript
// Problem: Event listeners never removed
class Component {
  constructor() {
    // Listener holds reference to 'this'
    window.addEventListener('resize', this.handleResize);
  }

  handleResize = () => {
    console.log(this.state);
  }

  // Missing cleanup!
}

// Fix: Remove listeners on cleanup
class Component {
  constructor() {
    window.addEventListener('resize', this.handleResize);
  }

  handleResize = () => {
    console.log(this.state);
  }

  destroy() {
    window.removeEventListener('resize', this.handleResize);
  }
}
```

### Closure Leaks

```typescript
// Problem: Closure retains large data
function processData() {
  const largeData = new Array(1000000).fill('x');

  return function callback() {
    // callback retains reference to largeData
    // even if it doesn't use it
    console.log('Processing...');
  };
}

// Fix: Don't close over unnecessary data
function processData() {
  const largeData = new Array(1000000).fill('x');
  const result = processLargeData(largeData);

  return function callback() {
    // Only retains 'result', not 'largeData'
    console.log('Result:', result);
  };
}
```

### Timer Leaks

```typescript
// Problem: Interval never cleared
function startPolling() {
  setInterval(() => {
    fetchData();
  }, 1000);
  // Interval ID lost, can never be cleared
}

// Fix: Track and clear timers
class Poller {
  private intervalId: NodeJS.Timeout | null = null;

  start() {
    this.intervalId = setInterval(() => {
      fetchData();
    }, 1000);
  }

  stop() {
    if (this.intervalId) {
      clearInterval(this.intervalId);
      this.intervalId = null;
    }
  }
}
```

### DOM Reference Leaks

```typescript
// Problem: Detached DOM nodes held in memory
const elements: Element[] = [];

function addElement() {
  const el = document.createElement('div');
  document.body.appendChild(el);
  elements.push(el);  // Reference kept
}

function removeElement() {
  const el = elements[0];
  el.remove();  // Removed from DOM
  // But still in array! Memory leak.
}

// Fix: Clear references when removing
function removeElement() {
  const el = elements.shift();  // Remove from array
  el?.remove();  // Remove from DOM
}
```

### Cache Leaks

```typescript
// Problem: Unbounded cache
const cache = new Map();

function getCached(key: string) {
  if (!cache.has(key)) {
    cache.set(key, expensiveComputation(key));
  }
  return cache.get(key);
}
// Cache grows forever

// Fix: Bounded cache with LRU eviction
import LRU from 'lru-cache';

const cache = new LRU({
  max: 500,  // Maximum entries
  maxAge: 1000 * 60 * 60  // 1 hour TTL
});

function getCached(key: string) {
  if (!cache.has(key)) {
    cache.set(key, expensiveComputation(key));
  }
  return cache.get(key);
}
```

### Subscription Leaks

```typescript
// Problem: Subscriptions never unsubscribed
class UserComponent {
  subscription: Subscription;

  ngOnInit() {
    this.subscription = userService.user$.subscribe(user => {
      this.user = user;
    });
  }

  // Missing ngOnDestroy - subscription leaks!
}

// Fix: Unsubscribe on destroy
class UserComponent {
  subscription: Subscription;

  ngOnInit() {
    this.subscription = userService.user$.subscribe(user => {
      this.user = user;
    });
  }

  ngOnDestroy() {
    this.subscription?.unsubscribe();
  }
}
```

## Detection Tools

### Node.js Memory Debugging

```javascript
// Check memory usage
const used = process.memoryUsage();
console.log({
  heapUsed: Math.round(used.heapUsed / 1024 / 1024) + ' MB',
  heapTotal: Math.round(used.heapTotal / 1024 / 1024) + ' MB',
  external: Math.round(used.external / 1024 / 1024) + ' MB',
  rss: Math.round(used.rss / 1024 / 1024) + ' MB'
});

// Track memory over time
setInterval(() => {
  const used = process.memoryUsage();
  console.log('Heap used:', Math.round(used.heapUsed / 1024 / 1024), 'MB');
}, 5000);
```

### Heap Snapshots

```javascript
// Take heap snapshot
const v8 = require('v8');

// Snapshot 1: Baseline
v8.writeHeapSnapshot('heap-1.heapsnapshot');

// Run operations that might leak
doSuspectedLeakingOperations();

// Snapshot 2: After operations
v8.writeHeapSnapshot('heap-2.heapsnapshot');

// Compare snapshots in Chrome DevTools:
// 1. Open DevTools > Memory
// 2. Load both snapshots
// 3. Select "Comparison" view
// 4. Look for objects that grew
```

### Chrome DevTools Memory Panel

```markdown
## Heap Snapshot Workflow

1. Open DevTools > Memory tab
2. Take heap snapshot (baseline)
3. Perform suspected leaking actions
4. Take another heap snapshot
5. Compare snapshots:
   - Objects allocated between snapshots
   - Detached DOM trees
   - Growing arrays/maps

## What to Look For

- Detached HTMLElement: DOM nodes removed but referenced
- Closure: Functions holding unexpected references
- Array: Growing collections
- Object: Accumulating objects

## Allocation Timeline

1. Select "Allocation instrumentation on timeline"
2. Record while performing actions
3. Blue bars = allocated
4. Gray bars = freed
5. Persistent blue bars = potential leaks
```

## Memory Leak Investigation

### Step-by-Step Process

```markdown
1. Establish baseline
   - Start fresh application
   - Note initial memory usage

2. Create suspected leak condition
   - Perform action repeatedly
   - Watch memory growth

3. Confirm leak
   - Force garbage collection
   - If memory doesn't drop, leak confirmed

4. Take heap snapshots
   - Before: Baseline snapshot
   - After: Post-action snapshot
   - Compare to find retained objects

5. Find retaining path
   - In heap snapshot, find growing objects
   - Look at "Retainers" to see what holds reference

6. Fix the leak
   - Remove the retaining reference
   - Verify memory is freed

7. Verify fix
   - Repeat leak condition
   - Confirm memory is stable
```

### Finding Retaining Paths

```
Heap Snapshot Analysis:

Object: Large Array (10MB)
└── Retainers:
    └── callback in processData()
        └── timerId in setInterval()
            └── global

This tells us:
- A large array is being retained
- By a callback function
- Called from setInterval
- Which is global

Fix: Clear the interval and null the reference
```

## Prevention Patterns

### WeakMap and WeakSet

```typescript
// Bad: Map retains keys forever
const cache = new Map();
cache.set(domElement, data);
// domElement can't be garbage collected

// Good: WeakMap allows GC
const cache = new WeakMap();
cache.set(domElement, data);
// When domElement is removed from DOM,
// it can be garbage collected
```

### Cleanup Patterns

```typescript
// Pattern: Disposable
interface Disposable {
  dispose(): void;
}

class ResourceManager implements Disposable {
  private resources: Resource[] = [];
  private listeners: (() => void)[] = [];

  addResource(resource: Resource) {
    this.resources.push(resource);
  }

  addListener(el: EventTarget, event: string, handler: EventListener) {
    el.addEventListener(event, handler);
    this.listeners.push(() => el.removeEventListener(event, handler));
  }

  dispose() {
    this.resources.forEach(r => r.release());
    this.resources = [];
    this.listeners.forEach(remove => remove());
    this.listeners = [];
  }
}
```

### React Cleanup

```typescript
// React hooks cleanup pattern
function Component() {
  useEffect(() => {
    const subscription = api.subscribe(handleData);
    const interval = setInterval(poll, 1000);
    window.addEventListener('resize', handleResize);

    // Cleanup function - runs on unmount
    return () => {
      subscription.unsubscribe();
      clearInterval(interval);
      window.removeEventListener('resize', handleResize);
    };
  }, []);
}
```

## Memory Leak Checklist

```markdown
## Code Review Checklist

- [ ] Event listeners have corresponding removeEventListener
- [ ] setInterval/setTimeout have corresponding clear
- [ ] Subscriptions have unsubscribe on cleanup
- [ ] Caches have size limits or TTL
- [ ] Large objects are dereferenced after use
- [ ] Closures don't capture unnecessary scope

## Testing Checklist

- [ ] Memory usage is stable under sustained load
- [ ] Memory returns to baseline after operations
- [ ] No detached DOM nodes after navigation
- [ ] No growing collections in heap snapshots
```
