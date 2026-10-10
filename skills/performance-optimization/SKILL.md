---
name: performance-optimization
description: Use when optimizing application performance. Covers database query optimization, caching strategies, API optimization, frontend code splitting, lazy loading, and monitoring.
version: 1.0.0
updated: 2026-06-21
---

# Performance Optimization

## Tujuan
Memastikan aplikasi berjalan dengan performa optimal, scalable, dan efisien dalam penggunaan resources.

## Prinsip Dasar

### 1. Measure First, Optimize Later
- Jangan optimize prematurely
- Gunakan profiling tools untuk identify bottlenecks
- Set performance budgets dan monitor
- A/B test optimization changes

### 2. The Three R's
- **Reduce**: Kurangi payload size, requests, queries
- **Reuse**: Cache, connection pooling, object pooling
- **Recycle**: Clean up resources, garbage collection

## Database Optimization

### 1. Query Optimization

❌ **N+1 Query Problem:**
```php
// Bad: N+1 queries (1 + 100 queries)
$users = User::all(); // 1 query
foreach ($users as $user) {
    echo $user->profile->bio; // 100 queries jika ada 100 users
}
```

✅ **Eager Loading:**
```php
// Good: 2 queries only
$users = User::with('profile')->get(); // 2 queries total
foreach ($users as $user) {
    echo $user->profile->bio;
}
```

✅ **Use Indexes:**
```sql
-- Create index untuk columns yang sering di-query
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_orders_user_status ON orders(user_id, status);
CREATE INDEX idx_products_created ON products(created_at DESC);

-- Composite index untuk multi-column queries
CREATE INDEX idx_logs_user_date ON logs(user_id, created_at);
```

✅ **Avoid SELECT *:**
```php
// Bad: Fetch semua columns
$users = DB::table('users')->get();

// Good: Hanya columns yang dibutuhkan
$users = DB::table('users')->select('id', 'name', 'email')->get();
```

✅ **Pagination:**
```php
// Jangan load semua data sekaligus
$products = Product::paginate(20); // 20 items per page

// Cursor pagination untuk large datasets
$products = Product::cursorPaginate(20);
```

✅ **Query Caching:**
```php
// Cache expensive queries
$stats = Cache::remember('dashboard-stats', 3600, function () {
    return DB::table('orders')
        ->selectRaw('COUNT(*) as total, SUM(amount) as revenue')
        ->where('status', 'completed')
        ->first();
});
```

### 2. Database Connection Pooling

```php
// Gunakan persistent connections
'mysql' => [
    'driver' => 'mysql',
    'options' => [
        PDO::ATTR_PERSISTENT => true,
    ],
],
```

## Caching Strategy

### 1. Application Cache (Redis/Memcached)

```php
// Cache user profile
$user = Cache::remember("user.{$userId}", 3600, function () use ($userId) {
    return User::with('profile', 'roles')->find($userId);
});

// Cache busting ketika data berubah
User::updated(function ($user) {
    Cache::forget("user.{$user->id}");
});

// Tag-based caching untuk mass invalidation
Cache::tags(['users', 'profiles'])->put("user.{$userId}", $user, 3600);
Cache::tags(['users'])->flush(); // Clear semua user cache
```

### 2. HTTP Cache

```php
// Browser cache dengan ETag
return response($content)
    ->header('Cache-Control', 'public, max-age=3600')
    ->header('ETag', md5($content));

// Conditional requests
if ($request->header('If-None-Match') === $etag) {
    return response('', 304); // Not Modified
}
```

### 3. CDN untuk Static Assets

```
- Images, CSS, JS di CDN
- Use versioning untuk cache busting
- Compress assets (gzip, brotli)
- Lazy load images
```

## API Optimization

### 1. Response Size Optimization

```php
// Pagination
$users = User::paginate(20);

// Field filtering (GraphQL-style)
// GET /api/users?fields=id,name,email
$fields = $request->input('fields', '*');
$users = User::select(explode(',', $fields))->get();

// Compression
return response($data)
    ->header('Content-Encoding', 'gzip');
```

### 2. Async Processing

```php
// Background jobs untuk heavy tasks
dispatch(new ProcessVideoJob($video));
dispatch(new SendEmailJob($user))->delay(now()->addMinutes(5));

// Queue prioritization
dispatch(new CriticalJob)->onQueue('high');
dispatch(new RegularJob)->onQueue('default');
dispatch(new LowPriorityJob)->onQueue('low');
```

