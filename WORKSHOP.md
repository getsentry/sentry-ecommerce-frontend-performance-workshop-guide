# Workshop Flow & Expected Application Behavior

## Workshop Overview

**Goal**: Transform a broken e-commerce application into a production-ready, high-performance system by learning Sentry's performance monitoring, error tracking, distributed tracing, Session Replay, and AI-powered debugging.

**Target Scenario**: Holiday shopping season is approaching with high traffic spikes. The app has multiple performance and UX issues that will hurt conversions during peak sales periods (Black Friday, Cyber Monday).

---

## Initial State (Before Workshop)

### Application Problems

The e-commerce app (`unborked`) starts in a degraded state with multiple issues:

**Frontend Issues:**

- ❌ **Slow LCP**: Images load late (> 4s), blocking main content rendering
- ❌ **High CLS**: Promotional banner appears after page load, causing layout shift
- ❌ **Poor TTFB**: Sale page blocks rendering while waiting for API (> 1800ms)
- ❌ **No loading states**: Users see blank screens or no feedback during operations

**Backend Issues:**

- ❌ **N+1 Query Problem**: Sale API executes 101 database queries (2.5-3s response time)
- ❌ **Missing indexes**: Slow database queries even after fixing N+1 pattern
- ❌ **No instrumentation**: Can't see what's causing slowdowns

**UX Issues:**

- ❌ **Hidden validation errors**: Checkout form fails silently, users rage-click
- ❌ **No visual feedback**: Buttons don't show loading/disabled states
- ❌ **Confusing interactions**: Elements look clickable but aren't (dead clicks)
- ❌ **Poor error messages**: Users don't know what went wrong

**Monitoring:**

- ❌ No error tracking
- ❌ No performance monitoring
- ❌ No alerts
- ❌ No visibility into user experience

### Business Impact

- **Checkout conversion**: 45% (very poor)
- **Rage click rate**: 28% of sessions (users are frustrated)
- **Cart abandonment**: 55% (users give up)
- **Expected holiday revenue loss**: Significant due to poor performance and UX

---

## Module Breakdown

### Module 0: Quickstart (15 minutes)

**Objective**: Get the development environment running

**Actions:**

1. Fork the `unborked` repository to enable Seer AI PR creation
2. Clone the repository locally
3. Install dependencies with `pnpm install`
4. Set up PostgreSQL database (automatic with Neon or manual)
5. Run migrations and seed data
6. Start development servers (`pnpm dev`)
7. Create two Sentry projects (Frontend: React, Backend: Node.js)

**Application State After:**

- ✅ App runs locally at http://localhost:5173
- ✅ Backend API running at http://localhost:3001
- ✅ Database populated with products, users, inventory
- ⚠️ Still has all the performance/UX issues (not yet instrumented)

---

### Module 1: Setting Up Performance & Error Monitoring (45 minutes)

**Objective**: Instrument Sentry SDKs to start collecting performance and error data

**Actions:**

**Frontend Instrumentation:**

1. Install `@sentry/react` SDK
2. Create `instrument.ts` with:
   - Browser tracing integration (Web Vitals, LCP, CLS)
   - Session Replay integration
   - Distributed tracing config (`tracePropagationTargets`)
   - Sample rates (100% for dev, 10% recommended for prod)
3. Import instrumentation first in `main.tsx`
4. Add Sentry DSN to `.env`
5. Configure sourcemaps with Sentry Wizard + Vite plugin

**Backend Instrumentation:**

1. Install `@sentry/node` SDK
2. Create `instrument.ts` with:
   - Performance monitoring (100% traces)
   - Console logs enabled (`enableLogs: true`)
3. Import instrumentation first in `index.ts`
4. Add Express error handler (`setupExpressErrorHandler`)
5. Add Sentry DSN to `.env`

**Application State After:**

- ✅ Frontend transactions flowing to Sentry (pageload, navigation)
- ✅ Backend transactions flowing to Sentry (HTTP requests)
- ✅ Session Replays being recorded (10% sample rate)
- ✅ Web Vitals data being collected (LCP, CLS, TTFB, FCP, INP)
- ✅ Errors automatically captured
- ⚠️ Still has all the performance issues (not fixed yet)
- ⚠️ No alerts configured
- ⚠️ Can see problems in Sentry but haven't diagnosed them yet

