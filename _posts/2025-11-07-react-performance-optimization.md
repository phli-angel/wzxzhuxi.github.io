---
layout: post
title: "Optimizing React Performance: Our Top 5 Techniques"
date: 2025-11-07 14:20:00 +0800
categories: frontend react performance
author: Jordan Kim
---

Performance matters. Users expect instant feedback, and every millisecond counts. Here are the five techniques that gave us the biggest performance wins in our React app.

## 1. Code Splitting with React.lazy()

**Problem**: Our initial bundle was 2.3MB. Users waited 8+ seconds on 3G.

**Solution**: Route-based code splitting.

```jsx
import { lazy, Suspense } from 'react';

const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  );
}
```

**Result**: Initial load reduced to 380KB. TTI improved by 73%.

## 2. Memoization (But Don't Overdo It)

Use `useMemo` and `React.memo` where it actually matters - expensive calculations and frequent re-renders.

```jsx
const ExpensiveList = React.memo(({ items, filter }) => {
  const filtered = useMemo(() =>
    items.filter(item => item.category === filter),
    [items, filter]
  );

  return filtered.map(item => <Item key={item.id} {...item} />);
});
```

**Pro tip**: Profile first. We wasted time memoizing components that rendered once.

## 3. Virtual Scrolling for Long Lists

Rendering 10,000 DOM nodes? No. Use `react-window` or `react-virtual`.

```jsx
import { FixedSizeList } from 'react-window';

const Row = ({ index, style }) => (
  <div style={style}>Item {index}</div>
);

<FixedSizeList
  height={600}
  itemCount={10000}
  itemSize={50}
  width="100%"
>
  {Row}
</FixedSizeList>
```

**Result**: Scroll performance at 60fps with 50k items.

## 4. Debounce Input Handlers

Search-as-you-type was hammering our API. Debouncing saved us.

```jsx
import { useDebouncedCallback } from 'use-debounce';

const SearchInput = () => {
  const debounced = useDebouncedCallback(
    (value) => fetchResults(value),
    300
  );

  return <input onChange={e => debounced(e.target.value)} />;
};
```

**Result**: API calls reduced by 85%.

## 5. Optimize Images

- Use WebP format (70% smaller than JPEG)
- Implement lazy loading with Intersection Observer
- Responsive images with `srcset`

```jsx
<img
  loading="lazy"
  srcSet="small.webp 480w, medium.webp 800w, large.webp 1200w"
  sizes="(max-width: 600px) 480px, 800px"
  alt="Product image"
/>
```

## Measurement Tools

- **Lighthouse**: CI integration, set performance budgets
- **React DevTools Profiler**: Find unnecessary renders
- **Bundle Analyzer**: Identify bloat

## Bottom Line

Performance optimization is about priorities:

1. Measure first (use real user metrics)
2. Fix the biggest bottlenecks
3. Validate improvements
4. Repeat

Our LCP went from 4.2s to 1.1s. Worth it.

---

*Questions about React performance? Let's discuss in the comments!*
