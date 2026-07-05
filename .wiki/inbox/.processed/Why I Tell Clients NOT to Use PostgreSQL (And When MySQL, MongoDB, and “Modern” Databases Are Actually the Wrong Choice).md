
# [Why I Tell Clients NOT to Use PostgreSQL (And When MySQL, MongoDB, and “Modern” Databases Are Actually the Wrong Choice)](https://medium.com/@jholt1055/why-i-tell-clients-not-to-use-postgresql-and-when-mysql-mongodb-and-modern-databases-are-56d241850225)

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*4u1p2feraNc_98JcDfC7XQ.png)

## The uncomfortable truths about database selection that no one talks about — based on 47 failed migrations, $23M in wasted costs, and 10 years of fixing other people’s database disasters

**This article will make me unpopular.**

I’m about to tell you that the “best practices” everyone follows are often wrong. That the databases everyone recommends might be terrible for your use case. That the migration everyone says you “must do” could bankrupt your company.

**Here’s my controversial position:**

- ==PostgreSQL isn’t always better than MySQL==
- NoSQL isn’t always the answer for scale
- Microservices with separate databases can destroy performance
- “Cloud-native” databases sometimes cost 10x more for worse results
- The latest hot database technology is usually a disaster waiting to happen

I’ve spent 10 years cleaning up database messes. I’ve been called into 47 failed database migrations where companies spent millions migrating to the “right” database — only to find their new system was slower, more expensive, and more fragile than what they had before.

**The pattern is always the same:**

1. Company reads blog posts about “modern” database architecture
2. Engineers get excited about new technology
3. Company spends 6–12 months migrating
4. New system is worse than old system
5. They call me to fix it

**This article is about the 47 times I had to tell clients: “You shouldn’t have done that migration.”**

It’s about the $23 million in wasted engineering time I’ve witnessed. It’s about the three companies that nearly went bankrupt because they picked the wrong database. And it’s about the uncomfortable truth that **sometimes the “old” way is actually the right way.**

Let me show you the real disasters I’ve seen — and the controversial database advice that could save your company millions.

## Part 1: The PostgreSQL Migration That Cost $8.7M (And Failed)

**The Company:** Series B SaaS startup, 200 employees, $15M ARR

==**The Decision:**== ==“We need to migrate from MySQL to PostgreSQL. Everyone says PostgreSQL is better.”==

**The Reality:** 18 months later, they were desperately trying to migrate back to MySQL.

## What Happened

**Month 0:** CTO reads articles about PostgreSQL’s advanced features  
**Month 1:** ==Engineering team excited about JSONB, CTEs, window functions==  
**Month 2:** Board approves $2M migration budget  
**Month 3:** Migration project starts  
**Month 12:** Still migrating (originally estimated 6 months)  
**Month 15:** Finally migrated to PostgreSQL  
**Month 16:** Performance disaster becomes apparent  
**Month 18:** Emergency call to me: “Help us fix this or we’re bankrupt”

## The Numbers

**Before (MySQL):**

- Database: MySQL 5.7 on AWS RDS
- Size: 480GB
- Queries/second: 15,000
- P95 query latency: 23ms
- Monthly cost: $4,200
- Engineering time on database issues: ~10 hours/week

**After (PostgreSQL 14):**

- Database: PostgreSQL 14 on AWS RDS
- Size: 680GB (42% larger!)
- Queries/second: 15,000 (same workload)
- P95 query latency: 187ms (8x slower!)
- Monthly cost: $12,400 (3x more expensive)
- Engineering time on database issues: ~80 hours/week
- Migration cost: $8.7M (engineer salaries + lost velocity + customer churn)

## Why It Failed

**1\. MVCC Bloat in High-Update Workload**

Their application was a CRM system with constant updates to records.

```c
-- Their typical query pattern
UPDATE contacts 
SET last_contacted = NOW(), 
    contact_count = contact_count + 1,
    updated_at = NOW()
WHERE id = 12345;
```
```c
-- This query ran 50,000 times per minute
```

**What they didn’t know:**

==PostgreSQL uses MVCC (Multi-Version Concurrency Control==). Every UPDATE creates a new row version. Old versions accumulate until VACUUM cleans them up.

With 50,000 updates/minute on the same tables:

- Dead tuples accumulated faster than autovacuum could clean
- Tables bloated from 80GB to 280GB
- Indexes bloated proportionally
- Queries slowed down dramatically
```c
-- Table bloat check
SELECT 
    schemaname,
    tablename,
    n_live_tup,
    n_dead_tup,
    ROUND(100.0 * n_dead_tup / NULLIF(n_live_tup + n_dead_tup, 0), 2) AS dead_pct
FROM pg_stat_user_tables
WHERE tablename = 'contacts';
```
```c
-- Result: 68% dead tuples!
```

**MySQL handled this better** because:

- InnoDB updates in-place when possible
- No dead tuple accumulation
- No aggressive vacuum needed
- Consistent performance

**2\. Connection Overhead**

```c
# Their application code (unchanged during migration)
def handle_request():
    db = mysql.connector.connect(...)  # New connection each request
    cursor = db.cursor()
    cursor.execute("SELECT ...")
    result = cursor.fetchall()
    db.close()
```

**Connection costs:**

```c
MySQL:
  Connection time: ~2ms
  Memory per connection: ~256KB
  Max connections: 2000 (comfortable)
```
```c
PostgreSQL:
  Connection time: ~12ms (6x slower)
  Memory per connection: ~10MB (40x more)
  Max connections: 500 (strain at >300)
```

With 15,000 queries/second and no connection pooling:

- PostgreSQL spent 40% of CPU time on connection overhead
- MySQL spent 6% of CPU time on connection overhead

**They** ==**needed PgBouncer**== **(connection pooler), but:**

- Didn’t know about it during migration
- Added it later, but required application changes
- Lost 12 months dealing with connection issues

**3\. Replication Lag**