**What You See in Sentry:**

- Poor Web Vitals scores (red/yellow indicators)
- Slow transaction durations (> 2s for sale API)
- Some console errors in replays
- No clear understanding of root causes yet

---

### Module 2: Optimizing Web Vitals (60 minutes)

**Objective**: Fix critical frontend performance issues affecting LCP, CLS, and TTFB

**Issue #1: Slow Loading Images (Poor LCP)**

**Problem:**

- Homepage product images load > 4s
- Images load after React hydration
- No width/height attributes (causes layout shifts)
- Full-resolution images served (3.2MB each)

**Fix:**

```tsx
// Add proper image attributes
<img
  src={product.imageUrl}
  alt={product.name}
  width={300}
  height={300}
  loading="eager" // For above-fold
  decoding="async"
  style={{ aspectRatio: '1/1' }}
/>
```

**Result:**

- ✅ LCP improves from > 4s to < 2.5s (good)
- ✅ Images load immediately without shifts

---

**Issue #2: Banner Layout Shift (Poor CLS)**

**Problem:**

- Promotional banner appears after auth check
- Causes content to jump down (layout shift)
- CLS > 0.25 (poor) for unauthenticated users

**Fix:**

```tsx
// Reserve space for banner to prevent shift
<div className="promo-banner-container" style={{ minHeight: '48px' }}>
  {!isLoading && !isAuthenticated && (
    <div className="promo-banner">🎉 New Customer? Get 10% off!</div>
  )}
</div>
```

**Added Sentry tags for analysis:**

```tsx
Sentry.setTag('banner.rendered', isAuthenticated ? 'no' : 'yes');
Sentry.setTag('user.authenticated', isAuthenticated);
```

**Result:**

- ✅ CLS improves from > 0.25 to < 0.1 (good)
- ✅ Can filter Web Vitals by banner state in Sentry
- ✅ Can identify which user segments have issues

---

**Issue #3: Render-Blocking Sale API (Poor TTFB)**

**Problem:**

- Sale page waits for API before rendering anything
- Users see blank screen for 2-3 seconds
- TTFB > 1800ms (poor)

**Fix:**

```tsx
// Show skeleton immediately, load data in background
{
  loading ? (
    <ProductGridSkeleton count={12} /> // ✅ Shows immediately
  ) : (
    <ProductGrid products={products} />
  );
}
```

**Added performance tracking:**

```tsx
const span = Sentry.startInactiveSpan({
  name: 'sale.products.load',
  op: 'http.client',
});
```

**Result:**

- ✅ TTFB improves from > 1800ms to < 800ms (good)
- ✅ Users see content immediately (perceived performance)
- ⚠️ Skeleton still loads for 2-3s (API is still slow - will fix in Module 3)

**Application State After Module 2:**

- ✅ All Web Vitals in "good" range (LCP < 2.5s, CLS < 0.1, TTFB < 800ms)
- ✅ 24% improvement in projected conversion rate (from Web Vitals research)
- ✅ Sentry tags enable filtering by user state
- ⚠️ Backend API still slow (users wait for skeleton to resolve)

---

### Module 3: Distributed Tracing & Backend Performance (75 minutes)

**Objective**: Use distributed tracing to trace frontend slowness to backend bottlenecks and fix N+1 query problems

**Understanding Distributed Tracing (New to this module):**

- Learn what distributed tracing is
- Verify frontend → backend connection works
- See request flow from browser → API → database
- Use "View Full Trace" to see complete waterfall

---

**Problem Discovery: Sale API is Slow**

**Simulate Load:**

```bash
pnpm traffic  # Simulates 100 concurrent users
```

**What Sentry Shows:**

- Backend `GET /api/products/sale`: **p95 = 2500ms** (terrible)
- Under load: Some requests > 5s
- Frontend skeleton waits 2-3s for products

**Using Distributed Tracing:**

1. Find slow `/sale` pageload transaction in frontend
2. Click "View Full Trace"
3. See complete waterfall:
   - Frontend: `/sale` pageload (2500ms total)
   - HTTP request to backend (2300ms)
   - Backend: `GET /api/products/sale` (2200ms)
   - **101 database queries** (the "comb" pattern)

---

**Root Cause: N+1 Query Problem**

**What's happening:**

