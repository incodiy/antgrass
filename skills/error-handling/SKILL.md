---
name: error-handling
description: Use when implementing error handling or logging. Enforces custom exception hierarchy, structured logging, try-catch best practices, and proper error response formats.
version: 1.0.0
updated: 2026-06-21
---

# Error Handling & Logging

## Tujuan
Implement consistent error handling dan comprehensive logging untuk debugging, monitoring, dan maintainability.

## Prinsip Dasar

### 1. Fail Fast, Fail Loud
- Detect errors early
- Don't hide errors
- Propagate meaningful error messages

### 2. Graceful Degradation
- Application tetap functional meski ada error
- Fallback mechanisms
- User-friendly error messages

### 3. Separation of Concerns
- Business logic errors vs technical errors
- Different handling untuk different error types
- Centralized error handling

## Exception Hierarchy

### Custom Exceptions

```php
// Base exception
abstract class ApplicationException extends Exception
{
    protected $errorCode;
    protected $statusCode = 500;
    
    public function getErrorCode(): string
    {
        return $this->errorCode;
    }
    
    public function getStatusCode(): int
    {
        return $this->statusCode;
    }
}

// Business logic exceptions
class ValidationException extends ApplicationException
{
    protected $statusCode = 422;
    protected $errorCode = 'VALIDATION_ERROR';
}

class AuthenticationException extends ApplicationException
{
    protected $statusCode = 401;
    protected $errorCode = 'AUTHENTICATION_FAILED';
}

class AuthorizationException extends ApplicationException
{
    protected $statusCode = 403;
    protected $errorCode = 'UNAUTHORIZED_ACTION';
}

class ResourceNotFoundException extends ApplicationException
{
    protected $statusCode = 404;
    protected $errorCode = 'RESOURCE_NOT_FOUND';
}

class BusinessRuleException extends ApplicationException
{
    protected $statusCode = 400;
    protected $errorCode = 'BUSINESS_RULE_VIOLATION';
}

// Technical exceptions
class DatabaseException extends ApplicationException
{
    protected $statusCode = 500;
    protected $errorCode = 'DATABASE_ERROR';
}

class ExternalServiceException extends ApplicationException
{
    protected $statusCode = 503;
    protected $errorCode = 'EXTERNAL_SERVICE_UNAVAILABLE';
}
```

### Exception Usage

```php
// Throw custom exceptions
if (!$user) {
    throw new ResourceNotFoundException('User tidak ditemukan');
}

if ($order->total > $user->balance) {
    throw new BusinessRuleException('Saldo tidak mencukupi');
}

if (!$this->authorize($user, 'delete', $post)) {
    throw new AuthorizationException('Anda tidak memiliki izin untuk menghapus post ini');
}
```

## Error Response Format

### Standard JSON Response

```php
// Success response
{
    "success": true,
    "message": "Operasi berhasil",
    "data": {
        "id": "uuid",
        "name": "John Doe"
    }
}

// Error response
{
    "success": false,
    "message": "Terjadi kesalahan",
    "error_code": "VALIDATION_ERROR",
    "errors": {
        "email": ["Email wajib diisi", "Format email tidak valid"],
        "password": ["Password minimal 8 karakter"]
    },
    "trace_id": "abc-123-def" // untuk tracking
}
```

### Global Exception Handler

```php
// Laravel App\Exceptions\Handler
public function register()
{
    $this->renderable(function (ApplicationException $e, $request) {
        return response()->json([
            'success' => false,
            'message' => $e->getMessage(),
            'error_code' => $e->getErrorCode(),
            'trace_id' => $request->header('X-Request-ID'),
        ], $e->getStatusCode());
    });
    
    $this->renderable(function (ValidationException $e, $request) {
        return response()->json([
            'success' => false,
            'message' => 'Validasi gagal',
            'error_code' => 'VALIDATION_ERROR',
            'errors' => $e->errors(),
        ], 422);
    });
}
```

## Try-Catch Best Practices

### Specific Exceptions