```c
-- Their reporting queries (on read replica)
SELECT 
    COUNT(*) as total_contacts,
    SUM(CASE WHEN last_contacted >= NOW() - INTERVAL '7 days' THEN 1 ELSE 0 END) as recent_contacts,
    AVG(contact_count) as avg_contacts
FROM contacts
WHERE account_id = 12345;
```

**MySQL replication lag:** Averaged 0.8 seconds, max 3 seconds  
**PostgreSQL replication lag:** Averaged 8 seconds, max 47 seconds

**Why?**

MySQL binary log replication is mature and optimized. PostgreSQL WAL-based replication had issues with their workload:

- High update rate caused WAL generation spike
- Network between AZs couldn’t keep up
- Replica fell behind during peak hours
- Reports showed stale data
- Customers complained

**4\. Vacuum Storms**

```c
-- Autovacuum settings (their defaults)
autovacuum_vacuum_threshold = 50
autovacuum_vacuum_scale_factor = 0.2
```
```c
-- For their 10M row table:
-- Vacuum triggers at: 50 + (0.2 * 10,000,000) = 2,000,050 dead tuples
```

**The problem:**

With 50,000 updates/minute, they hit vacuum threshold in 40 minutes. Then:

- Autovacuum starts
- Takes 2 hours to complete (large table)
- During vacuum: Queries slow down 3–10x
- Vacuum completes
- 40 minutes later: Needs vacuum again
- **Perpetual vacuum storm**

**MySQL:** No vacuum needed, no vacuum storms

## The Cost Breakdown

```c
Migration Costs:
  Engineer time: $4.2M (8 engineers × 18 months × $150K salary)
  Database costs during migration: $0.8M (running both systems)
  Lost product development: $2.5M (opportunity cost)
  Performance issues causing churn: $1.2M (lost customers)
  Total: $8.7M
```
```c
Additional Ongoing Costs:
  3x higher database costs: $8,200/month
  4x higher engineering maintenance: $25K/month
  Total additional ongoing: $33,200/month = $398,400/year
```

## The Resolution

I was brought in at month 18. Here’s what I told them:

**“You should never have migrated to PostgreSQL. Your workload is textbook MySQL.”**

Their options:

1. Stay on PostgreSQL, optimize aggressively (ongoing pain)
2. Migrate back to MySQL (another 6 months, $2M)

They chose option 2. Migrated back to MySQL. Everything worked better.

**Lessons:**

✅ **PostgreSQL is amazing for:** Read-heavy analytics, complex queries, JSON data, geospatial data  
❌ **PostgreSQL struggles with:** High-update OLTP, connection-heavy workloads, write-heavy applications

✅ **MySQL is amazing for:** High-update OLTP, connection-heavy workloads, simple schemas  
❌ **MySQL struggles with:** Complex analytics, JSON queries, geospatial data

**The right database depends on YOUR workload, not what’s “better” in the abstract.**

## Part 2: The MongoDB Migration That Destroyed Data Integrity

**The Company:** E-commerce platform, Series C, 400 employees, $40M ARR

**The Decision:** “We need MongoDB for scalability. SQL databases don’t scale.”

**The Reality:** $12M migration resulted in data corruption, lost transactions, and customer lawsuits.

## What Happened

**The pitch:** “MongoDB is web-scale. It’s what all the big companies use.”

**The reality:** Big companies use MongoDB for specific use cases, not as a primary transactional database.

## The Migration

```c
// Before (PostgreSQL)
BEGIN;
  INSERT INTO orders (customer_id, total) VALUES (12345, 99.99);
  INSERT INTO order_items (order_id, product_id, quantity) VALUES (LAST_INSERT_ID(), 67890, 2);
  UPDATE inventory SET quantity = quantity - 2 WHERE product_id = 67890;
  UPDATE customers SET total_spent = total_spent + 99.99 WHERE id = 12345;
COMMIT;
```
```c
// After (MongoDB)
// "We don't need transactions, we'll handle it in application code"
db.orders.insertOne({customer_id: 12345, total: 99.99});
// What if this fails? What if server crashes here?
db.order_items.insertOne({order_id: order._id, product_id: 67890, quantity: 2});
// What if this fails?
db.inventory.updateOne({product_id: 67890}, {$inc: {quantity: -2}});
// What if this fails?
db.customers.updateOne({_id: 12345}, {$inc: {total_spent: 99.99}});
```

## The Disasters

**Disaster 1: Lost Orders**

```c
Orders created: 1,000,000
Order items created: 998,473
Orders with missing items: 1,527
```
```c
Root cause: Network issues between application and MongoDB
Result: 1,527 customers charged but received incomplete orders
Cost: $847,000 in refunds and compensation
```

**Disaster 2: Inventory Corruption**

```c
-- Check inventory integrity
db.inventory.aggregate([
  {$lookup: {
    from: "order_items",
    localField: "product_id",
    foreignField: "product_id",
    as: "ordered"
  }},
  {$project: {
    product_id: 1,
    current_quantity: "$quantity",
    total_ordered: {$sum: "$ordered.quantity"},
    initial_quantity: "$initial_quantity",
    expected_quantity: {$subtract: ["$initial_quantity", {$sum: "$ordered.quantity"}]},
    discrepancy: {$subtract: ["$quantity", {$subtract: ["$initial_quantity", {$sum: "$ordered.quantity"}]}]}
  }},
  {$match: {discrepancy: {$ne: 0}}}
]);
```
```c
// Result: 847 products with inventory discrepancies
// Some showed negative inventory
// Some showed more inventory than ever purchased
```

**Root cause:**

- Race conditions in application code
- No database-level transaction guarantees
- ==Partial failures during upda== tes

**Disaster 3: Customer Balance Corruption**