```typescript
// ❌ N+1 pattern: 101 total queries
const products = await db.select().from(products).where(eq(products.onSale, true)); // 1 query

const enriched = await Promise.all(
  products.map(async (p) => {
    const discount = await db.select()...;  // 50 queries
    const inventory = await db.select()...; // 50 queries
    return { ...p, discount, inventory };
  })
);
// Total: 101 queries, 2750ms wasted on database roundtrips
```

---

**Fix #1: Add Instrumentation**

Add Sentry spans to see individual queries:

```typescript
const saleProducts = await Sentry.startSpan(
  { name: 'db.query.sale_products', op: 'db.query' },
  async () => {
    return await db.select().from(products)...;
  }
);

span.setAttributes({
  'products.count': saleProducts.length,
  'db.queries.total': 1 + saleProducts.length * 2, // 101!
});
```

**Result:**

- ✅ Can see all 101 queries in Sentry waterfall
- ✅ Clear visibility into the problem

---

**Fix #2: Optimize with SQL JOINs**

```typescript
// ✅ Optimized: Single query with JOINs
const products = await Sentry.startSpan(
  { name: 'db.query.sale_products_optimized', op: 'db.query' },
  async () => {
    return await db
      .select({
        id: products.id,
        name: products.name,
        price: products.price,
        discountPercent: discounts.percentage,
        stock: inventory.quantity,
      })
      .from(products)
      .leftJoin(discounts, eq(products.id, discounts.productId))
      .leftJoin(inventory, eq(products.id, inventory.productId))
      .where(eq(products.onSale, true))
      .limit(50);
  }
);

span.setAttributes({
  'db.queries.total': 1, // Down from 101!
});
```

**Result:**

- ✅ API response: **2500ms → 150ms** (94% faster)
- ✅ Database queries: **101 → 1** (99% reduction)
- ✅ Frontend skeleton duration: **2500ms → 150ms**

---

**Fix #3: Add Database Indexes**

```sql
-- Index for WHERE clause
CREATE INDEX idx_products_on_sale ON products(on_sale);

-- Indexes for JOINs
CREATE INDEX idx_discounts_product_id ON discounts(product_id);
CREATE INDEX idx_inventory_product_id ON inventory(product_id);
```

**Result:**

- ✅ Query duration: **150ms → 50ms** (further 67% improvement)
- ✅ Final API response time: **50ms** (98% faster than original)

---

**Performance Comparison:**

| Metric                     | Before (N+1) | After (JOINs + Indexes) | Improvement       |
| -------------------------- | ------------ | ----------------------- | ----------------- |
| API Response Time (p95)    | 2500ms       | 50ms                    | **98% faster**    |
| Database Queries           | 101 queries  | 1 query                 | **99% reduction** |
| Frontend Skeleton Duration | 2500ms       | 50ms                    | **98% faster**    |
| User Experience            | Terrible     | Fast ✅                 | 🎉                |

**Application State After Module 3:**

- ✅ Sale API optimized (2500ms → 50ms)
- ✅ N+1 queries eliminated
- ✅ Database indexed properly
- ✅ Distributed tracing shows single optimized query
- ✅ Users see products in < 300ms (was 2500ms)
- ✅ Can handle Black Friday traffic load

---

### Module 4: Session Replay & User Experience Debugging (60 minutes)

**Objective**: Use Session Replay to identify and fix UX issues that don't throw errors but hurt conversions

**The Scenario:**

- Checkout conversion drops from 65% → 45% overnight
- Error rate is normal (< 0.5%)
- Performance metrics look good
- But users complain "checkout isn't working"

**Problem: Technical metrics look fine, but conversions are tanking**

---

**Discovery: Rage Clicks & User Frustration**

**Filter replays by frustration signals:**

```
click.tag:"button"
click.selector:*checkout*
rage_click_count:>0
```

**What you see in replay:**

1. User fills checkout form (name, email, card details)
2. Clicks "Place Order" button
3. **Clicks again... and again... 10+ times!**
4. No visual feedback (no spinner, no state change)
5. User gives up and leaves

**Console logs reveal:**

```
[DEBUG] Checkout button clicked
[ERROR] Card number validation failed: Invalid format
[DEBUG] Checkout button clicked
[ERROR] Card number validation failed: Invalid format
...
```

**Root Cause:**

