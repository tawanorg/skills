---
title: Read Error Messages
impact: critical
tags: [debugging, fundamentals, errors]
---

# Read Error Messages

**Actually read the error message. The answer is often right there.**

## The Rule

Before doing anything else:
1. Read the entire error message
2. Read the stack trace
3. Understand what it's telling you
4. Only then start investigating

## Why This Matters

```
90% of developers glance at errors without reading them.
80% of errors tell you exactly what's wrong.

Time spent reading error: 30 seconds
Time spent guessing without reading: 30 minutes
```

## Incorrect Approach

```javascript
// Error appears
// Developer glances at it: "TypeError... something undefined..."
// Developer: "Ugh, let me add some console.logs"

console.log('data', data);
console.log('user', user);
console.log('items', items);
// 10 minutes later, still confused
```

## Correct Approach

```javascript
// Error:
// TypeError: Cannot read properties of undefined (reading 'map')
//     at UserList (src/components/UserList.tsx:23:18)
//     at renderWithHooks (node_modules/react-dom/...)
//     at mountIndeterminateComponent (node_modules/react-dom/...)

// Read and parse:
// 1. TYPE: TypeError - type-related issue
// 2. WHAT: Cannot read 'map' of undefined
//          → Something is undefined that shouldn't be
//          → We're calling .map() on it
// 3. WHERE: UserList.tsx, line 23, column 18
// 4. WHEN: During React render (renderWithHooks)

// Go directly to UserList.tsx:23
// Line 23: {users.map(user => <UserCard user={user} />)}
//                 ^^^
//          users is undefined

// Root cause found in 30 seconds
```

## Anatomy of an Error Message

```
TypeError: Cannot read properties of undefined (reading 'email')
│          │                                           │
│          │                                           └── Property accessed
│          └── What happened
└── Error type (helps categorize)

    at getUserEmail (src/utils/user.ts:45:12)
    │               │                  │  │
    │               │                  │  └── Column
    │               │                  └── Line number
    │               └── File path
    └── Function name

    at processUsers (src/services/userService.ts:123:5)
    at async handler (src/api/users.ts:67:3)
    │
    └── Call stack (read bottom-to-top for flow)
```

## Error Message Types

| Error Type | Meaning | First Check |
|------------|---------|-------------|
| TypeError | Wrong type operation | Null/undefined value |
| ReferenceError | Using undefined variable | Typo, scope, import |
| SyntaxError | Invalid code structure | Missing bracket, comma |
| RangeError | Value out of range | Array index, recursion |
| NetworkError | Network request failed | URL, connectivity, CORS |
| TimeoutError | Operation took too long | Performance, deadlock |

## Common Patterns

```javascript
// "X is not a function"
// → X exists but is wrong type
const data = await fetchData();  // returns object, not function
data();  // Error: data is not a function

// "Cannot read property 'X' of undefined"
// → Parent object is undefined
user.profile.settings  // user.profile is undefined

// "X is not defined"
// → Variable doesn't exist in scope
console.log(userData);  // Did you mean 'user'? Import missing?

// "Maximum call stack exceeded"
// → Infinite recursion
function bad() { bad(); }  // Check termination condition
```

## Reading Stack Traces

```javascript
Error: Database connection failed
    at connectDB (src/db/connect.ts:34:11)        // ← Start here
    at initializeApp (src/app.ts:12:3)
    at main (src/index.ts:5:1)

// Read top-to-bottom: Where error was thrown
// Read bottom-to-top: How we got there

// This tells us:
// 1. main() called initializeApp()
// 2. initializeApp() called connectDB()
// 3. connectDB() threw the error at line 34
// 4. → Go to src/db/connect.ts:34 to investigate
```

## The 60-Second Rule

Spend at least 60 seconds reading the error before:
- Adding console.logs
- Searching online
- Asking for help

Most bugs are solved within that 60 seconds of careful reading.