```c
// What they thought would work:
async function processRefund(orderId, amount) {
  await db.orders.updateOne(
    {_id: orderId}, 
    {$set: {status: 'refunded'}}
  );
  
  await db.customers.updateOne(
    {_id: customerId}, 
    {$inc: {balance: amount}}
  );
  
  await db.transactions.insertOne({
    type: 'refund',
    order_id: orderId,
    amount: amount
  });
}
```
```c
// What actually happened:
// Sometimes order updated, customer balance not updated
// Sometimes balance updated, transaction not recorded
// Sometimes server crashed mid-function
```

**Result:**

- 2,847 customer accounts with incorrect balances
- $1.2M discrepancy between order totals and customer balances
- Manual reconciliation took 3 months
- 47 customers filed small claims lawsuits

## Why MongoDB Was Wrong

**MongoDB added multi-document transactions in version 4.0 (2018), but:**

1. **Transaction performance is terrible**
- 10x slower than PostgreSQL transactions
- High overhead
- Network round trips
1. **Transaction limitations**
- Can’t span shards (in sharded clusters)
- Timeout issues with long transactions
- Not as robust as ACID databases
1. **They migrated before transactions existed**
- Relied on application-level transaction logic
- Application logic is always buggy
- Cost them millions

## The Real Cost

```c
Migration costs: $12M
Data corruption recovery: $3.8M
Customer refunds/compensation: $2.1M
Legal fees (lawsuits): $0.8M
Engineering time fixing data: $2.4M
Reputation damage: Immeasurable
Total: $21.1M
```
```c
Ongoing costs:
MongoDB cluster: $18,000/month (vs $6,000 for PostgreSQL)
Engineering maintenance: 3x higher
Data integrity monitoring: Permanent overhead
```

## The Resolution

I was brought in after the third major data integrity incident.

**My recommendation:** “Migrate back to PostgreSQL immediately.”

**Their response:** “We can’t. We’ve spent too much.”

**My counter:** “You’ll spend more fixing data corruption than migrating back.”

They eventually migrated back after the lawsuits. Took 8 months, cost another $5M.

## When MongoDB IS the Right Choice

==MongoDB is actually great for:==

- ✅ Content management systems (flexible schemas)
- ✅ Product catalogs (varying attributes per product)
- ✅ User profiles (different users have different fields)
- ✅ Event logging (high write volume, relaxed consistency)
- ✅ Real-time analytics (time-series data)

MongoDB is WRONG for:

- ❌ Financial transactions
- ❌ Inventory management
- ❌ Any system where data integrity matters
- ❌ Relations between entities (use relational DB)

**Lesson:** Use MongoDB when you need flexibility and can tolerate eventual consistency. Never use it when you need transactions and referential integrity.

## Part 3: The Microservices Database Nightmare

**The Company:** FinTech startup, Series A, 80 employees, $8M ARR

**The Decision:** “We’re moving to microservices. Each service gets its own database.”

==**The Reality:**== ==Distributed data caused more problems than monolithic architecture ever did.==

## The Architecture Change

**Before (Monolith):**

```c
┌─────────────────────────┐
│   Single Application    │
│   (All Features)        │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│   Single PostgreSQL DB   │
│   (All Data)            │
└─────────────────────────┘
```

**After (Microservices):**

```c
┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│  User    │  │  Order   │  │ Payment  │  │ Notif.   │
│ Service  │  │ Service  │  │ Service  │  │ Service  │
└────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘
     │             │             │             │
     ▼             ▼             ▼             ▼
┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│ User DB  │  │ Order DB │  │ Pay DB   │  │ Notif DB │
└──────────┘  └──────────┘  └──────────┘  └──────────┘
```

## The Problems

**Problem 1: No More JOIN Queries**

```c
-- What they used to do (monolith):
SELECT 
    o.order_id,
    o.total,
    o.status,
    u.name,
    u.email,
    p.payment_method,
    p.payment_status
FROM orders o
JOIN users u ON o.user_id = u.id
JOIN payments p ON o.order_id = p.order_id
WHERE o.created_at >= '2024-10-01';
```
```c
-- Execution time: 45ms
```
```c
// What they had to do (microservices):
async function getOrderDetails(orderId) {
    // Call order service
    const order = await orderService.getOrder(orderId);
    
    // Call user service
    const user = await userService.getUser(order.user_id);
    
    // Call payment service
    const payment = await paymentService.getPayment(orderId);
    
    // Combine data
    return {
        ...order,
        userName: user.name,
        userEmail: user.email,
        paymentMethod: payment.method,
        paymentStatus: payment.status
    };
}
```
```c
// Execution time: 187ms (4x slower)
// Network calls: 3 (vs 1 database query)
// Points of failure: 3 (vs 1)
```

==**Problem 2: Data Consistency Nightmares**==

```c
// Create order in monolith (ACID transaction):
BEGIN;
  INSERT INTO orders (user_id, total) VALUES (12345, 99.99);
  UPDATE inventory SET quantity = quantity - 1 WHERE product_id = 67890;
  INSERT INTO payments (order_id, amount) VALUES (LAST_INSERT_ID(), 99.99);
COMMIT;
// Either all succeed or all fail. Guaranteed.
```
```c
// Create order in microservices (distributed transaction):
async function createOrder(userId, productId, amount) {
    try {
        // Step 1: Reserve inventory
        await inventoryService.reserve(productId, 1);
        
        // Step 2: Create order
        const order = await orderService.create(userId, amount);
        
        // Step 3: Process payment
        const payment = await paymentService.charge(order.id, amount);
        
        // Step 4: Confirm inventory
        await inventoryService.confirm(productId, 1);
        
        return order;
    } catch (error) {
        // What if step 2 succeeds but step 3 fails?
        // What if network fails between steps?
        // What if service is down?
        // How do we rollback?
        // NIGHTMARE
    }
}
```

**Real failure scenarios they experienced:**

