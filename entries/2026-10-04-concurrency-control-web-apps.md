# ⚖️ Optimistic vs. Pessimistic Concurrency Control

## Optimistic Concurrency Control (OCC)
Assume conflict is rare. Record a `version` or `updated_at` column:
```sql
UPDATE inventory 
SET stock = stock - 1, version = version + 1 
WHERE id = 42 AND version = 3;
```
If 0 rows are updated, conflict occurred; reload and retry.

## Pessimistic Concurrency Control (PCC)
Assume conflict is frequent. Lock rows upfront:
```sql
SELECT * FROM inventory WHERE id = 42 FOR UPDATE;
```
Guarantees serial access but increases lock contention and deadlock risks.
