# Performance Measurement Guide: LCP & INP

Shopify App Store requires:
- **LCP (Largest Contentful Paint)** < 2.5 seconds
- **INP (Interaction to Next Paint)** < 200 milliseconds

## What These Metrics Mean

### LCP — Largest Contentful Paint
Measures how long it takes for the **largest visible element** (typically a heading, image, or data table) to render on screen. This is the user's perception of "the page has loaded."

For CatalogNow, the LCP element is likely:
- **Dashboard**: The product stats cards or the main heading
- **Products list**: The product data table
- **Product detail**: The product title + description area

### INP — Interaction to Next Paint
Measures the **worst-case delay** between a user interaction (click, tap, keypress) and the next visual update. This captures how "responsive" the app feels.

For CatalogNow, critical interactions to test:
- Clicking "Sync Products" button
- Clicking a product row to navigate
- Clicking "Accept" / "Dismiss" on AI suggestions
- Switching between nav items

## Step-by-Step Measurement

### Method 1: Chrome DevTools (Recommended)

1. **Start the app**
   ```bash
   shopify app dev
   ```

2. **Open your dev store** in Chrome and navigate to the app

3. **Open DevTools** → Performance tab (Cmd+Option+I → Performance)

4. **Measure LCP:**
   - Click the "Record" button (circle icon)
   - Reload the page (Cmd+R)
   - Wait for the page to fully load, then stop recording
   - In the results, look for the **"LCP"** marker in the Timings lane
   - The timestamp shows your LCP value
   - Target: < 2,500ms

5. **Measure INP:**
   - Start a new recording
   - Perform interactions: click buttons, navigate between pages
   - Stop recording
   - Look for **"Interaction"** entries in the Timings lane
   - The longest interaction duration is your INP
   - Target: < 200ms

### Method 2: Lighthouse (Quick Audit)

1. Open DevTools → Lighthouse tab
2. Select:
   - Mode: **Navigation** (for LCP) or **Timespan** (for INP)
   - Device: **Desktop** (since Shopify admin is desktop-only)
   - Categories: **Performance**
3. Click "Analyze page load"
4. Check the LCP and INP scores in the results

**Important:** Run Lighthouse with the app embedded in Shopify admin (not standalone), because App Bridge adds overhead.

### Method 3: web-vitals Library (Automated)

Add temporary instrumentation to measure in production:

```typescript
// Add to app/routes/app.tsx temporarily
import { onLCP, onINP } from 'web-vitals';

if (typeof window !== 'undefined') {
  onLCP((metric) => console.log('LCP:', metric.value, 'ms'));
  onINP((metric) => console.log('INP:', metric.value, 'ms'));
}
```

Install: `npm install web-vitals`

Remove this before submitting to the App Store.

### Method 4: Performance Observer API (No Dependencies)

```javascript
// Paste in browser console while using the app
new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    if (entry.entryType === 'largest-contentful-paint') {
      console.log(`LCP: ${entry.startTime.toFixed(0)}ms`, entry.element);
    }
  }
}).observe({ type: 'largest-contentful-paint', buffered: true });

new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    console.log(`INP candidate: ${entry.duration}ms`, entry.name);
  }
}).observe({ type: 'event', buffered: true, durationThreshold: 16 });
```

## What To Do If Metrics Are Too High

### LCP > 2.5s — Fixes

| Cause | Fix |
|-------|-----|
| Slow loader data fetch | Add database indexes, reduce query scope |
| Large Polaris bundle | Already mitigated by Vite route-based splitting |
| Blocking CSS | Polaris styles are loaded via `?url` (non-blocking) — already correct |
| Slow Shopify admin embed | Out of your control; focus on app-side optimizations |
| Large product lists | Paginate (already implemented) |

### INP > 200ms — Fixes

| Cause | Fix |
|-------|-----|
| Heavy state updates on click | Use `useTransition` or `useDeferredValue` |
| Synchronous API calls blocking UI | Move to `fetcher.submit()` (already used) |
| Large re-renders | Memoize with `React.memo` / `useMemo` |
| Many DOM nodes | Virtualize long product lists |

## CatalogNow-Specific Expectations

The app should perform well because:
1. **SSR via Remix** — HTML is pre-rendered server-side, LCP element is in the initial response
2. **Route-based code splitting** — Only loads JS for the current route
3. **Polaris components** — Optimized, lightweight UI components
4. **Loader pattern** — Data is fetched server-side before rendering, no client-side waterfalls
5. **No heavy client-side JS** — No large charts, maps, or canvas rendering

Expected results for a typical product list page:
- LCP: ~800-1500ms (depending on Cloudflare tunnel latency in dev)
- INP: ~50-100ms (Polaris buttons are lightweight)