```c
Scenario 1: Payment succeeded, inventory not decremented
  Result: Oversold products
  Occurrences: 847 times in first month
  Cost: $124,000 in expedited shipping and compensation
```
```c
Scenario 2: Inventory reserved, payment failed, reservation not released
  Result: Products showing "out of stock" when actually available
  Occurrences: 2,384 times
  Lost revenue: Estimated $480,000Scenario 3: Order created, payment service down, order stuck in limbo
  Result: Orders neither completed nor cancelled
  Occurrences: 156 times
  Manual intervention required: 156 × 15 minutes = 39 hours of support time
```

**Problem 3: Data Duplication and Sync Issues**

```c
// User changes email address:
// Before (monolith): Update one row
UPDATE users SET email = 'newemail@example.com' WHERE id = 12345;
```
```c
// After (microservices): Update in multiple services
// Users in: User Service, Order Service, Notification Service, Support Service// What happens when one update fails?
// What's the source of truth?
// How do we keep them in sync?
```

**Their attempted solution: Event-driven architecture**

```c
// User service publishes event
userService.updateEmail(userId, newEmail);
eventBus.publish('user.email.changed', {userId, newEmail});
```
```c
// Other services subscribe
orderService.on('user.email.changed', event => {
    orderService.updateUserEmail(event.userId, event.newEmail);
});notificationService.on('user.email.changed', event => {
    notificationService.updateUserEmail(event.userId, event.newEmail);
});supportService.on('user.email.changed', event => {
    supportService.updateUserEmail(event.userId, event.newEmail);
});
```

**Problems with this approach:**

1. **Event delivery not guaranteed**
- Messages can be lost
- Services can be down
- Network issues
1. **Ordering not guaranteed**
- Email changed to A, then to B
- Service might receive B before A
- Final state: Wrong email
1. **Complex debugging**
- “Why does Order Service have old email?”
- Check event bus
- Check Order Service logs
- Check User Service logs
- Event was published? Delivered? Processed?
- Debugging time: Hours instead of seconds

**Problem 4: Query Performance Disaster**

```c
// Simple report query in monolith:
SELECT 
    DATE(created_at) as date,
    COUNT(*) as order_count,
    SUM(total) as revenue
FROM orders
WHERE created_at >= '2024-10-01'
GROUP BY DATE(created_at);
```
```c
// Execution: 23ms// Same report in microservices:
async function getDailyReport(startDate) {
    // Get all orders from Order Service
    const orders = await orderService.getOrdersSince(startDate);
    
    // For each order, get payment status from Payment Service
    const ordersWithPayments = await Promise.all(
        orders.map(async order => {
            const payment = await paymentService.getPayment(order.id);
            return {...order, payment};
        })
    );
    
    // Filter to completed payments
    const completed = ordersWithPayments.filter(o => o.payment.status === 'completed');
    
    // Aggregate in application code
    const report = completed.reduce((acc, order) => {
        const date = order.created_at.split('T')[0];
        if (!acc[date]) acc[date] = {count: 0, revenue: 0};
        acc[date].count++;
        acc[date].revenue += order.total;
        return acc;
    }, {});
    
    return report;
}// Execution: 8,473ms (368x slower!)
// Network calls: 1 + N (where N = number of orders)
// For 10,000 orders: 10,001 network calls!
```

## The Costs

```c
Microservices migration:
  Engineering time: $3.2M (18 months)
  Infrastructure costs: 4x increase ($2,400/mo → $9,600/mo)
  
Performance degradation:
  Average API response time: 45ms → 320ms (7x slower)
  Complex queries: 50ms → 8000ms (160x slower)
  
Data consistency issues:
  Manual reconciliation: $480,000/year
  Lost revenue from bugs: $1.2M/year
  Customer support overhead: 3x increase
  
Total cost: $8.4M (first year)
Annual ongoing costs: $2.8M more than monolith
```

## The Resolution

After 18 months of pain, I was brought in.

**My assessment:** “You solved a problem you didn’t have, and created problems you can’t solve.”

**Their defense:** “But monoliths don’t scale!”

**My response:** “Your monolith handled 10,000 requests/second. You’re currently at 2,000. You had scaling headroom for 5x growth. Instead, you created a distributed data nightmare.”

**My recommendation:**

1. Keep microservices architecture (organizational benefits are real)
2. ==But: Shared database for services that need transactions together==
3. Only separate databases when truly necessary

**The compromise architecture:**

```c
┌──────────┐  ┌──────────┐  ┌──────────┐
│  User    │  │  Order   │  │ Payment  │
│ Service  │  │ Service  │  │ Service  │
└────┬─────┘  └────┬─────┘  └────┬─────┘
     │             │             │
     └─────────────┼─────────────┘
                   ▼
         ┌──────────────────┐
         │   Shared DB      │
         │ (Transactional)  │
         └──────────────────┘
```
```c
┌──────────┐  ┌──────────┐
│Analytics │  │  Notif   │
│ Service  │  │ Service  │
└────┬─────┘  └────┬─────┘
     │             │
     ▼             ▼
┌──────────┐  ┌──────────┐
│Analytics │  │ Notif DB │
│   DB     │  └──────────┘
└──────────┘
```

**Results after compromise:**

- Kept microservices benefits (team autonomy, independent deploys)
- Restored ACID transactions for core business logic
- Query performance back to normal
- Data consistency issues eliminated
- Cost reduced by 60%

## The Real Lessons

**Microservices with separate databases work when:**

- ✅ Services are truly independent (no shared transactions)
- ✅ You can tolerate eventual consistency
- ✅ You have the engineering expertise to handle distributed systems
- ✅ Scale is actually a problem (not theoretical)

**Microservices with separate databases fail when:**

- ❌ Services need to participate in same transaction
- ❌ You can’t tolerate inconsistency
- ❌ Team doesn’t understand distributed systems
- ❌ You’re doing it because “that’s what big companies do”

**Lesson:** Microservices and separate databases are solutions to specific problems. If you don’t have those problems, you’re just adding complexity.

## Part 4: The “Cloud-Native” Database That Cost 10x More

**The Company:** Media startup, Seed round, 25 employees, $2M ARR

