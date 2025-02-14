---
layout: default
title: Fault tolerance and resilience patterns for Go
---

# Overview

Failsafe-go is a library for building fault tolerant Go applications. It works by wrapping executable logic with one or more resilience [policies], which can be combined and [composed][policy-composition] as needed. Different policies handle different situations, and include:

- Failure handling: [Retry][retry], [Circuit Breaker][circuit-breakers], [Fallback][fallbacks]
- Load limiting: [Circuit Breaker][circuit-breakers], [Bulkhead][bulkheads], [Rate Limiter][rate-limiters]
- Time limiting: [Timeout][timeouts], [Hedge][hedge]

## Getting Started

To see how Failsafe-go works, we'll create a [retry policy][retry] that defines which failures to handle and when retries should be performed:

```go
retryPolicy := retrypolicy.Builder[any]().
  HandleErrors(ErrConnecting).
  WithDelay(time.Second).
  WithMaxRetries(3).
  Build()
```

We can then [Run] or [Get] a result from a `func` with retries:

```go
// Run with retries
err := failsafe.Run(Connect, retryPolicy)

// Get with retries
response, err := failsafe.Get(SendRequest, retryPolicy)
```

### Asynchronous Execution

Executing a `func` [asynchronously][async-execution] with retries is simple:

```go
// Run with retries asynchronously
result := failsafe.RunAsync(Connect, retryPolicy)

// Get with retries asynchronously
result := failsafe.GetAsync(SendRequest, retryPolicy)
```

The returned [ExecutionResult] can be used to wait for the execution to be done and gets its result or error.

### Composing Policies

Multiple [policies] can be composed to add additional layers of resilience or to handle different failures in different ways:

```go
fallback := fallback.WithResult(backupConnection)
circuitBreaker := circuitbreaker.WithDefaults[any]()
timeout := timeout.With[any](10*time.Second)

// Get with fallback, retries, circuit breaker, and timeout
failsafe.Get(Connect, fallback, retryPolicy, circuitBreaker, timeout)
```

Order does matter when composing policies. See the [policy composition][policy-composition] overview for more details.

### Executor

Policy compositions can also be saved for later use via an [Executor]:

```go
executor := failsafe.NewExecutor[any](retryPolicy, circuitBreaker)
err := executor.Run(Connect)
```

## Further Reading

Read more about [policies] and how they're used, then explore some of Failsafe-go's other features in the site menu.

{% include common-links.html %}