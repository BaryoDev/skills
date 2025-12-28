---
name: baryo-observability
description: Production observability standards covering structured logging, distributed tracing, metrics collection, alerting, and error tracking across all platforms.
---

# BaryoDev Observability Standard

You are the SRE. **If you can't observe it, you can't operate it.**

## Core Philosophy

"Production systems must be transparent. Every request tells a story—make sure you can read it."

---

## Part 1: Structured Logging

### Never Use Console Statements in Production

```typescript
// ❌ NEVER: Unstructured logging
console.log('User logged in: ' + userId);

// ✅ ALWAYS: Structured logging
logger.info('User logged in', {
  userId,
  action: 'login',
  ip: request.ip,
  userAgent: request.headers['user-agent'],
  timestamp: new Date().toISOString(),
});
```

### Log Levels

| Level     | When to Use                                 |
| --------- | ------------------------------------------- |
| **ERROR** | System is broken, needs immediate attention |
| **WARN**  | Something unexpected, but system continues  |
| **INFO**  | Normal operations, business events          |
| **DEBUG** | Detailed info for troubleshooting           |
| **TRACE** | Very detailed, usually off in production    |

### Logging Configuration

```typescript
// TypeScript with Pino
import pino from 'pino';

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  formatters: {
    level: (label) => ({ level: label }),
  },
  base: {
    service: 'my-service',
    version: process.env.APP_VERSION,
    environment: process.env.NODE_ENV,
  },
});
```

```csharp
// C# with Serilog
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Information()
    .Enrich.WithProperty("Service", "MyService")
    .Enrich.WithProperty("Version", Assembly.GetExecutingAssembly().GetName().Version)
    .WriteTo.Console(new JsonFormatter())
    .WriteTo.Seq("http://localhost:5341")
    .CreateLogger();
```

```python
# Python with structlog
import structlog

structlog.configure(
    processors=[
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.add_log_level,
        structlog.processors.JSONRenderer()
    ],
    context_class=dict,
    logger_factory=structlog.PrintLoggerFactory(),
)

logger = structlog.get_logger()
logger.info("user_logged_in", user_id=user_id, action="login")
```

---

## Part 2: Distributed Tracing

### Use OpenTelemetry (Language-Agnostic)

**Trace Context**: Every request gets a trace ID that flows through all services.

```typescript
// TypeScript - OpenTelemetry setup
import { trace, context, SpanStatusCode } from '@opentelemetry/api';

const tracer = trace.getTracer('my-service');

async function processOrder(orderId: string) {
  return tracer.startActiveSpan('processOrder', async (span) => {
    try {
      span.setAttribute('order.id', orderId);
      
      await validateOrder(orderId);
      await chargePayment(orderId);
      await shipOrder(orderId);
      
      span.setStatus({ code: SpanStatusCode.OK });
    } catch (error) {
      span.setStatus({ code: SpanStatusCode.ERROR, message: error.message });
      span.recordException(error);
      throw error;
    } finally {
      span.end();
    }
  });
}
```

```csharp
// C# - OpenTelemetry
using var activity = ActivitySource.StartActivity("ProcessOrder");
activity?.SetTag("order.id", orderId);

try {
    await ValidateOrder(orderId);
    await ChargePayment(orderId);
    await ShipOrder(orderId);
    activity?.SetStatus(ActivityStatusCode.Ok);
} catch (Exception ex) {
    activity?.SetStatus(ActivityStatusCode.Error, ex.Message);
    activity?.RecordException(ex);
    throw;
}
```

### Trace Context Propagation

**Always propagate trace context** across HTTP calls:

```typescript
// Propagate trace context in HTTP headers
const headers = {};
propagation.inject(context.active(), headers);

await fetch('http://other-service/api', { headers });
```

---

## Part 3: Metrics Collection

### Key Metrics (RED Method)

| Metric       | What It Measures              |
| ------------ | ----------------------------- |
| **Rate**     | Requests per second           |
| **Errors**   | Error rate (%)                |
| **Duration** | Response time (p50, p95, p99) |

### Prometheus Metrics

```typescript
// TypeScript with prom-client
import { Counter, Histogram, Registry } from 'prom-client';

const registry = new Registry();

const httpRequestsTotal = new Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'path', 'status'],
  registers: [registry],
});

const httpRequestDuration = new Histogram({
  name: 'http_request_duration_seconds',
  help: 'HTTP request duration in seconds',
  labelNames: ['method', 'path'],
  buckets: [0.01, 0.05, 0.1, 0.5, 1, 5],
  registers: [registry],
});

// Middleware
app.use((req, res, next) => {
  const end = httpRequestDuration.startTimer({ method: req.method, path: req.path });
  
  res.on('finish', () => {
    httpRequestsTotal.inc({ method: req.method, path: req.path, status: res.statusCode });
    end();
  });
  
  next();
});
```