```php
// ❌ Bad: Catch generic exception
try {
    $user = $this->userService->create($data);
} catch (Exception $e) {
    // Too broad, catches everything
    return response()->json(['error' => 'Something went wrong'], 500);
}

// ✅ Good: Catch specific exceptions
try {
    $user = $this->userService->create($data);
} catch (ValidationException $e) {
    return response()->json(['error' => $e->getMessage()], 422);
} catch (DatabaseException $e) {
    Log::error('Database error during user creation', [
        'error' => $e->getMessage(),
        'data' => $data,
    ]);
    return response()->json(['error' => 'Terjadi kesalahan sistem'], 500);
}
```

### Don't Swallow Exceptions

```php
// ❌ Bad: Silent failure
try {
    $this->sendEmail($user);
} catch (Exception $e) {
    // Error hilang, tidak ada log
}

// ✅ Good: Log dan handle
try {
    $this->sendEmail($user);
} catch (EmailException $e) {
    Log::error('Failed to send email', [
        'user_id' => $user->id,
        'error' => $e->getMessage(),
    ]);
    
    // Queue untuk retry
    dispatch(new SendEmailJob($user))->delay(now()->addMinutes(5));
}
```

### Cleanup Resources

```php
// ✅ Always cleanup dalam finally
$connection = null;
try {
    $connection = $this->openConnection();
    $connection->process($data);
} catch (ConnectionException $e) {
    Log::error('Connection failed', ['error' => $e->getMessage()]);
    throw $e;
} finally {
    // Cleanup resources
    if ($connection) {
        $connection->close();
    }
}
```

## Logging Best Practices

### Log Levels

```php
// DEBUG: Detailed debug information
Log::debug('User query executed', ['sql' => $query, 'time' => $executionTime]);

// INFO: Interesting events
Log::info('User logged in', ['user_id' => $user->id, 'ip' => $request->ip()]);

// NOTICE: Normal but significant
Log::notice('Password reset requested', ['user_id' => $user->id]);

// WARNING: Exceptional occurrences yang tidak error
Log::warning('API rate limit approaching', ['user_id' => $user->id, 'count' => 95]);

// ERROR: Runtime errors yang tidak require immediate action
Log::error('Failed to process payment', [
    'order_id' => $order->id,
    'error' => $e->getMessage(),
    'trace' => $e->getTraceAsString(),
]);

// CRITICAL: Critical conditions
Log::critical('Database connection lost', ['host' => config('database.host')]);

// ALERT: Action must be taken immediately
Log::alert('Disk space critically low', ['available' => $diskSpace]);

// EMERGENCY: System is unusable
Log::emergency('Application crashed', ['error' => $e->getMessage()]);
```

### Structured Logging

```php
// ✅ Good: Structured, searchable logs
Log::info('Order created', [
    'order_id' => $order->id,
    'user_id' => $user->id,
    'total' => $order->total,
    'items_count' => $order->items->count(),
    'payment_method' => $order->payment_method,
    'ip_address' => $request->ip(),
    'user_agent' => $request->userAgent(),
    'timestamp' => now()->toIso8601String(),
]);

// ❌ Bad: Unstructured
Log::info("Order {$order->id} created by user {$user->id}");
```

### Context & Correlation

```php
// Add request ID untuk trace across services
class AddRequestIdMiddleware
{
    public function handle($request, Closure $next)
    {
        $requestId = $request->header('X-Request-ID', Str::uuid());
        $request->headers->set('X-Request-ID', $requestId);
        
        Log::withContext(['request_id' => $requestId]);
        
        return $next($request);
    }
}

// Logs akan include request_id automatically
Log::info('Processing order'); // Include request_id in context
```

### What to Log

```php
// ✅ DO log:
- Authentication events (login, logout, failed attempts)
- Authorization failures
- Input validation errors
- Business logic errors
- External service calls & responses
- Database errors
- Performance issues (slow queries)
- Security events
- Application start/stop

// ❌ DON'T log:
- Passwords (plaintext atau hashed)
- Credit card numbers
- Social security numbers
- API keys / secrets
- Session tokens
- Personal identifiable information (PII) - be careful

// Sanitize sensitive data
Log::info('User updated', [
    'user_id' => $user->id,
    'email' => $this->maskEmail($user->email), // john***@example.com
    'ip' => $this->maskIp($request->ip()),    // 192.168.***.***
]);
```