- Validation errors exist but aren't shown to user
- Button doesn't disable or show loading state
- Users have no idea what's wrong

---

**Fix #1: Show Validation Errors**

```tsx
const [validationErrors, setValidationErrors] = useState<string[]>([]);

const handleSubmit = async (e) => {
  const errors = validateCheckoutForm(formData);

  if (errors.length > 0) {
    setValidationErrors(errors);

    // Track validation failures
    Sentry.logger.warn('Checkout validation failed', { errors });
    span?.setAttributes({
      'checkout.validation_failed': true,
      'checkout.validation_errors': errors.join(', '),
    });

    return; // Don't proceed
  }

  // ... proceed with checkout
};

// Show errors to user
{
  validationErrors.length > 0 && (
    <div className="validation-errors">
      <p>Please fix the following errors:</p>
      <ul>
        {validationErrors.map((error, i) => (
          <li key={i}>{error}</li>
        ))}
      </ul>
    </div>
  );
}
```

---

**Fix #2: Add Clear Loading State**

```tsx
<button
  type="submit"
  disabled={isProcessing}
  className={isProcessing ? 'loading' : ''}
>
  {isProcessing ? (
    <>
      <span className="spinner" />
      Processing...
    </>
  ) : (
    'Place Order'
  )}
</button>
```

**Result:**

- ✅ Errors shown clearly to users
- ✅ Button disables while processing (prevents rage clicks)
- ✅ Visual feedback with spinner
- ✅ Users know what's happening

---

**Fix #3: Dead Clicks (Elements That Look Clickable But Aren't)**

**Filter replays:**

```
dead_click_count:>2
url:*/checkout*
```

**Discovery:**

- Users keep clicking "Apply" next to promo code field
- But there's no button there (promo applies on blur)

**Fix:**

```tsx
// Before: Auto-applies on blur (confusing)
<input
  name="promoCode"
  onBlur={handleApplyPromo}
  placeholder="Enter promo code"
/>

// After: Explicit Apply button
<div className="promo-code-field">
  <input
    name="promoCode"
    placeholder="Enter promo code"
  />
  <button
    type="button"
    onClick={handleApplyPromo}
  >
    Apply
  </button>
</div>
```

---

**Measuring Impact:**

Add conversion tracking:

```tsx
// When checkout starts
const checkoutSpan = Sentry.startInactiveSpan({
  name: 'checkout.started',
  op: 'checkout.funnel',
});
checkoutSpan.setAttributes({
  'checkout.stage': 'started',
  'checkout.cart_value': cartTotal,
});

// When checkout completes
const completionSpan = Sentry.startInactiveSpan({
  name: 'checkout.completed',
  op: 'checkout.funnel',
});
completionSpan.setAttributes({
  'checkout.stage': 'completed',
  'checkout.order_id': orderId,
});
```

**Before vs After:**

| Metric                | Before Fixes | After Fixes | Improvement |
| --------------------- | ------------ | ----------- | ----------- |
| Conversion Rate       | 45%          | 63%         | **+18%** ✅ |
| Rage Click Sessions   | 28%          | 5%          | **-82%** ✅ |
| Dead Click Sessions   | 15%          | 3%          | **-80%** ✅ |
| Avg Checkout Duration | 4m 30s       | 2m 15s      | **-50%** ✅ |

**Application State After Module 4:**

- ✅ Validation errors visible to users
- ✅ Clear loading states on all actions
- ✅ Dead click issues fixed (all interactive elements work)
- ✅ Conversion rate improved 18% (45% → 63%)
- ✅ User frustration signals dramatically reduced
- ✅ Session Replay + console logs provide full debugging context

---

### Module 5: Production Readiness & AI-Powered Debugging (60 minutes)

**Objective**: Set up alerts, configure GitHub integration, and leverage Seer AI for automated debugging

---

**Part 1: Setting Up Alert Rules (Moved from Module 1)**

**Why now instead of Module 1?**

