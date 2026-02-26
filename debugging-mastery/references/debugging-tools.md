# Debugging Tools Guide

A comprehensive guide to debugging tools and when to use them.

## Debuggers

### Node.js Debugger

```bash
# Start with inspector
node --inspect app.js

# Start and break on first line
node --inspect-brk app.js

# Connect via Chrome DevTools
# Open chrome://inspect in Chrome
```

```javascript
// Add breakpoint in code
function processOrder(order) {
  debugger;  // Execution pauses here
  // ...
}
```

### VS Code Debugger

```json
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug App",
      "program": "${workspaceFolder}/src/index.js",
      "env": {
        "NODE_ENV": "development"
      }
    },
    {
      "type": "node",
      "request": "attach",
      "name": "Attach to Process",
      "port": 9229
    }
  ]
}
```

### Chrome DevTools

```markdown
## Breakpoint Types

- **Line breakpoint**: Click line number
- **Conditional breakpoint**: Right-click → Add conditional
- **Logpoint**: Right-click → Add logpoint (logs without stopping)
- **DOM breakpoint**: Right-click element → Break on...
- **XHR breakpoint**: Sources → XHR/fetch Breakpoints
- **Event listener breakpoint**: Sources → Event Listener Breakpoints

## Useful Features

- **Watch expressions**: Track variable values
- **Call stack**: See how you got here
- **Scope**: View local, closure, global variables
- **Console**: Evaluate expressions in current scope
- **Network**: Inspect requests/responses
```

## Logging

### Structured Logging

```typescript
// Use structured logging libraries
import pino from 'pino';

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  formatters: {
    level: (label) => ({ level: label })
  }
});

// Good: Structured, searchable
logger.info({ orderId: order.id, userId: user.id }, 'Processing order');

// Bad: String concatenation
console.log('Processing order ' + order.id + ' for user ' + user.id);
```

### Log Levels

```typescript
// Use appropriate levels
logger.trace('Detailed debugging');  // Most verbose
logger.debug('Debugging information');
logger.info('Normal operations');
logger.warn('Warning conditions');
logger.error('Error conditions');
logger.fatal('Fatal errors');  // Least verbose

// Set level via environment
// LOG_LEVEL=debug npm start
```

### Contextual Logging

```typescript
// Add request context
app.use((req, res, next) => {
  req.logger = logger.child({
    requestId: req.id,
    userId: req.user?.id,
    path: req.path
  });
  next();
});

// Use throughout request
req.logger.info('Processing started');
// Output: {"requestId":"abc","userId":"123","path":"/orders","msg":"Processing started"}
```

## Profiling

### CPU Profiling

```javascript
// Node.js built-in profiler
const inspector = require('inspector');
const fs = require('fs');

const session = new inspector.Session();
session.connect();

// Start profiling
session.post('Profiler.enable');
session.post('Profiler.start');

// ... run code to profile ...

// Stop and save
session.post('Profiler.stop', (err, { profile }) => {
  fs.writeFileSync('profile.cpuprofile', JSON.stringify(profile));
  // Open in Chrome DevTools → Performance
});
```

### Memory Profiling

```javascript
// Heap snapshot
const v8 = require('v8');
const filename = v8.writeHeapSnapshot();
console.log(`Heap snapshot: ${filename}`);
// Open in Chrome DevTools → Memory

// Memory usage
console.log(process.memoryUsage());
// { heapUsed, heapTotal, external, rss }
```

### Flame Graphs

```bash
# Generate flame graph with 0x
npm install -g 0x
0x app.js
# Opens browser with flame graph

# Or use clinic
npm install -g clinic
clinic flame -- node app.js
```

## Network Debugging

### curl

```bash
# Basic request
curl https://api.example.com/users

# With headers
curl -H "Authorization: Bearer token" https://api.example.com/users

# POST with JSON
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"name": "John"}' \
  https://api.example.com/users

# Verbose output (shows headers, timing)
curl -v https://api.example.com/users

# Show timing
curl -w "@curl-format.txt" -o /dev/null -s https://api.example.com/users

# curl-format.txt:
#     time_namelookup:  %{time_namelookup}\n
#        time_connect:  %{time_connect}\n
#     time_appconnect:  %{time_appconnect}\n
#    time_pretransfer:  %{time_pretransfer}\n
#       time_redirect:  %{time_redirect}\n
#  time_starttransfer:  %{time_starttransfer}\n
#                     ----------\n
#          time_total:  %{time_total}\n
```