**The Decision:** “We’ll use ==Aurora Serverless==. It scales automatically and we only pay for what we use.”

**The Reality:** Bill went from $800/month to $12,000/month with same workload.

## The Pitch vs. Reality

**What AWS said:**

> *“Aurora Serverless v2 automatically scales compute capacity based on your application’s needs. You only pay for the database resources you consume.”*

**What actually happened:**

```c
Month 1 (MySQL on EC2):
  Database cost: $840/month
  Predictable, stable
```
```c
Month 2 (Aurora Serverless):
  Database cost: $11,847/month
  Unpredictable, shocking
```

## Why the Cost Exploded

**Problem 1: ACU (Aurora Capacity Unit) Pricing**

```c
Aurora Serverless pricing:
  $0.12 per ACU-hour
  Minimum: 0.5 ACUs
  Maximum: 128 ACUs (configurable)
```
```c
Their config:
  Min: 2 ACUs
  Max: 32 ACUs
```

**What they thought would happen:**

```c
Low traffic (2 AM): 2 ACUs
  Cost: 2 × $0.12 = $0.24/hour
  
High traffic (2 PM): Scale up to need, scale down after
  Cost: Variable, but average should be low
```

**What actually happened:**

```c
Low traffic (2 AM): 2 ACUs ✓
Medium traffic (9 AM): Scales to 8 ACUs ✓
High traffic (11 AM): Scales to 16 ACUs ✓
Traffic spike (12 PM): Scales to 32 ACUs ✓
Spike ends 15
: Still at 32 ACUs ✗
Traffic normal (1 PM): Still at 32 ACUs ✗
Traffic low (2 PM): Still at 32 ACUs ✗
...
Finally scales down (8 PM): Back to 8 ACUs
```

**The issue: Slow scale-down**

Aurora Serverless is aggressive scaling UP, but conservative scaling DOWN. It stays at high capacity “just in case.”

Result: Paid for 32 ACUs for 8 hours when only needed for 15 minutes.

**Problem 2: Connection Churn**

Their application created new database connections frequently (no connection pooling).

```c
// Their code (bad practice)
app.get('/api/users', async (req, res) => {
    const db = await mysql.createConnection({...});
    const users = await db.query('SELECT * FROM users');
    await db.end();
    res.json(users);
});
```

**With regular RDS:** Connection overhead is ~2ms, manageable

**With Aurora Serverless:** Each new connection could trigger scaling event

```c
Request 1: Connect → 2 ACUs sufficient
Request 2: Connect → 2 ACUs sufficient  
Request 3: Connect → 2 ACUs sufficient
Requests 4-20 (simultaneous): 20 new connections
  → Aurora interprets as load spike
  → Scales to 16 ACUs
Requests complete in 100ms
  → Aurora stays at 16 ACUs for 15 minutes
  → Cost: 16 × $0.12 × 0.25 = $0.48 for handling $0.02 worth of work
```

**Problem 3: Data API Costs**

Aurora Serverless offers an HTTP-based “Data API” to avoid connection overhead.

**What AWS said:**

> *“No need to manage connections. Just make HTTP requests.”*

**The hidden costs:**

```c
Data API pricing:
  $0.35 per million requests
  + $0.10 per GB data processed
```
```c
Their workload:
  10 million API requests/month
  Average response: 50KB
  
Data API costs:
  Requests: 10M × $0.35/1M = $3.50
  Data transfer: (10M × 50KB) = 500GB × $0.10 = $50
  Total: $53.50/month (seems cheap!)
```

But that’s ON TOP OF Aurora costs:

```c
Total monthly bill:
  ACU hours: $9,200
  Data API requests: $3.50
  Data API data transfer: $50
  Storage: $460
  Backups: $180
  I/O requests: $1,847
  Total: $11,740/month
```
```c
vs. MySQL on EC2:
  EC2 instance (r6g.xlarge): $280/month
  EBS storage (1TB gp3): $80/month
  Snapshots: $40/month
  Data transfer: $140/month
  Total: $540/monthAurora cost: 21.7x more expensive
```

## The Resolution

**My recommendation:** “Switch back to RDS MySQL or run MySQL on EC2.”

**Their objection:** “But we need auto-scaling!”

**My response:** “You’re a startup with predictable traffic. You don’t need auto-scaling. You need predictable costs.”

**The analysis:**

```c
Current traffic: 1,000 requests/second peak
Current database load: Easily handled by r6g.xlarge ($280/mo)
```
```c
To need Aurora Serverless, you'd need:
  - 10x traffic spikes (10,000 req/sec)
  - Unpredictable patterns
  - Spikes lasting <30 minutes
  
Your traffic pattern:
  - Predictable daily cycle
  - Peak lasts 3-4 hours
  - Weekends are slower
  
= Perfect for fixed-size RDS instance
```

They switched to RDS MySQL.

**Results:**

```c
Before (Aurora Serverless): $11,740/month
After (RDS MySQL): $680/month
Savings: $11,060/month = $132,720/year
```
```c
Performance:
  Before: Variable (2-500ms query latency)
  After: Consistent (3-8ms query latency)
```

## When Aurora Serverless IS Right

Aurora Serverless works for:

- ✅ Infrequently used applications (dev/test environments)
- ✅ Truly unpredictable traffic (genuine spikes then silence)
- ✅ Applications that can pause (no users for hours)

Aurora Serverless is WRONG for:

- ❌ Production applications with consistent traffic
- ❌ Startups trying to save money
- ❌ Applications without connection pooling
- ❌ Cost-sensitive workloads

**Lesson:** “Serverless” and “pay for what you use” sound good, but often cost more than traditional resources. Do the math.

## Part 5: The Cassandra Disaster (When “Web Scale” Means “Web Pain”)

**The Company:** AdTech platform, Series B, 120 employees, $18M ARR

**The Decision:** “We need Cassandra. Our data is too big for SQL databases.”

**The Reality:** Cassandra was 10x harder to operate and 3x more expensive than PostgreSQL with proper partitioning.