- You now understand WHAT to alert on (you've experienced the issues)
- You know WHY these alerts matter (you've seen the business impact)
- You can set appropriate thresholds based on your optimization work

**Alert #1: Frontend Error Rate**

```
Type: Issues
Condition: Error rate > 1%
Environment: Production
Action: Notify via Email/Slack
```

_Rationale: After fixing issues, 1% error rate indicates a regression_

**Alert #2: Web Vitals (LCP)**

```
Type: Metric Alert
Metric: p75(measurements.lcp)
Condition: > 2500ms for 5 minutes
Filter: transaction.op:pageload
Action: Notify
```

_Rationale: You optimized LCP in Module 2; this catches regressions_

**Alert #3: Backend Performance**

```
Type: Metric Alert
Metric: p95(transaction.duration)
Condition: > 1000ms (1 second)
Action: Notify
```

_Rationale: You fixed N+1 queries in Module 3; this catches similar issues_

---

**Part 2: Configuring GitHub Integration for Seer**

**Why?** Seer needs repo access to:

- Read code for context-aware suggestions
- Open PRs with automated fixes
- Suggest specific code changes

**Steps:**

1. Enable AI features in Sentry org settings
2. Install GitHub integration
3. Authorize Sentry on GitHub
4. Select `unborked` repository
5. Confirm access and enable Autofix

---

**Part 3: Using Seer AI for Root Cause Analysis**

**Scenario 1: N+1 Query Analysis**

**Ask Seer about the sale API issue (from before your fix):**

Seer's analysis:

```
🔍 Root Cause Analysis

Performance Issue Detected: N+1 Query Pattern

Your application is executing 101 database queries to fetch
sale products:
- 1 initial query to fetch 50 products
- 50 queries to fetch discount information (one per product)
- 50 queries to fetch inventory information (one per product)

Impact:
- Total query time: 2,750ms (96% of request duration)
- Database load: 101 connections per request
- Under load: This will exhaust your connection pool

Recommended Fix:
Use SQL JOINs to fetch all data in a single query.

Estimated improvement: 95% faster (2750ms → 120ms)
```

**Seer provides exact code:**

```typescript
// ✅ Suggested fix (Single query with JOINs)
const products = await db
  .select({
    id: products.id,
    discountPercent: discounts.percentage,
    stock: inventory.quantity,
  })
  .from(products)
  .leftJoin(discounts, eq(products.id, discounts.productId))
  .leftJoin(inventory, eq(products.id, inventory.productId))
  .where(eq(products.onSale, true));
```

**Ask follow-up: "What indexes should I add?"**

Seer responds:

```
Based on your query, I recommend adding:
- Index on products.on_sale for WHERE clause filtering
- Index on discounts.product_id for JOIN performance
- Index on inventory.product_id for JOIN performance

Run these migrations:
CREATE INDEX idx_products_on_sale ON products(on_sale);
CREATE INDEX idx_discounts_product_id ON discounts(product_id);
CREATE INDEX idx_inventory_product_id ON inventory(product_id);
```

---

**Scenario 2: Poor LCP Analysis**

Seer analyzes slow LCP transaction:

```
🔍 Root Cause Analysis

Poor LCP Detected: 4,200ms

Your Largest Contentful Paint is caused by slow-loading
product images on the homepage.

Issues Found:
1. Images loading after React hydration (delayed 800ms)
2. Full-resolution images served (3.2MB each)
3. No lazy loading for below-the-fold images
4. No image preloading for hero images

Impact on Business:
- 53% of mobile users abandon sites with LCP > 3s
- Your LCP of 4.2s likely costs ~15% of conversions

Recommended Fixes:
1. Add eager loading for above-fold images
2. Serve WebP format (70% smaller)
3. Add width/height attributes to prevent layout shifts
4. Preload hero image
```

With specific code changes:

```tsx
// ✅ Suggested
<img
  src={product.imageUrl}
  alt={product.name}
  width={300}
  height={300}
  loading="eager"
  decoding="async"
  style={{ aspectRatio: '1/1' }}
/>
```

---

**Part 4: Seer Opens PRs Automatically**

**How it works:**

1. Click "Create PR with Fix" in Seer analysis
2. Seer creates a branch: `seer/fix-n1-query-sale-api`
3. Applies code changes
4. Adds database migrations
5. Writes tests
6. Opens PR on GitHub

**PR includes:**

```markdown
## Issue

The sale products endpoint was executing 101 database queries
per request, causing 2.5s response times under load.

## Root Cause

N+1 query pattern: fetching related data (discounts, inventory)
in a loop instead of using SQL JOINs.

## Solution

- Rewrote query to use LEFT JOINs
- Added database indexes for JOIN performance
- Reduced query count from 101 → 1

## Performance Impact

- Response time: 2500ms → 120ms (95% improvement)
- Database load: 99% reduction in queries

## Testing

- Added unit tests for optimized query
- Load tested with 100 concurrent requests
- Verified distributed tracing shows single query

🤖 Generated by Sentry Seer
```

**Files Changed:**

- `apps/api/src/routes/products.ts` - Optimized query
- `apps/api/drizzle/migrations/0003_add_indexes.sql` - Indexes
- `apps/api/src/routes/products.test.ts` - Tests

---

**Part 5: Ongoing Optimization with Seer**

**Weekly performance review:**

1. Go to Performance → Sort by p95 duration
2. Click top 5 slowest transactions
3. Ask Seer: "How can I optimize this?"

**Example: Slow checkout API**

Seer suggests:

```
🔍 Optimization Opportunities

Transaction: POST /api/checkout
Current p95: 1,200ms

Potential optimizations:

1. Cache user data (saves 200ms)
   - User info fetched on every checkout
   - Cache with 5-minute TTL

2. Parallelize payment processing (saves 300ms)
   - Payment gateway and inventory check run sequentially
   - Can run in parallel with Promise.all()

3. Optimize tax calculation (saves 150ms)
   - Tax API called even when same address
   - Cache tax rates by ZIP code

Estimated improvement: 650ms faster (46% improvement)
```

**Priority matrix:**

| Optimization        | Time Saved | Effort | ROI         |
| ------------------- | ---------- | ------ | ----------- |
| Cache user data     | 200ms      | Low    | High ⭐⭐⭐ |
| Parallelize payment | 300ms      | Medium | High ⭐⭐⭐ |
| Cache tax rates     | 150ms      | Low    | Medium ⭐⭐ |

---

**When to Trust Seer vs Manual Review:**

**✅ Trust Seer for:**

- Obvious performance issues (N+1 queries, missing indexes)
- Standard optimizations (image loading, caching)
- Database query optimization
- Code following common patterns

**⚠️ Review carefully:**

- Changes to business logic
- Security-sensitive code (auth, payments)
- Complex algorithm changes
- Code with subtle edge cases

**❌ Don't use Seer for:**

- Critical payment processing logic
- Complex business rules
- Quick production hotfixes (review manually first)

---

**Measuring Seer's Impact (After 1 Month):**

| Metric                       | Value                  |
| ---------------------------- | ---------------------- |
| Seer PRs created             | 23                     |
| PRs merged                   | 18 (78%)               |
| Avg. performance improvement | 62% faster             |
| Time saved                   | ~40 developer hours    |
| Issues prevented             | 5 production incidents |

**Application State After Module 5:**

- ✅ Alert rules configured (error rate, LCP, backend performance)
- ✅ GitHub integration connected for Seer AI
- ✅ Seer can analyze issues and suggest fixes
- ✅ Automated PR creation enabled
- ✅ Weekly optimization workflow established
- ✅ Proactive monitoring for regressions

---

## Final State (After Workshop Completion)

### Application Improvements

**Performance Metrics:**

| Metric                      | Before      | After       | Improvement    |
| --------------------------- | ----------- | ----------- | -------------- |
| Homepage LCP                | 4200ms      | 1800ms      | **57% faster** |
| CLS Score                   | 0.28 (poor) | 0.05 (good) | **82% better** |
| Sale Page TTFB              | 1900ms      | 200ms       | **89% faster** |
| Sale API Response (p95)     | 2500ms      | 50ms        | **98% faster** |
| Database Queries (sale API) | 101         | 1           | **99% fewer**  |
| Checkout Conversion Rate    | 45%         | 63%         | **+18%**       |
| Rage Click Rate             | 28%         | 5%          | **-82%**       |
| Dead Click Rate             | 15%         | 3%          | **-80%**       |
| Avg. Checkout Time          | 4m 30s      | 2m 15s      | **50% faster** |

---

### Technical Capabilities Gained

**Monitoring & Observability:**

- ✅ Frontend performance monitoring (Web Vitals, custom spans)
- ✅ Backend performance monitoring (API endpoints, database queries)
- ✅ Error tracking across full stack
- ✅ Distributed tracing (browser → API → database)
- ✅ Session Replay with privacy masking
- ✅ Console logs captured in context
- ✅ Custom tags for filtering by user state/features

**Performance Optimizations:**

- ✅ Image optimization (sizing, lazy loading, preloading)
- ✅ Layout shift prevention (reserved space)
- ✅ Skeleton screens for perceived performance
- ✅ N+1 query elimination (SQL JOINs)
- ✅ Database indexing for fast queries
- ✅ Proper loading states and user feedback

**UX Improvements:**

- ✅ Visible validation errors
- ✅ Clear loading states on all actions
- ✅ Interactive elements work as expected
- ✅ Checkout flow streamlined

**Operational Excellence:**

- ✅ Alerts configured for critical metrics
- ✅ GitHub integration for Seer AI
- ✅ Automated issue detection and suggestions
- ✅ Weekly optimization workflow
- ✅ Custom dashboards for business metrics

---

### Business Impact

**Revenue Impact:**

- **18% increase in checkout conversion** (45% → 63%)
- **50% faster checkout process** (4m 30s → 2m 15s)
- **Projected holiday season readiness**: Can handle 10x traffic spike
- **Customer satisfaction**: 82% reduction in frustration signals

**Estimated Holiday Season Revenue Gain:**

```
Baseline traffic: 100,000 visitors during Black Friday week
Old conversion: 45% = 45,000 orders × $150 avg = $6,750,000
New conversion: 63% = 63,000 orders × $150 avg = $9,450,000

Additional revenue: $2,700,000 (40% increase) 💰
```

**Developer Productivity:**

- **40 hours saved per month** using Seer AI
- **5 production incidents prevented** through proactive monitoring
- **90% reduction in debugging time** with Session Replay
- **Real-time visibility** into user experience

---

### Ready for Production

**Deployment Checklist:**

- ✅ All performance optimizations merged
- ✅ Sentry instrumentation in production
- ✅ Sample rates configured (10-30% for traces)
- ✅ Alert rules active
- ✅ Session Replay privacy settings verified
- ✅ Dashboards created for stakeholders
- ✅ Team trained on Sentry + Seer workflow

**Continuous Improvement Process:**

1. **Weekly**: Review Sentry dashboards for slow transactions
2. **Weekly**: Ask Seer about optimization opportunities
3. **Weekly**: Watch Session Replays for UX issues
4. **Monthly**: Review Web Vitals trends
5. **Monthly**: Measure conversion rate improvements

---

## Key Learning Outcomes

### Students Will Learn:

1. **Instrumentation Best Practices**
   - How to set up Sentry SDKs for React and Node.js
   - Importance of sourcemaps for production debugging
   - Distributed tracing configuration
   - Sample rate strategies

2. **Performance Debugging**
   - How to interpret Web Vitals (LCP, CLS, TTFB)
   - Using waterfalls to identify bottlenecks
   - Tracing frontend issues to backend causes
   - Understanding N+1 query patterns

3. **Optimization Techniques**
   - Image optimization strategies
   - Layout shift prevention
   - Skeleton screens for perceived performance
   - SQL JOIN optimization
   - Database indexing

4. **User Experience Debugging**
   - Reading Session Replays effectively
   - Identifying frustration patterns (rage clicks, dead clicks)
   - Combining replays with console logs
   - Privacy considerations for Session Replay

5. **Operational Excellence**
   - Setting appropriate alert thresholds
   - Creating custom dashboards
   - Using Seer AI for automated debugging
   - Establishing weekly optimization workflows

6. **Business Impact**
   - Connecting technical metrics to conversion rates
   - Measuring ROI of performance improvements
   - Communicating wins to stakeholders

---

## Workshop Success Criteria

By the end of this workshop, students should be able to:

- ✅ Instrument a full-stack application with Sentry
- ✅ Identify and fix Web Vitals issues
- ✅ Use distributed tracing to debug end-to-end performance
- ✅ Leverage Session Replay to fix UX issues
- ✅ Set up production-ready alerts and dashboards
- ✅ Use Seer AI for automated debugging and suggestions
- ✅ Measure and communicate business impact of performance work

**Most importantly**: Students gain confidence that their application is **ready for high-traffic production scenarios** like Black Friday, with comprehensive monitoring, proactive alerts, and the ability to quickly debug any issues that arise.
