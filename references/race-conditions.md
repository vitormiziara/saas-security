# Race Conditions — Protection Patterns

For **any** check-then-act operation, implement race condition protection.
This is mandatory, not optional.

---

## The Core Problem

A race condition occurs when two requests execute simultaneously and both pass a check that should only allow one to succeed:
Request A:  Check balance ($100 >= $100) ✓ → [gap] → Deduct $100
Request B:  Check balance ($100 >= $100) ✓ → [gap] → Deduct $100
Result:     Balance = -$100 (should be $0)

---

## Mandatory Protection Scenarios

Every one of these MUST be explicitly protected:

- Purchases, payments, transfers, and withdrawals
- Coupons, discounts, and single-use promotions
- Refunds and chargebacks
- Likes, votes, counters, and toggles (like/unlike, follow/unfollow)
- Creation of unique resources (username, slug, email)
- Plan upgrades and downgrades
- Single-use links and invitations
- Any financial operation where abuse generates monetary gain

---

## Protection Pattern 1: Atomic Database Transactions

The check and the action MUST be in the same transaction:
```typescript
// WRONG — Race condition possible
const balance = await db.query('SELECT balance FROM wallets WHERE user_id = $1', [userId]);
if (balance.rows[0].balance >= amount) {
  await db.query('UPDATE wallets SET balance = balance - $1 WHERE user_id = $2', [amount, userId]);
}

// CORRECT — Atomic operation, check and action in one statement
const result = await db.query(
  'UPDATE wallets SET balance = balance - $1 WHERE user_id = $2 AND balance >= $1 RETURNING balance',
  [amount, userId]
);
if (result.rowCount === 0) throw new Error('Insufficient balance');
```

---

## Protection Pattern 2: SELECT FOR UPDATE (Row Lock)

For complex operations that require reading before writing:
```typescript
// CORRECT — Lock the row while processing
await db.transaction(async (trx) => {
  const wallet = await trx.query(
    'SELECT balance FROM wallets WHERE user_id = $1 FOR UPDATE',
    [userId]
  );
  
  if (wallet.rows[0].balance < amount) {
    throw new Error('Insufficient balance');
  }
  
  await trx.query(
    'UPDATE wallets SET balance = balance - $1 WHERE user_id = $2',
    [amount, userId]
  );
  
  await trx.query(
    'INSERT INTO transactions (user_id, amount, type) VALUES ($1, $2, $3)',
    [userId, amount, 'debit']
  );
});
```

---

## Protection Pattern 3: UNIQUE CONSTRAINTS

Use database constraints as a defense layer for truly unique operations:
```sql
-- Coupon: one use per user
ALTER TABLE coupon_redemptions ADD CONSTRAINT unique_user_coupon 
  UNIQUE (user_id, coupon_id);

-- Like: one like per user per post
ALTER TABLE likes ADD CONSTRAINT unique_user_post_like 
  UNIQUE (user_id, post_id);

-- Invitation: one-time use
ALTER TABLE invitations ADD COLUMN used_at TIMESTAMP;
ALTER TABLE invitations ADD CONSTRAINT unique_used_invitation 
  UNIQUE (token) WHERE used_at IS NOT NULL;
```

---

## Protection Pattern 4: Idempotency Keys

For critical operations, the same request repeated = the same result:
```typescript
// Client sends a unique key with every critical operation
POST /api/payments
Headers: Idempotency-Key: uuid-v4-generated-client-side
Body: { amount: 100, currency: "USD" }

// Server implementation
async function processPayment(idempotencyKey: string, payload: PaymentPayload) {
  const existing = await db.query(
    'SELECT result FROM idempotency_cache WHERE key = $1 AND expires_at > NOW()',
    [idempotencyKey]
  );
  
  if (existing.rows.length > 0) {
    return JSON.parse(existing.rows[0].result);
  }
  
  const result = await executePayment(payload);
  
  await db.query(
    "INSERT INTO idempotency_cache (key, result, expires_at) VALUES ($1, $2, NOW() + INTERVAL '24 hours')",
    [idempotencyKey, JSON.stringify(result)]
  );
  
  return result;
}
```

---

## Protection Pattern 5: Optimistic Locking

For non-financial resources where conflicts are rare:
```typescript
ALTER TABLE documents ADD COLUMN version INTEGER DEFAULT 1;

const result = await db.query(
  'UPDATE documents SET content = $1, version = version + 1 WHERE id = $2 AND version = $3 RETURNING version',
  [newContent, documentId, expectedVersion]
);

if (result.rowCount === 0) {
  throw new ConflictError('Document was modified by another request. Please retry.');
}
```

---

## Testing Race Conditions
```typescript
test('coupon cannot be used twice simultaneously', async () => {
  const couponId = await createSingleUseCoupon();
  const userId = await createUser();
  
  const results = await Promise.allSettled(
    Array(10).fill(null).map(() => 
      request.post('/api/coupons/redeem').send({ couponId, userId })
    )
  );
  
  const successes = results.filter(r => r.status === 'fulfilled' && r.value.status === 200);
  expect(successes).toHaveLength(1);
});
```

---

## Quick Reference: What to Use When

| Scenario | Primary Protection | Secondary Protection |
|---|---|---|
| Financial debit/credit | Atomic UPDATE with WHERE condition | SELECT FOR UPDATE in transaction |
| Single-use coupon | UNIQUE CONSTRAINT (user_id, coupon_id) | Atomic conditional UPDATE |
| Payment processing | Idempotency key | UNIQUE CONSTRAINT on transaction ID |
| Like/unlike toggle | UNIQUE CONSTRAINT + INSERT ON CONFLICT | Atomic UPDATE |
| Unique username | UNIQUE CONSTRAINT on username | Check + INSERT in transaction |
| Inventory deduction | `UPDATE SET stock = stock - 1 WHERE stock >= 1` | SELECT FOR UPDATE |
| Refund request | Idempotency key + status check | UNIQUE CONSTRAINT on order_id |