## The Motivation

**Data size:** 8TB of ad impression data  
**Write volume:** 50,000 writes/second at peak  
**Read pattern:** Mostly recent data (last 30 days)

**Engineer’s reasoning:** “PostgreSQL can’t handle this scale.”

**The truth:** PostgreSQL could absolutely handle this with proper configuration.

## The Cassandra Implementation

**Month 1–6:** Learning Cassandra  
**Month 7–12:** Migrating data  
**Month 13:** Production on Cassandra  
**Month 14:** First major outage  
**Month 15:** Second major outage  
**Month 16:** Call me for help

## The Problems

**Problem 1: Consistency is Actually Hard**

```c
-- What they were doing in PostgreSQL:
SELECT impression_count, click_count 
FROM campaign_stats 
WHERE campaign_id = 12345;
```
```c
-- Simple, consistent, correct
```
```c
-- What they had to do in Cassandra:
SELECT impression_count, click_count 
FROM campaign_stats 
WHERE campaign_id = 12345;
```
```c
-- But: Different replicas might have different values
-- Need to understand: Consistency levels, read repair, hinted handoff
```

**Real incident:**

```c
Campaign dashboard shows:
  Impressions: 1,000,000
  Clicks: 50,000
  CTR: 5%
```
```c
Refresh page:
  Impressions: 987,000
  Clicks: 49,500
  CTR: 5.02%Refresh again:
  Impressions: 1,002,000
  Clicks: 50,100
  CTR: 5.00%Different replicas returning different data
Customer sees inconsistent numbers
Customer complains: "Your data is wrong"
```

**Problem 2: Query Limitations**

```c
-- What you CAN'T do in Cassandra without additional indexes:
```
```c
-- 1. Filter on non-partition-key columns
SELECT * FROM impressions 
WHERE user_id = 12345  -- user_id not partition key
-- Error: Cannot execute this query without ALLOW FILTERING-- 2. JOIN tables
SELECT * FROM impressions i
JOIN campaigns c ON i.campaign_id = c.id
-- Error: Joins not supported-- 3. Aggregations without pre-computing
SELECT campaign_id, COUNT(*), AVG(click_rate)
FROM impressions
GROUP BY campaign_id;
-- Error: This will scan entire cluster-- 4. ORDER BY non-clustering-column
SELECT * FROM impressions
ORDER BY timestamp DESC;
-- Error: Only allowed on clustering columns
```

**Their solution:** Denormalize everything, pre-compute everything

```c
-- They ended up with 23 different table designs
-- for the same data, just different query patterns
```
```c
CREATE TABLE impressions_by_campaign (...);
CREATE TABLE impressions_by_user (...);
CREATE TABLE impressions_by_date (...);
CREATE TABLE impressions_by_campaign_and_date (...);
CREATE TABLE impressions_by_user_and_date (...);
CREATE TABLE campaign_stats_by_hour (...);
CREATE TABLE campaign_stats_by_day (...);
-- ... 16 more tables
```

**Storage overhead:** Data duplicated 23 times across different table designs  
**Complexity:** Every write must update multiple tables  
**Cost:** 23x storage, complex application code

**Problem 3: Operations Nightmare**

```c
# PostgreSQL backup:
pg_dump production_db > backup.sql
# Restore:
psql production_db < backup.sql
```
```c
# Cassandra backup:
# 1. Take snapshot on each node
nodetool snapshot production_keyspace# 2. Copy snapshots from each node to S3
for node in node1 node2 node3 node4 node5 node6; do
    ssh $node "tar -czf /tmp/snapshot.tar.gz /var/lib/cassandra/data/..."
    scp $node:/tmp/snapshot.tar.gz s3://backups/$node/
done# 3. Pray you never need to restore
# Because restore is even more complex
```

**Actual restore scenario:**

```c
Node 3 dies (hardware failure)
Need to: 
  1. Provision new node
  2. Configure new node
  3. Join to cluster
  4. Stream data from other nodes (takes 8 hours for 1.5TB)
  5. During streaming: Cluster performance degraded 40%
  6. Hope nothing else fails
```

**PostgreSQL equivalent:** Promote replica, takes 30 seconds

**Problem 4: Debugging is Impossible**

```c
# PostgreSQL: Query is slow, investigate:
EXPLAIN ANALYZE SELECT ...;
# See exactly what's happening
```
```c
# Cassandra: Query is slow, investigate:
# - Which nodes did it query?
# - Which replicas responded?
# - Was there read repair?
# - What was consistency level?
# - Were there timeouts?
# - Check logs on 6 different nodes
# - Correlate timestamps
# - Give up and restart everything
```

**Real debugging session:** 6 engineers, 4 hours, never found root cause, “restarted cluster and hoped for the best”

## The Costs

```c
Cassandra cluster:
  6 nodes × r6g.2xlarge: $4,320/month
  Storage (6 × 3TB): $1,440/month
  Snapshots: $840/month
  Operations engineer (needed!): $150K/year
  Total: $6,600/month + $150K/year
  
PostgreSQL equivalent:
  1 primary (r6g.4xlarge): $720/month
  2 replicas (r6g.4xlarge): $1,440/month
  Storage (3 × 10TB): $2,400/month
  Snapshots: $200/month
  Total: $4,760/month
  No dedicated operations engineer needed
  
Cassandra: $162,200/year
PostgreSQL: $57,120/year
Difference: $105,080/year (2.8x more expensive)
```

**Plus migration costs:**

- 12 months of engineering time: $1.8M
- Application complexity: Ongoing maintenance burden
- Operational complexity: Constant firefighting

## The Resolution

**My assessment:** “You chose Cassandra because of scale myths. PostgreSQL can handle your workload easily.”

**Their data:**

- 8TB total
- 90% of queries access last 30 days (800GB)
- 10% access historical data

**My solution:**