### 3. API Rate Limiting

```php
// Prevent abuse & ensure fair usage
Route::middleware('throttle:60,1')->group(function () {
    // 60 requests per minute
});
```

## Frontend Optimization

### 1. Code Splitting & Lazy Loading

```javascript
// React lazy loading
const Dashboard = React.lazy(() => import('./Dashboard'));

// Vue lazy loading
const Dashboard = () => import('./Dashboard.vue');

// Image lazy loading
<img src="image.jpg" loading="lazy" />
```

### 2. Bundle Size Optimization

```javascript
// Tree shaking - import only what you need
import { debounce } from 'lodash'; // ❌ imports entire lodash
import debounce from 'lodash/debounce'; // ✅ import only debounce

// Analyze bundle size
npm run build -- --analyze
```

### 3. Memoization

```javascript
// React useMemo
const expensiveValue = useMemo(() => {
    return complexCalculation(data);
}, [data]);

// React useCallback
const handleClick = useCallback(() => {
    doSomething(id);
}, [id]);
```

## Server-Side Optimization

### 1. PHP Optimization

```php
// Use OPcache
opcache.enable=1
opcache.memory_consumption=256
opcache.max_accelerated_files=20000

// Composer autoload optimization
composer dump-autoload --optimize --classmap-authoritative

// Use generators untuk large datasets
function getUsers() {
    $users = User::cursor(); // Generator
    foreach ($users as $user) {
        yield $user;
    }
}
```

### 2. Resource Cleanup

```php
// Close connections
DB::disconnect();

// Unset large variables
unset($largeArray);

// Clear collections
$collection->clear();
```

### 3. Optimize Configuration

```php
// Production .env
APP_DEBUG=false
APP_LOG_LEVEL=error

// Cache configs
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

## Monitoring & Profiling

### 1. Application Performance Monitoring (APM)

```php
// Laravel Telescope untuk development
// New Relic, Datadog, Scout untuk production

// Custom timing
$startTime = microtime(true);
// ... code execution ...
$executionTime = microtime(true) - $startTime;
Log::info("Operation took {$executionTime} seconds");
```

### 2. Database Query Monitoring

```php
// Log slow queries
DB::listen(function ($query) {
    if ($query->time > 1000) { // > 1 second
        Log::warning('Slow query detected', [
            'sql' => $query->sql,
            'bindings' => $query->bindings,
            'time' => $query->time,
        ]);
    }
});
```

### 3. Performance Metrics

Monitor key metrics:
```
- Response time (p50, p95, p99)
- Throughput (requests per second)
- Error rate
- Database query time
- Memory usage
- CPU usage
- Cache hit ratio
```

## Load Testing

```bash
# Apache Bench
ab -n 1000 -c 100 http://localhost/api/endpoint

# k6
k6 run --vus 100 --duration 30s script.js

# Artillery
artillery quick --count 100 --num 10 http://localhost/api/endpoint
```

## Performance Checklist

```
Database
□ Indexes dibuat untuk frequently queried columns
□ N+1 queries resolved dengan eager loading
□ Pagination implemented
□ Connection pooling enabled
□ Slow query monitoring active

Caching
□ Application cache (Redis/Memcached) configured
□ Cache strategy implemented
□ Cache invalidation handled properly
□ HTTP caching headers set
□ CDN configured untuk static assets

API
□ Response pagination
□ Field filtering available
□ Compression enabled
□ Rate limiting configured
□ Background jobs untuk heavy tasks

Frontend
□ Code splitting implemented
□ Lazy loading untuk routes & images
□ Bundle size optimized
□ Assets compressed (gzip/brotli)
□ Memoization untuk expensive calculations

Monitoring
□ APM tool configured
□ Slow queries logged
□ Performance metrics tracked
□ Alerts configured
□ Load testing performed
```

## Performance Targets

Set realistic targets berdasarkan application type:
```
Web Application:
- Page load: < 3 seconds
- API response: < 200ms (p95)
- Database query: < 100ms (p95)

Real-time Application:
- API response: < 100ms (p95)
- WebSocket latency: < 50ms
```