### Log Channels

```php
// config/logging.php
'channels' => [
    'stack' => [
        'driver' => 'stack',
        'channels' => ['daily', 'slack'],
    ],
    
    'security' => [
        'driver' => 'daily',
        'path' => storage_path('logs/security.log'),
        'level' => 'info',
        'days' => 90,
    ],
    
    'audit' => [
        'driver' => 'daily',
        'path' => storage_path('logs/audit.log'),
        'level' => 'info',
        'days' => 365, // Keep audit logs longer
    ],
];

// Usage
Log::channel('security')->warning('Failed login attempt', [
    'email' => $email,
    'ip' => $request->ip(),
]);

Log::channel('audit')->info('User data modified', [
    'user_id' => $user->id,
    'changed_by' => auth()->id(),
    'changes' => $changes,
]);
```

## Error Monitoring & Alerting

### Integration dengan Monitoring Tools

```php
// Sentry, Bugsnag, Rollbar integration
public function report(Throwable $e)
{
    if ($this->shouldReport($e)) {
        // Send to monitoring service
        app('sentry')->captureException($e);
    }
    
    parent::report($e);
}
```

### Alert Configuration

```php
// Slack notification untuk critical errors
'slack' => [
    'driver' => 'slack',
    'url' => env('LOG_SLACK_WEBHOOK_URL'),
    'username' => 'Error Logger',
    'emoji' => ':boom:',
    'level' => 'critical',
],

// Email notification
if (app()->environment('production') && $exception instanceof CriticalException) {
    Mail::to('dev-team@example.com')->send(new CriticalErrorNotification($exception));
}
```

## Frontend Error Handling

### JavaScript

```javascript
// Global error handler
window.addEventListener('error', (event) => {
    logError({
        message: event.message,
        source: event.filename,
        line: event.lineno,
        column: event.colno,
        stack: event.error?.stack,
    });
});

// Promise rejection handler
window.addEventListener('unhandledrejection', (event) => {
    logError({
        message: event.reason,
        type: 'unhandled_promise_rejection',
    });
});

// API error handling
async function fetchData() {
    try {
        const response = await fetch('/api/data');
        
        if (!response.ok) {
            throw new ApiError('Request failed', response.status);
        }
        
        return await response.json();
    } catch (error) {
        if (error instanceof ApiError) {
            showUserMessage('Gagal memuat data. Silakan coba lagi.');
        } else {
            showUserMessage('Terjadi kesalahan. Silakan refresh halaman.');
        }
        
        logError(error);
        throw error;
    }
}
```

## 🚨 Self-Enforcement — WAJIB di Setiap Error Handling

**Sebelum menulis kode error handling, WAJIB cek:**
1. Apakah exception sudah spesifik (bukan generic `Exception`)? JANGAN catch-all.
2. Apakah error response sesuai format standar (success, message, error_code, errors)?
3. Apakah ada logging untuk setiap catch block? JANGAN swallow exception.
4. Apakah sensitive data TIDAK masuk ke log atau response?
5. Apakah try-catch sudah ada finally block untuk cleanup resources?

**JIKA ada aturan yang dilanggar → koreksi SEBELUM lanjut ke task lain.**

---

## Error Handling Checklist

```
Exception Handling
□ Custom exceptions untuk different error types
□ Specific exception catching (avoid generic Exception)
□ Proper error propagation
□ Resource cleanup in finally blocks
□ User-friendly error messages

Logging
□ Structured logging dengan context
□ Appropriate log levels
□ Request ID untuk correlation
□ Separate log channels (security, audit, etc)
□ No sensitive data in logs
□ Log rotation configured

Monitoring
□ Error monitoring tool integrated (Sentry, etc)
□ Alerts configured untuk critical errors
□ Dashboard untuk error tracking
□ Regular review of error logs

User Experience
□ Graceful error handling
□ Meaningful error messages
□ Fallback mechanisms
□ Loading states & error states
□ Retry mechanisms where appropriate
```