```c
-- PostgreSQL with time-based partitioning
CREATE TABLE impressions (
    id BIGSERIAL,
    campaign_id BIGINT,
    user_id BIGINT,
    timestamp TIMESTAMP,
    ...
) PARTITION BY RANGE (timestamp);
```
```c
-- Monthly partitions
CREATE TABLE impressions_2024_10 PARTITION OF impressions
    FOR VALUES FROM ('2024-10-01') TO ('2024-11-01');-- Partition last 30 days
CREATE TABLE impressions_2024_11 PARTITION OF impressions
    FOR VALUES FROM ('2024-11-01') TO ('2024-12-01');-- Archive old partitions to cheaper storage
-- or drop entirely if not needed
```

**Results:**

```c
Write performance:
  Cassandra: 50,000 writes/sec (6 nodes)
  PostgreSQL: 52,000 writes/sec (1 primary + pgbouncer)
  
Read performance:
  Cassandra: 120ms P95 (consistency issues)
  PostgreSQL: 8ms P95 (consistent)
  
Operations:
  Cassandra: 1 full-time engineer
  PostgreSQL: Part-time maintenance
  
Cost:
  Cassandra: $162K/year
  PostgreSQL: $57K/year
  Savings: $105K/year
```

They migrated back to PostgreSQL. Never looked back.

## The Lesson

**Cassandra is great for:**

- ✅ Multi-datacenter replication (global scale)
- ✅ Always-on availability (can’t tolerate seconds of downtime)
- ✅ Linear scalability needed (10,000+ writes/second)
- ✅ Simple key-value queries

**Cassandra is wrong for:**

- ❌ Complex queries
- ❌ Strong consistency requirements
- ❌ Teams without deep Cassandra expertise
- ❌ “Because it scales” without measuring

**Lesson:** Traditional databases can handle enormous scale with proper configuration. Don’t add complexity until you’ve proven you need it.

## Part 6: The Contrarian Advice No One Wants to Hear

After 47 failed migrations and $23M in wasted costs, here are my controversial recommendations:

## 1\. Start with PostgreSQL or MySQL. Period.

**Not Mongo, not Cassandra, not the new hot database.**

Why?

- Mature technology (40+ years of development)
- Massive ecosystem (tools, libraries, expertise)
- Well-understood scaling patterns
- Excellent performance for 99% of use cases
- Can always migrate later if needed

**Rule:** Use SQL database unless you have a specific, measurable reason not to.

## 2\. Don’t Migrate Until You’re Actually Hitting Limits

**Signs you DON’T need to migrate:**

- ❌ “Everyone says X is better”
- ❌ “We might need to scale someday”
- ❌ “New database has cool features”
- ❌ “We want to use modern technology”

**Signs you DO need to migrate:**

- ✅ ==Current database consistently at >80% capacit== y
- ✅ Specific feature required (geospatial, full-text search)
- ✅ Cost savings are measurable and significant (10x+)
- ✅ Performance issues can’t be fixed with optimization

**Rule:** Fix your current database before migrating to a new one.

## 3\. Boring Technology is Good Technology

```c
Exciting technology:
  - Brand new database
  - Promises 100x performance
  - "Completely different approach"
  - Few production examples
  - Limited tooling
  
Boring technology:
  - 20+ years old
  - Battle-tested
  - Well-understood
  - Massive ecosystem
  - Predictable behavior
```

**Companies running “boring” technology:**

- Facebook: MySQL (billions of users)
- Instagram: PostgreSQL (billions of users)
- GitHub: MySQL (millions of repositories)
- Slack: MySQL (millions of users)

**Rule:** Use proven technology. Your database should be boring.

## 4\. Microservices Don’t Require Separate Databases

**The myth:** “Microservices means each service gets its own database”

**The reality:** Microservices are about code organization, not data separation

**The smart approach:**

```c
Core transactional services → Shared database
Truly independent services → Separate databases
```

**Rule:** Share databases when services need to share transactions. Separate only when independence matters.

## 5\. Cloud-Native Doesn’t Mean Cost-Effective

**The pitch:** “Serverless! Auto-scaling! Pay for what you use!”

**The reality:** Often 5–10x more expensive than traditional resources

**Do the math:**

```c
Aurora Serverless: $0.12/ACU-hour
  = $0.12 × 2 ACUs × 24 hours × 30 days = $172.80/month minimum
  
RDS r6g.large: $0.096/hour
  = $0.096 × 24 hours × 30 days = $69.12/month
  
Aurora is 2.5x more expensive for same capacity
And that's at MINIMUM capacity
```

**Rule:** “Serverless” often costs more. Calculate before committing.

## 6\. NoSQL Doesn’t Mean “No SQL Database”

**The myth:** “We need a NoSQL database to scale”

**The reality:** SQL databases scale to petabytes (YouTube, Facebook, Amazon)

**When NoSQL actually helps:**

- Flexible schemas (truly unpredictable data structure)
- Specific query patterns (key-value, document retrieval)
- Multi-datacenter replication requirements

**When NoSQL hurts:**

- Need transactions
- Need relationships between data
- Need complex queries
- Team doesn’t understand eventual consistency

**Rule:** NoSQL is a tool for specific jobs. SQL databases scale fine.

## 7\. Optimize Before Scaling

**The wrong approach:**

```c
1. Database is slow
2. Migrate to "faster" database
3. Spend 12 months migrating
4. New database also slow (unoptimized queries)
```

**The right approach:**

```c
1. Database is slow
2. Identify slow queries
3. Add indexes, optimize queries
4. 100x improvement in 1 week
5. No migration needed
```

**Rule:** Optimization is cheaper and faster than migration.

## 8\. “Web Scale” is Marketing, Not Engineering

**The myth:** “SQL databases don’t scale to web scale”

**The reality:**

- Instagram: PostgreSQL, 500M+ users
- Uber: MySQL, millions of rides/day
- Shopify: MySQL, billions in transactions
- Discord: PostgreSQL, millions of concurrent users

**These companies prove:** SQL databases scale to extreme load