### Wireshark/tcpdump

```bash
# Capture HTTP traffic on port 80
sudo tcpdump -i any port 80 -A

# Capture and save to file
sudo tcpdump -i any port 443 -w capture.pcap

# Open capture.pcap in Wireshark for analysis
```

### Browser DevTools Network Tab

```markdown
## Features

- **Preserve log**: Keep requests across page loads
- **Disable cache**: Force fresh requests
- **Throttling**: Simulate slow networks
- **Filter**: By type, URL, status
- **Copy as cURL**: Right-click → Copy as cURL
- **Timing**: See DNS, SSL, TTFB, download

## What to Check

- Status codes (4xx, 5xx)
- Request/response headers
- Request body (for POST/PUT)
- Response body
- Timing waterfall
- Initiator (what triggered the request)
```

## Database Debugging

### SQL Query Analysis

```sql
-- PostgreSQL: Explain query plan
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 123;

-- MySQL: Explain
EXPLAIN SELECT * FROM orders WHERE user_id = 123;

-- PostgreSQL: See active queries
SELECT pid, now() - pg_stat_activity.query_start AS duration, query
FROM pg_stat_activity
WHERE (now() - pg_stat_activity.query_start) > interval '5 minutes';

-- MySQL: See active queries
SHOW FULL PROCESSLIST;
```

### Query Logging

```typescript
// Prisma query logging
const prisma = new PrismaClient({
  log: [
    { emit: 'event', level: 'query' },
    { emit: 'stdout', level: 'error' },
    { emit: 'stdout', level: 'warn' }
  ]
});

prisma.$on('query', (e) => {
  console.log('Query:', e.query);
  console.log('Duration:', e.duration + 'ms');
});
```

## Container/Process Debugging

### Docker

```bash
# View logs
docker logs container_name
docker logs -f container_name  # Follow

# Execute command in container
docker exec -it container_name /bin/sh

# Inspect container
docker inspect container_name

# View resource usage
docker stats container_name
```

### Kubernetes

```bash
# View logs
kubectl logs pod_name
kubectl logs -f pod_name  # Follow
kubectl logs pod_name -c container_name  # Specific container

# Execute command in pod
kubectl exec -it pod_name -- /bin/sh

# Describe pod (events, status)
kubectl describe pod pod_name

# Port forward for local debugging
kubectl port-forward pod_name 8080:80
```

### Process Debugging

```bash
# Find process
ps aux | grep node

# Check what's using a port
lsof -i :3000
# or
netstat -tulpn | grep 3000

# Strace: trace system calls
strace -p PID

# Check file descriptors
ls -la /proc/PID/fd

# Check memory maps
cat /proc/PID/maps
```

## Error Tracking

### Sentry Setup

```typescript
import * as Sentry from '@sentry/node';

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 0.1,  // 10% of transactions
});

// Manual capture
try {
  doRiskyOperation();
} catch (error) {
  Sentry.captureException(error, {
    extra: {
      orderId: order.id,
      userId: user.id
    }
  });
  throw error;
}
```

## Tool Selection Guide

| Problem | First Tool | Also Consider |
|---------|------------|---------------|
| Logic error | Debugger | Logging |
| Performance | Profiler | Flame graph |
| Memory leak | Heap snapshot | Memory profiler |
| Network issue | curl / DevTools | Wireshark |
| Database slow | EXPLAIN ANALYZE | Query logging |
| Production error | Error tracking | Logs |
| Intermittent bug | Logging | Tracing |
| Race condition | Debugger + logging | Stress testing |

## Debugging Environment Setup

```json
// package.json scripts
{
  "scripts": {
    "start": "node src/index.js",
    "start:debug": "node --inspect src/index.js",
    "start:debug-brk": "node --inspect-brk src/index.js",
    "profile:cpu": "0x src/index.js",
    "profile:memory": "node --heap-prof src/index.js"
  }
}
```

```bash
# .bashrc / .zshrc debugging aliases
alias debug-node='node --inspect'
alias debug-node-brk='node --inspect-brk'
alias curl-timing='curl -w "@~/.curl-format.txt" -o /dev/null -s'
```