### Business Metrics

Track domain-specific metrics:

```typescript
const ordersCreated = new Counter({
  name: 'orders_created_total',
  help: 'Total orders created',
  labelNames: ['plan', 'region'],
});

const revenueTotal = new Counter({
  name: 'revenue_total_usd',
  help: 'Total revenue in USD',
  labelNames: ['plan'],
});
```

---

## Part 4: Health Checks

### Liveness vs Readiness

| Check         | Purpose               | When to Fail             |
| ------------- | --------------------- | ------------------------ |
| **Liveness**  | Is the app alive?     | App is deadlocked/frozen |
| **Readiness** | Can it serve traffic? | Dependencies unavailable |

```typescript
// Health check endpoints
app.get('/health/live', (req, res) => {
  res.json({ status: 'ok' });
});

app.get('/health/ready', async (req, res) => {
  const checks = {
    database: await checkDatabase(),
    cache: await checkCache(),
    externalApi: await checkExternalApi(),
  };
  
  const allHealthy = Object.values(checks).every(c => c.healthy);
  
  res.status(allHealthy ? 200 : 503).json({
    status: allHealthy ? 'ok' : 'degraded',
    checks,
  });
});
```

---

## Part 5: Error Tracking

### Capture Errors with Context

```typescript
// Sentry-like error tracking
import * as Sentry from '@sentry/node';

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 0.1, // 10% of transactions
});

// Add context
Sentry.setUser({ id: userId, email: userEmail });
Sentry.setTag('feature', 'checkout');
Sentry.setContext('order', { orderId, items: cart.length });

// Capture error
try {
  await processPayment();
} catch (error) {
  Sentry.captureException(error);
  throw error;
}
```

---

## Part 6: Alerting

### Alert on Symptoms, Not Causes

```yaml
# Good: Alert on symptoms
- alert: HighErrorRate
  expr: rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) > 0.05
  for: 5m
  labels:
    severity: critical
  annotations:
    summary: "High error rate (> 5%)"

- alert: SlowResponses
  expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 1
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "p95 latency > 1 second"
```

### Alert Severity Levels

| Level        | Response Time    | Example                 |
| ------------ | ---------------- | ----------------------- |
| **Critical** | Immediate (page) | Service down, data loss |
| **Warning**  | Business hours   | High latency, capacity  |
| **Info**     | Next review      | Trends, minor issues    |

---

## Part 7: Dashboards

### Essential Dashboards

1. **Service Health**: Error rate, latency, throughput
2. **Infrastructure**: CPU, memory, disk, network
3. **Business**: Orders, revenue, active users
4. **Dependencies**: Database, cache, external APIs

### Dashboard Template

```
┌─────────────────────────────────────────────────────────────┐
│ Service: order-service                    Last 24h │
├─────────────────┬─────────────────┬─────────────────────────┤
│ Requests/sec    │ Error Rate      │ p95 Latency            │
│ [GRAPH]         │ [GRAPH]         │ [GRAPH]                │
│ 1,234 rps       │ 0.05%           │ 120ms                  │
├─────────────────┴─────────────────┴─────────────────────────┤
│ Top Errors                                                  │
│ 1. TimeoutError (23 occurrences)                           │
│ 2. ValidationError (12 occurrences)                        │
├─────────────────────────────────────────────────────────────┤
│ Dependencies                                                │
│ Database: ✅ 5ms    Cache: ✅ 1ms    Payment API: ⚠️ 250ms  │
└─────────────────────────────────────────────────────────────┘
```

---

## Part 8: Observability Checklist

### Every Service Must Have

- [ ] Structured JSON logging
- [ ] Request/response logging with trace IDs
- [ ] Distributed tracing (OpenTelemetry)
- [ ] RED metrics exposed (/metrics)
- [ ] Health check endpoints (/health/live, /health/ready)
- [ ] Error tracking (Sentry or equivalent)
- [ ] Dashboards in monitoring system
- [ ] Alerts for critical paths

### On Every Request

- [ ] Log request start with trace ID
- [ ] Create trace span
- [ ] Record duration metric
- [ ] Log request completion with status
- [ ] Capture errors with context

---

**Remember**: Observability is not logging. It's the ability to understand your system's state from its outputs.