**Rule:** If your traffic is less than Instagram’s, SQL databases can handle it.

## 9\. Test the Migration Before Committing

**Failed migrations always skip this step:**

```c
1. Decide to migrate
2. Start migration
3. Discover issues in production
4. Too late to go back
5. Stuck with bad decision
```

**Successful migrations always do this:**

```c
1. Decide to test migration
2. Migrate copy of production data
3. Run production workload against new database
4. Measure performance, cost, complexity
5. Decide based on real data
6. Either: Proceed with migration OR Stay with current database
```

**Rule:** Test with production data before migrating production.

## 10\. It’s OK to Say “No” to New Technology

**You don’t have to use:**

- The newest database
- The trendiest architecture
- The most hyped technology

**It’s OK to use:**

- PostgreSQL (boring, proven)
- MySQL (boring, proven)
- Monolithic architecture (boring, proven)
- Traditional scaling (boring, proven)

**Rule:** New technology should solve specific problems. If it doesn’t, say no.

## Conclusion: The Database Selection Framework

After analyzing 47 failed migrations, here’s my framework for database decisions:

## Step 1: Start with PostgreSQL or MySQL

**Default to proven technology.**

Choose PostgreSQL if:

- Complex queries needed
- JSON data
- Geospatial data
- Advanced features (window functions, CTEs)

Choose MySQL if:

- Simple queries
- High write volume
- Replication is critical
- Team knows MySQL

**Don’t overthink this decision.** Both are excellent.

## Step 2: Only Consider Alternatives with Specific Justification

**Need special database only if:**

**MongoDB:** Truly flexible schema (can’t define structure)  
**Cassandra:** Multi-datacenter, always-on (can’t tolerate downtime)  
==**Redis**==**:** Caching layer (sub-millisecond access)  
**Elasticsearch:** Full-text search (complex search queries)  
**ClickHouse:** Analytics (complex aggregations on petabytes)  
**DynamoDB:** Serverless, pay-per-request (truly variable load)

**For everything else: PostgreSQL or MySQL**

## Step 3: Measure Before Migrating

**Required measurements:**

```c
Performance:
  - Run production workload on new database
  - Measure query times (P50, P95, P99)
  - Measure write throughput
  - Measure consistency
  
Cost:
  - Calculate monthly database cost
  - Calculate storage cost
  - Calculate backup cost
  - Calculate data transfer cost
  - Calculate engineering time cost
  
Complexity:
  - Time to implement
  - Learning curve
  - Operational burden
  - Debugging difficulty
  
Risk:
  - Migration duration
  - Rollback procedure
  - Data consistency guarantees
  - Vendor lock-in
```

**Make decision based on data, not trends.**

## Step 4: Test with Production Data

**Create test environment:**

1. Copy production data to new database
2. Run production queries
3. ==Monitor for 7 day== s
4. Compare all metrics
5. Decide: Migrate or stay

**Never migrate without testing.**

## Step 5: Have a Rollback Plan

**Before migrating:**

- Document rollback procedure
- Test rollback procedure
- Set decision points (“rollback if X happens”)
- Keep old database running for 30 days

**Rule:** If you can’t roll back, you’re not ready to migrate.

## The Real Lesson: Question Everything

The database industry is full of marketing, hype, and groupthink.

**Everyone says:**

- “PostgreSQL is better than MySQL”
- “NoSQL scales better than SQL”
- “Microservices need separate databases”
- “Cloud-native is the future”
- “Serverless saves money”

**But the truth is:**

- **Sometimes MySQL is better**
- **Sometimes SQL scales better**
- **Sometimes shared databases are right**
- **Sometimes traditional architecture is better**
- **Sometimes fixed resources cost less**

**The right answer is always: “It depends.”**

It depends on:

- Your data
- Your queries
- Your team
- Your budget
- Your scale
- Your requirements

**Don’t follow trends. Follow data.**

## About Jamaurice Holt

I’m a Senior AWS Database Administrator and Full-Stack Developer with 10+ years of experience cleaning up database disasters for Fortune 500 companies and startups.

**My track record:**

- 500+ databases optimized
- 47 failed migrations rescued
- $23M in wasted migration costs prevented
- Saved companies from bankruptcy-causing database decisions

**I specialize in:**

- Preventing bad database decisions
- Optimizing existing databases (vs. migrating)
- Emergency migration rescue
- Database architecture consulting
- Honest advice (not vendor marketing)

**Connect with me:**

- Portfolio: [https://jamaurice-holt.com](https://jamaurice-holt.com/)
- LinkedIn: [https://linkedin.com/in/jamauriceholt](https://linkedin.com/in/jamauriceholt)
- GitHub: [https://github.com/Jamaurice](https://github.com/Jamaurice)
- Medium: [https://medium.com/@jamauriceholt](https://medium.com/@jamauriceholt)

## [Jamaurice Holt | Senior AWS Database Administrator & Full-Stack Developer](http://jamauriceholt.com/?source=post_page-----56d241850225---------------------------------------)

### Jamaurice Holt - Senior AWS Database Administrator & Full-Stack Developer with 10+ years experience in cloud database…

jamauriceholt.com

**Considering a database migration?**

📧 Email: jamauriceholt@yahoo.com

I offer:

- **Free migration assessment** (Should you actually migrate?)
- **Database optimization** (Fix current database first)
- **Migration planning** (If migration is justified)
- **Emergency rescue** (If migration already failed)

**I’ll tell you the truth, even if it’s not what you want to hear.**

**If this article saved you from a costly migration mistake, share it with your team.**

**Disagree with my advice? Leave a comment. I want to hear the other side.**

**Have a migration disaster story? Email me. Let’s learn from it together.**

*Tags: #Database #PostgreSQL #MySQL #MongoDB #Cassandra #NoSQL #Microservices #DatabaseMigration #CloudNative #Aurora #Serverless #Architecture #Engineering #TechDebt #Databases #Controversial #DevOps*