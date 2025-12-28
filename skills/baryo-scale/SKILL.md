---
name: baryo-scale
description: Scalability patterns for caching, database optimization, and horizontal scaling.
---

# BaryoDev Scale Standard

You are the Performance Architect. **Build for 10x your current load.**

## Caching Strategy

### Cache Layers
- Browser (1ms) → CDN (10ms) → Redis (5ms) → Database (50ms)

### Cache-Aside Pattern
```typescript
async function getUser(id: string): Promise<User> {
  const cached = await cache.get(`user:${id}`);
  if (cached) return JSON.parse(cached);
  
  const user = await db.users.findUnique({ where: { id } });
  await cache.set(`user:${id}`, JSON.stringify(user), 'EX', 3600);
  return user;
}
```

## Database Optimization

### Indexing
```sql
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at DESC);
```

### Avoid N+1 Queries
```typescript
// ❌ N+1
for (const user of users) {
  const orders = await db.orders.findMany({ where: { userId: user.id } });
}

// ✅ Eager loading
const users = await db.users.findMany({ include: { orders: true } });
```

## Horizontal Scaling

### Stateless Services
- No in-memory sessions
- Use Redis for shared state
- All instances interchangeable

### Rate Limiting (Token Bucket)
```typescript
async function checkRateLimit(userId: string, limit: number): Promise<boolean> {
  const count = await redis.incr(`ratelimit:${userId}`);
  if (count === 1) await redis.expire(`ratelimit:${userId}`, 60);
  return count <= limit;
}
```

## Performance Budgets
- API p95: 200ms
- API p99: 500ms
- Page load: 3s
- JS bundle: 200kb

## Checklist
- [ ] Load tested at 10x traffic
- [ ] Database indexed
- [ ] Caching implemented
- [ ] Connection pools configured
- [ ] CDN for static assets
- [ ] Autoscaling configured
