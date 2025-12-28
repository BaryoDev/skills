---
name: baryo-resilience
description: Disaster recovery, business continuity, and system resilience patterns.
---

# BaryoDev Resilience Standard

You are the Reliability Engineer. **Plan for failure, design for recovery.**

## Circuit Breaker Pattern
```typescript
enum CircuitState { Closed, Open, HalfOpen }

class CircuitBreaker {
  private failures = 0;
  private state = CircuitState.Closed;
  
  async execute<T>(fn: () => Promise<T>): Promise<T> {
    if (this.state === CircuitState.Open) {
      throw new Error('Circuit open');
    }
    
    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }
}
```

## Retry with Backoff
```typescript
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await sleep(Math.pow(2, i) * 1000 + Math.random() * 1000);
    }
  }
  throw new Error('Unreachable');
}
```

## Backup Strategy
- **RPO** (Recovery Point Objective): Max data loss acceptable
- **RTO** (Recovery Time Objective): Max downtime acceptable

### 3-2-1 Rule
- 3 copies of data
- 2 different storage types
- 1 offsite copy

## Disaster Recovery Plan

1. **Detection**: Monitoring alerts
2. **Assessment**: Determine scope
3. **Activation**: Execute DR plan
4. **Recovery**: Restore services
5. **Validation**: Verify functionality
6. **Review**: Document lessons learned

## Zero-Downtime Deployments
```yaml
# Rolling deployment
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 25%
    maxUnavailable: 0
```

## Checklist
- [ ] Backups automated and tested
- [ ] DR plan documented
- [ ] Failover tested quarterly
- [ ] Circuit breakers on external calls
- [ ] Retries with jitter
- [ ] Health checks comprehensive
- [ ] Rollback procedure documented
