---
name: senior-backend-engineer
description: Use when designing backend architecture, APIs, database schemas, authentication, caching, DevOps, and system design. Framework-agnostic — applies to Laravel, Python, Go, Node.js, and any backend stack. Covers security, performance, resilience, and production-ready patterns.
version: 1.1.0
updated: 2026-07-13
loads: [openapi-first, security-first, database-design, performance-optimization, error-handling, code-standards]
lazy_load_sections:
  core_always: ["Layered Architecture", "API Design", "Database Patterns", "Error Handling"]
  on_demand:
    caching: ["Redis", "Memcached", "Cache Invalidation", "CDN"]
    devops: ["Docker", "CI/CD", "Monitoring", "Logging"]
    microservices: ["Service Mesh", "Event-Driven", "Saga Pattern", "CQRS"]
    performance: ["Query Optimization", "Connection Pooling", "Load Balancing"]
---

# Senior Backend Engineer

## 🔴 PERINGATAN KRITIKAL — WAJIB DIIKUTI TANPA PENGECUALIAN

**Skill ini adalah standar emas untuk semua kode backend yang kamu tulis.** Framework-agnostic — pola dan prinsip di sini berlaku untuk PHP, Python, Go, Node.js, Java, atau stack apapun. Setiap pelanggaran bisa menyebabkan aplikasi crash, data korup, security breach, atau bencana production yang sulit dipulihkan.

🔴 **Konsekuensi jika dilanggar:**
- **Layered architecture dilanggar (logic di controller)** → kode gak bisa di-test, tiap ganti framework harus rewrite semua
- **SQL injection / input tanpa validasi** → data breach, jutaan data bocor, ganti rugi milyaran
- **Migration tanpa rollback** → data korup permanen, recovery butuh waktu berhari-hari
- **N+1 query / connection leak** → server collapse di traffic tinggi, aplikasi mati total
- **Error handling asal-asalan** → debug production jadi nightmare, stack trace bocor ke user

Model **WAJIB** mematuhi setiap aturan di bawah. **JANGAN skip, JANGAN tawar-menawar.**

---

## 📌 Tujuan

Skill ini memastikan setiap backend yang ditulis memiliki **arsitektur bersih (clean architecture), security-hardened, production-ready, dan scalable**. Mencakup separation of concerns di setiap layer, resource optimization, error resilience, serta pattern modern untuk microservices, event-driven, dan distributed systems.

---

# 🏗️ API Architecture & Separation of Concerns

---

### 1. Layered Architecture (Controller → Service → Repository → DTO)

**JANGAN PERNAH** menaruh business logic di controller. **WAJIB** pisahkan tanggung jawab ke layer yang tepat.

```
HTTP Request
    │
    ▼
┌─────────────┐
│  Controller  │ ← Only: parse input, call service, return response
├─────────────┤
│   Service    │ ← Business logic, orchestration, rules
├─────────────┤
│  Repository  │ ← Data access, ORM queries, raw SQL
├─────────────┤
│  DTO / Transformer │ ← Shape data for input/output
└─────────────┘
    │
    ▼
  Response
```

❌ **Salah:**
```php
// Controller — logic di sini? Gak bisa di-test, gak reusable
class UserController {
    public function store(Request $request) {
        $validated = $request->validate([/* ... */]);
        
        // Business logic di controller?!
        if (User::where('email', $validated['email'])->exists()) {
            return response()->json(['error' => 'Email already exists'], 409);
        }
        
        $user = User::create($validated);
        
        // Side effect di controller?!
        Mail::send(new WelcomeEmail($user));
        
        return response()->json($user, 201);
    }
}
```

✅ **Benar (Layered):**
```
Controller
    │  Parsing request, call service, return response
    ▼
UserService
    │  Business rules: check duplicate, orchestrasi
    ▼
UserRepository
    │  Data access: create user query
    ▼
UserDTO
    │  Shape response: hide password, format dates
```

```python
# controllers/user_controller.py
@router.post('/users')
def create_user(payload: CreateUserDTO):
    user = user_service.register(payload)
    return UserResponseDTO.from_model(user), 201
```

```python
# services/user_service.py
class UserService:
    def register(self, dto: CreateUserDTO) -> User:
        if self.repo.exists_by_email(dto.email):
            raise BusinessRuleException('Email already registered')
        
        user = self.repo.create(dto.to_entity())
        self.event_dispatcher.dispatch(UserRegisteredEvent(user.id))
        return user
```

```python
# repositories/user_repository.py
class UserRepository:
    def create(self, entity: User) -> User:
        return UserModel.create(entity.to_dict())
    
    def exists_by_email(self, email: str) -> bool:
        return UserModel.where('email', email).exists()
```

```python
# dtos/user_dto.py
class CreateUserDTO:
    email: str
    password: str
    full_name: str
    
    def to_entity(self) -> User:
        return User(email=self.email, password_hash=hash_password(self.password), ...)

class UserResponseDTO:
    id: str
    email: str
    full_name: str
    created_at: str
    
    @staticmethod
    def from_model(user: User) -> 'UserResponseDTO':
        return UserResponseDTO(
            id=user.id, email=user.email,
            full_name=user.full_name,
            created_at=user.created_at.isoformat()
        )
```

---

### 2. RESTful API Design

**Prinsip inti:** Resource-based URIs, consistent HTTP methods, appropriate status codes.
**Detail lengkap:** Lihat skill `openapi-first` — REST conventions, URI design, status codes, best practices.

---

### 3. Input/Output DTO & Data Transformation

**Prinsip inti:** Tiga layer data (Input DTO → Entity → Output DTO), jangan ekspose entity langsung.
**Detail lengkap:** Lihat skill `openapi-first` — DTO patterns, data transformation, request/response schemas.

---

### 4. Pagination & Filtering Strategy

**WAJIB** gunakan pagination untuk semua endpoint yang mengembalikan list. Pilih strategi pagination yang sesuai dengan skala data.

❌ **Salah:**
```sql
-- Offset pagination tanpa index — scan full table tiap page
SELECT * FROM users LIMIT 10 OFFSET 10000;
-- Makin besar offset, makin lambat (masih baca semua row sebelumnya)
```

✅ **Benar — Pilih strategi sesuai konteks:**

```python
# 1. Offset Pagination — cocok untuk UI dengan page selector
#    Kelebihan: bisa lompat ke page berapa aja
#    Kekurangan: slow di offset besar, data inconsistent kalo ada insert/delete
GET /api/users?page=2&per_page=20

# Response:
{
    "data": [...],
    "meta": {
        "current_page": 2,
        "per_page": 20,
        "total": 1500,
        "last_page": 75
    }
}

# 2. Cursor Pagination — cocok untuk infinite scroll, real-time feed
#    Kelebihan: fast, consistent meski ada insert/delete
#    Kekurangan: gak bisa lompat ke page sembarang
GET /api/users?cursor=eyJpZCI6MTAwfQ==&limit=20
# cursor = base64 encoded last item identifier

# Response:
{
    "data": [...],
    "meta": {
        "next_cursor": "eyJpZCI6MTIwfQ==",
        "has_more": true
    }
}

# Implementation cursor:
SELECT * FROM users
WHERE id > :cursor   -- indexed column!
ORDER BY id ASC
LIMIT :limit;
```

**Filter conventions:**
```python
# Filter via query params — konsisten
GET /api/orders?status=completed&min_total=100000&max_total=500000
GET /api/orders?created_after=2024-01-01&created_before=2024-12-31
GET /api/users?search=john&sort=-created_at,full_name
# sort = -created_at (desc), full_name (asc)

# Backend:
class OrderFilter:
    status: str | None
    min_total: float | None
    max_total: float | None
    created_after: date | None
    created_before: date | None
    sort: str = '-created_at'
    page: int = 1
    per_page: int = 20
```

---

### 5. API Versioning & Documentation (OpenAPI)

**Prinsip inti:** Version API sejak awal, dokumentasi dengan OpenAPI/Swagger.
**Detail lengkap:** Lihat skill `openapi-first` — versioning strategies, OpenAPI spec, auto-docs setup.

---

# 🔐 Security & Authentication

---

### 6. Authentication Strategy

**WAJIB** pilih strategi autentikasi sesuai konteks. **JANGAN PERNAH** simpan password plaintext.

❌ **Salah:**
```python
# Password MD5 — 2024, really?
user.password_hash = md5(password)
```

✅ **Benar:**
```python
# Hash password dengan bcrypt/argon2
from passlib.hash import bcrypt

user.password_hash = bcrypt.hash(password)

def verify(password: str, hash: str) -> bool:
    return bcrypt.verify(password, hash)
```

**Strategi autentikasi:**

| Approach | Use Case | Token Storage | Notes |
|----------|----------|---------------|-------|
| **JWT Access + Refresh** | SPA, Mobile | Memory (access) + httpOnly cookie (refresh) | Short-lived access (15min), refresh rotasi |
| **Session + Cookie** | Server-rendered apps | Server memory/Redis + signed cookie | Stateful, mudah revoke |
| **OAuth2** | Third-party login | Tergantung flow | Google, GitHub, etc |
| **API Key** | Machine-to-machine | Request header + server-side lookup | Simple, no user context |
| **PAT (Personal Access Token)** | Developer API access | Stored hashed in DB | Like GitHub tokens |

```go
// JWT flow — access + refresh
type TokenPair struct {
    AccessToken  string `json:"accessToken"`
    RefreshToken string `json:"refreshToken"`
}

func GenerateTokenPair(userID string) (*TokenPair, error) {
    accessToken, err := jwt.Generate(jwt.Claims{
        Subject: userID,
        ExpiresAt: time.Now().Add(15 * time.Minute),
    })
    
    refreshToken := generateRandomString(64)
    // Store refresh token hash in DB (not plaintext!)
    refreshStore.Save(userID, hash(refreshToken), 7 * 24 * time.Hour)
    
    return &TokenPair{AccessToken: accessToken, RefreshToken: refreshToken}, nil
}

func RefreshTokens(refreshToken string) (*TokenPair, error) {
    storedHash := refreshStore.Get(userID)  // from token lookup
    if !verify(refreshToken, storedHash) {
        return nil, ErrInvalidToken
    }
    
    // Rotate: revoke old refresh, issue new
    refreshStore.Delete(userID)
    return GenerateTokenPair(userID)
}
```

---

### 7. Authorization & RBAC

**SELALU** cek authorization di setiap endpoint. **JANGAN** cuma di frontend.

❌ **Salah:**
```python
# Cuma cek "is logged in" — tapi gak cek "can they do this?"
@router.delete('/posts/{id}')
def delete_post(id: str, current_user: User = Depends(get_current_user)):
    post = PostModel.find(id)
    # Siapa pun bisa delete post siapa pun!
    post.delete()
```

✅ **Benar:**
```python
# RBAC middleware + policy check

# 1. Define roles & permissions
ROLES = {
    'admin':   ['read', 'create', 'update', 'delete', 'manage_users'],
    'editor':  ['read', 'create', 'update'],
    'viewer':  ['read'],
}

# 2. Middleware/guard
def require_permission(permission: str):
    def decorator(current_user: User = Depends(get_current_user)):
        if permission not in current_user.permissions:
            raise ForbiddenException('Insufficient permissions')
        return current_user
    return decorator

# 3. Policy — resource-specific check (ownership, scoping)
class PostPolicy:
    @staticmethod
    def can_delete(user: User, post: Post) -> bool:
        return user.role == 'admin' or post.author_id == user.id

# 4. Apply di endpoint
@router.delete('/posts/{id}')
def delete_post(
    id: str,
    current_user: User = Depends(require_permission('delete'))
):
    post = post_service.get_by_id(id)
    
    if not PostPolicy.can_delete(current_user, post):
        raise ForbiddenException('Cannot delete this post')
    
    post_service.delete(id)
```

---

### 8. Security Hardening

**Prinsip inti:** Proteksi terhadap SQL injection, XSS, CSRF, CORS, dan input validation.
**Detail lengkap:** Lihat skill `security-first` — OWASP Top 10, prepared statements, security headers, CORS config, input validation.

---

# 🗄️ Data & Resources Optimization

---

### 9. Database Schema Design

**Prinsip inti:** UUID PKs, soft delete, audit fields, proper indexing, data types.
**Detail lengkap:** Lihat skill `database-design` — table design, naming conventions, indexing strategy, relationships.

---

### 10. Query Optimization (N+1, Connection Pool, EXPLAIN)

**Prinsip inti:** Hindari N+1 queries, gunakan connection pooling, pahami EXPLAIN plans.
**Detail lengkap:** Lihat skill `performance-optimization` — query tuning, indexing, connection pooling, N+1 prevention.

---

### 11. Transactions & ACID

**Prinsip inti:** Gunakan transaction untuk operasi multi-record. Pahami saga pattern untuk distributed transactions.
**Detail lengkap:** Lihat skill `database-design` — transaction isolation levels, saga patterns, compensating transactions.

---

### 12. Database Seeding & Factories

**Prinsip inti:** Gunakan factories untuk generate test data. Pisahkan reference seeds dari development seeds.
**Detail lengkap:** Lihat skill `database-design` — factory patterns, seed strategies, reference data management.

---

### 13. Caching Strategy

**Prinsip inti:** Multi-level caching (in-memory → Redis → DB), cache-aside pattern, TTL-based invalidation.
**Detail lengkap:** Lihat skill `performance-optimization` — caching strategies, Redis patterns, invalidation, stale-while-revalidate.

---

### 14. File Storage & CDN

**JANGAN PERNAH** simpan file di server lokal untuk production. Gunakan object storage + CDN.

❌ **Salah:**
```python
# Simpan di server lokal — gak scalable, backup susah
def upload_avatar(file):
    path = f'/var/www/uploads/{file.filename}'
    file.save(path)
    return f'/uploads/{file.filename}'
```

✅ **Benar:**
```python
# Object storage (S3, GCS, MinIO) + CDN
import boto3

s3 = boto3.client('s3')

def upload_avatar(user_id: str, file) -> str:
    # 
    ext = file.filename.rsplit('.', 1)[-1].lower()
    key = f'avatars/{user_id}/{uuid4()}.{ext}'
    
    ALLOWED_EXTENSIONS = {'jpg', 'jpeg', 'png', 'webp'}
    if ext not in ALLOWED_EXTENSIONS:
        raise ValidationException('Invalid image format')
    
    if len(file.read()) > 5 * 1024 * 1024:  # 5MB limit
        raise ValidationException('File too large')
    file.seek(0)
    
    s3.upload_fileobj(
        file, BUCKET_NAME, key,
        ExtraArgs={'ContentType': file.content_type}
    )
    
    # Generate signed URL (temporary access)
    return s3.generate_presigned_url(
        'get_object',
        Params={'Bucket': BUCKET_NAME, 'Key': key},
        ExpiresIn=3600  # 1 hour
    )
```

---

# 🧪 Testing, Errors & Quality

---

### 15. Testing Pyramid

**WAJIB** gunakan testing pyramid. **Unit test harus dominan**, integration sebagai pelengkap, e2e minimal.

```
        /\          E2E (smoke test — 5%)
       /  \
      /    \        Integration (service + DB — 25%)
     /______\
    /        \      Unit (pure logic — 70%)
   /__________\
```

❌ **Salah:**
```python
# Hanya e2e — slow, brittle, gak precise
def test_user_flow():
    response = client.post('/api/users', json={...})
    assert response.status_code == 201
    
    response = client.get(f'/api/users/{user_id}')
    assert response.status_code == 200
    # 1 test = everything. Gagal = gak tahu kenapa
```

✅ **Benar:**
```python
# Unit test — pure business logic, no I/O
def test_calculate_discount():
    service = DiscountService()
    
    assert service.calculate(100_000, 'VIP') == 20_000  # 20%
    assert service.calculate(50_000, 'REGULAR') == 2_500  # 5%
    assert service.calculate(10_000, 'NONE') == 0

# Integration test — service + DB
def test_create_user_with_duplicate_email():
    service = UserService(UserRepository(db_session))
    service.register(CreateUserDTO(email='same@mail.com', ...))
    
    with pytest.raises(BusinessRuleException):
        service.register(CreateUserDTO(email='same@mail.com', ...))

# E2E — happy path only
def test_user_registration_flow():
    response = client.post('/api/v1/users', json={
        'email': 'new@user.com', 'password': 'Str0ng!Pass', 'full_name': 'New User'
    })
    assert response.status_code == 201
    assert 'id' in response.json
```

---

### 16. Advanced Error Handling

**Prinsip inti:** Custom exception hierarchy + global handler. Jangan leak stack trace ke production.
**Detail lengkap:** Lihat skill `error-handling` — exception hierarchy, global handlers, structured error responses, RFC 7807.

---

### 17. Structured Logging & Monitoring

**Prinsip inti:** Structured logging (JSON), correlation ID for distributed tracing. Bukan print/echo.
**Detail lengkap:** Lihat skill `error-handling` — structured logging patterns, correlation IDs, log levels, monitoring integration.

---

### 18. Third-party API Integration

**WAJIB** gunakan circuit breaker + timeout per external service. **JANGAN** biarkan satu external API lambat ngerusak semuanya.

❌ **Salah:**
```python
# Single timeout untuk semua — external API lambat = semua request nunggu
def get_payment_status(order_id):
    response = requests.post(PAYMENT_GATEWAY_URL, json={...}, timeout=30)
    return response.json()
```

✅ **Benar:**
```python
import pybreaker
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

# Circuit breaker — stop calling jika API down
payment_breaker = pybreaker.CircuitBreaker(
    fail_max=5,           # 5 failures → open circuit
    reset_timeout=30,     # 30 detik coba lagi
)

@payment_breaker
def charge_payment(user_id: str, amount: int):
    response = requests.post(
        PAYMENT_GATEWAY_URL,
        json={'user_id': user_id, 'amount': amount},
        timeout=10,  # Per-service timeout!
    )
    response.raise_for_status()
    return response.json()

# Fallback — cache response terakhir
def get_exchange_rate(from_curr: str, to_curr: str) -> float:
    try:
        rate = fetch_live_rate(from_curr, to_curr)
        cache.set(f'exchange:{from_curr}:{to_curr}', rate, ttl=3600)
        return rate
    except (RequestException, CircuitBreakerError):
        cached = cache.get(f'exchange:{from_curr}:{to_curr}')
        if cached:
            logger.warn('using_stale_exchange_rate', from_curr=from_curr)
            return cached
        raise  # No fallback available — propagate error

# Retry with backoff
session = requests.Session()
retries = Retry(total=3, backoff_factor=0.5, status_forcelist=[500, 502, 503])
session.mount('https://', HTTPAdapter(max_retries=retries))
```

---

# 🚀 Async & Real-time

---

### 19. Background Jobs & Queues

**WAJIB** pindahkan task berat/lambat ke background queue. **JANGAN** proses di request-response cycle.

❌ **Salah:**
```python
# Send email di request — user nunggu 3 detik!
def register_user(dto):
    user = User.create(dto)
    send_email(user.email, 'Welcome!')  # 3 detik blocking!
    return user
```

✅ **Benar:**
```python
# Dispatch ke queue — langsung response, proses di background
def register_user(dto):
    user = User.create(dto)
    dispatch(SendWelcomeEmailJob(user.id))
    return user

# Job handler
class SendWelcomeEmailJob:
    def __init__(self, user_id: str):
        self.user_id = user_id
    
    def handle(self):
        user = User.find(self.user_id)
        email_service.send_welcome(user.email, user.full_name)
    
    # Retry logic
    def retry_until(self, attempt: int, error: Exception) -> bool:
        if attempt >= 3:
            logger.error('email_failed_permanently', user_id=self.user_id)
            return False  # Move to dead letter queue
        return True  # Retry
```

**Queue decision:**
| Queue | When to use |
|-------|-------------|
| **Redis** (simple) | Fast, low-throughput, loss-tolerant |
| **RabbitMQ** | Complex routing, guaranteed delivery, multiple consumers |
| **SQS** | Managed, auto-scale, AWS stack |
| **Beanstalkd** | Simple, lightweight, single-purpose |

**Dead Letter Queue:**
```python
# Jobs that failed permanently go to DLQ for manual inspection
# DLQ handler:
def handle_failed_jobs():
    failed_jobs = DeadLetterQueue.poll()
    for job in failed_jobs:
        notify_developer(job)
        # Manual: fix bug, re-queue, or discard
```

---

### 20. WebSocket / Real-time

Untuk data real-time, gunakan WebSocket dengan reconnection + heartbeats.

```python
# Server-side WebSocket manager
class WebSocketManager:
    def __init__(self):
        self.connections: dict[str, set[WebSocket]] = defaultdict(set)
    
    async def connect(self, user_id: str, ws: WebSocket):
        await ws.accept()
        self.connections[user_id].add(ws)
    
    async def disconnect(self, user_id: str, ws: WebSocket):
        self.connections[user_id].discard(ws)
        if not self.connections[user_id]:
            del self.connections[user_id]
    
    async def broadcast_to_user(self, user_id: str, event: dict):
        """Send event to all connections of a user."""
        dead_connections = set()
        for ws in self.connections.get(user_id, set()):
            try:
                await ws.send_json(event)
            except WebSocketDisconnect:
                dead_connections.add(ws)
        
        # Cleanup dead connections
        for ws in dead_connections:
            self.connections[user_id].discard(ws)
    
    async def broadcast_to_all(self, event: dict):
        """Broadcast to all connected users."""
        for user_id in list(self.connections.keys()):
            await self.broadcast_to_user(user_id, event)
```

```python
# WebSocket endpoint
@router.websocket('/ws/notifications')
async def notification_ws(websocket: WebSocket, current_user: User = Depends(get_current_user)):
    await ws_manager.connect(current_user.id, websocket)
    try:
        while True:
            # Wait for client messages (or just send server-side events)
            data = await websocket.receive_text()
            
            # Optional: client sends pings, server responds
            if data == 'ping':
                await websocket.send_text('pong')
    except WebSocketDisconnect:
        await ws_manager.disconnect(current_user.id, websocket)
```

---

### 21. Webhook Architecture & Idempotency

**WAJIB** implement idempotency key untuk webhook handlers. **JANGAN** proses event yang sama dua kali.

❌ **Salah:**
```python
# Webhook handler tanpa idempotency
@router.post('/webhooks/stripe')
def handle_stripe_webhook(payload: dict):
    event_type = payload['type']
    
    if event_type == 'payment_intent.succeeded':
        # Kalau Stripe kirim ulang (retry), order dikredit dua kali!
        order = Order.find(payload['data']['object']['id'])
        order.mark_as_paid()
```

✅ **Benar:**
```python
# Webhook handler dengan idempotency key
@router.post('/webhooks/stripe')
def handle_stripe_webhook(payload: dict, request: Request):
    # Stripe sends Idempotency-Key header
    idempotency_key = request.headers.get('Idempotency-Key')
    
    # Check if already processed
    if idempotency_store.exists(idempotency_key):
        return {'status': 'already_processed'}, 200
    
    # Process
    event_type = payload['type']
    event_data = payload['data']['object']
    
    try:
        with db.transaction():
            if event_type == 'payment_intent.succeeded':
                order_service.mark_paid(event_data['id'])
            
            # Mark as processed AFTER successful processing
            idempotency_store.set(idempotency_key, processed=True, ttl=86400)
    except Exception:
        # Don't store idempotency key — allow retry
        raise
    
    return {'status': 'ok'}, 200
```

```go
// Idempotency middleware
func IdempotencyMiddleware(store *redis.Client) gin.HandlerFunc {
    return func(c *gin.Context) {
        key := c.GetHeader("Idempotency-Key")
        if key == "" {
            c.Next()
            return
        }
        
        // Check if already processed
        exists, _ := store.Exists(c.Request.Context(), key).Result()
        if exists > 0 {
            result, _ := store.Get(c.Request.Context(), key).Result()
            c.JSON(200, gin.H{"status": "already_processed", "result": result})
            c.Abort()
            return
        }
        
        c.Next()
        
        // Store response after successful processing
        if c.Writer.Status() < 500 {
            store.Set(c.Request.Context(), key, "processed", 24*time.Hour)
        }
    }
}
```

---

### 22. Message Broker Decision Framework

🔄 **INTERAKTIF** (opsional) — Pilih berdasarkan kebutuhan.

```
Message volume:
[a] Low (< 1000/day)
[b] Medium (< 100K/day)
[c] High (> 100K/day) or streaming

Delivery guarantees:
[a] At-most-once (loss ok)
[b] At-least-once (no loss, duplicates ok)
[c] Exactly-once (no loss, no duplicates)

Routing complexity:
[a] Simple (publish to one queue)
[b] Complex (routing by type/priority/tenant)
[c] Very complex (event sourcing, replay, retention)

Persistence needed?
[ ] Yes — must survive broker restart
[ ] No — loss acceptable
```

| Kombinasi | Recommendation |
|-----------|---------------|
| Low + simple | Redis Pub/Sub or in-process queue |
| Medium + at-least-once | RabbitMQ or SQS |
| High + routing | RabbitMQ (exchange + bindings) |
| Very high + replay | Kafka (partitioned, persistent) |
| Streaming + retention | Kafka |

```python
# RabbitMQ — producer
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()

channel.exchange_declare(exchange='orders', exchange_type='topic')

channel.basic_publish(
    exchange='orders',
    routing_key='order.created',
    body=json.dumps({'order_id': order_id, 'total': 50000}).encode(),
    properties=pika.BasicProperties(
        delivery_mode=2,  # Persistent
        content_type='application/json'
    )
)
```

```python
# Kafka — producer
from kafka import KafkaProducer

producer = KafkaProducer(
    bootstrap_servers=['localhost:9092'],
    value_serializer=lambda v: json.dumps(v).encode(),
    acks='all',  # Wait for all replicas
)

producer.send('orders', {
    'event': 'order.created',
    'order_id': str(order_id),
    'total': 50000,
    'timestamp': datetime.utcnow().isoformat()
})
```

---

# 🔧 DevOps & Resilience

---

### 23. Docker & Containerization

**WAJIB** gunakan multi-stage builds, health checks, dan jangan jalan sebagai root.

❌ **Salah:**
```dockerfile
FROM node:18
COPY . .
RUN npm install
CMD ["npm", "start"]
# 1. No multi-stage = 1GB+ image
# 2. Root user = security risk
# 3. No health check
```

✅ **Benar:**
```dockerfile
# Stage 1: Build
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .

# Stage 2: Production
FROM node:18-alpine
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist

USER appuser

EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

CMD ["node", "dist/server.js"]
```

```dockerfile
# Python example
FROM python:3.12-slim AS builder
COPY requirements.txt .
RUN pip install --user -r requirements.txt

FROM python:3.12-slim
RUN adduser --system --group appuser
COPY --from=builder /root/.local /home/appuser/.local
COPY --chown=appuser:appuser app/ /app
USER appuser
HEALTHCHECK CMD curl --fail http://localhost:8000/health || exit 1
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

### 24. CI/CD Pipeline

**WAJIB** automate: lint → test → build → migrate → deploy. **JANGAN** deploy manual.

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main, staging]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: make lint
      - run: make test
      - run: make test-integration
  
  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - run: make build
      - run: make docker-build
      - run: make docker-push
  
  migrate:
    needs: build
    runs-on: ubuntu-latest
    environment: production
    steps:
      - run: make migrate
    # Rollback: make migrate-rollback (if failed)
  
  deploy:
    needs: migrate
    runs-on: ubuntu-latest
    environment: production
    steps:
      - run: make deploy
      - run: make health-check
    # Rollback: make rollback (previous image)
```

---

### 25. Database Migrations & Versioning

**Prinsip inti:** Version control semua perubahan database, selalu sediakan rollback, jangan edit migration yang sudah di-commit.
**Detail lengkap:** Lihat skill `database-design` — migration patterns, versioning, rollback strategies, data vs schema migration separation.

---

### 26. Environment Configuration

**WAJIB** validasi semua env config di startup. **JANGAN** tunggu error di tengah jalan.

❌ **Salah:**
```python
# Pake langsung — kalau DB_HOST undefined, error di runtime
DB_HOST = os.getenv('DB_HOST')
DB_PORT = os.getenv('DB_PORT')
# app jalan dulu baru crash pas konek DB
```

✅ **Benar:**
```python
# config.py — centralized, validated at startup
from pydantic_settings import BaseSettings

class AppConfig(BaseSettings):
    # App
    app_env: str = 'development'
    app_debug: bool = False
    app_secret: str  # No default = required!
    
    # Database
    db_host: str
    db_port: int = 5432
    db_name: str
    db_user: str
    db_password: str  # From env or vault
    
    # Redis
    redis_url: str = 'redis://localhost:6379/0'
    
    # External APIs
    payment_gateway_key: str  # Required — crash early if missing
    
    class Config:
        env_file = '.env'
        env_file_encoding = 'utf-8'
        # Auto-load from .env, .env.local, .env.production

# Startup validation
try:
    config = AppConfig()
    logger.info('config_loaded', env=config.app_env)
except ValidationError as e:
    logger.critical('config_validation_failed', errors=e.errors())
    sys.exit(1)  # Fail fast — jangan lanjut
```

**Env file hierarchy:**
```
.env              ← Defaults (committed, safe defaults)
.env.local        ← Local overrides (gitignored)
.env.production   ← Production overrides (gitignored, CI injects)
```

---

### 27. Secrets Management

**JANGAN PERNAH** commit secrets ke repository. **WAJIB** gunakan vault atau secrets manager.

❌ **Salah:**
```env
# .env di-commit ke GitHub
DB_PASSWORD=supersecret123
STRIPE_KEY=sk_live_abc123
```

✅ **Benar:**
```python
# Development: .env.local (gitignored)
# Production: Secrets Manager / Vault

# HashiCorp Vault
import hvac

client = hvac.Client(url=VAULT_ADDR, token=VAULT_TOKEN)
secret = client.secrets.kv.read_secret_version(path='production/db')
DB_PASSWORD = secret['data']['data']['password']

# AWS Secrets Manager
import boto3
from botocore.exceptions import ClientError

def get_secret(secret_name: str) -> dict:
    client = boto3.client('secretsmanager')
    response = client.get_secret_value(SecretId=secret_name)
    return json.loads(response['SecretString'])

secrets = get_secret('prod/myapp')
DB_PASSWORD = secrets['db_password']

# Atau minimal: environment variable dari CI/CD (GitHub Secrets, GitLab CI vars)
# JANGAN di .env file!
```

**What NOT to commit:**
```
.env                # Full env file
*.key               # Private keys
*.pem               # Certificates
secrets.*           # Secret files
service-account.json  # GCP/AWS service accounts
```

---

### 28. Graceful Shutdown

**WAJIB** handle SIGTERM/SIGINT dengan graceful: selesaikan request aktif, tutup koneksi, baru exit.

❌ **Salah:**
```python
# Process langsung mati — request terputus, koneksi DB lepas
# Default behavior — kill -2 = immediate death
```

✅ **Benar:**
```python
import signal
import asyncio

class Application:
    def __init__(self):
        self.is_shutting_down = False
        self.active_requests = 0
    
    async def shutdown(self, sig: signal.Signals):
        logger.info('shutdown_initiated', signal=sig.name)
        self.is_shutting_down = True
        
        # 1. Stop accepting new requests
        await server.stop()
        
        # 2. Wait for active requests to finish (max 30s)
        timeout = 30
        while self.active_requests > 0 and timeout > 0:
            await asyncio.sleep(1)
            timeout -= 1
        
        # 3. Close database connections
        await db_pool.close()
        
        # 4. Close Redis
        await redis_client.close()
        
        # 5. Flush logs
        await logger.flush()
        
        logger.info('shutdown_complete')
        sys.exit(0)
    
    def handle_request(self, request):
        self.active_requests += 1
        try:
            # Process request...
            pass
        finally:
            self.active_requests -= 1

# Register signal handlers
app = Application()
signal.signal(signal.SIGTERM, lambda s, f: asyncio.run(app.shutdown(s)))
signal.signal(signal.SIGINT, lambda s, f: asyncio.run(app.shutdown(s)))
```

```go
// Go — graceful shutdown
func main() {
    srv := &http.Server{Addr: ":8080"}
    
    go func() {
        if err := srv.ListenAndServe(); err != nil && err != http.ErrServerClosed {
            log.Fatalf("listen: %s\n", err)
        }
    }()
    
    // Wait for interrupt signal
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <-quit
    
    log.Println("Shutting down server...")
    
    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()
    
    if err := srv.Shutdown(ctx); err != nil {
        log.Fatal("Server forced to shutdown:", err)
    }
    
    log.Println("Server exited")
}
```

---

# 🧩 Advanced Architecture & Optimization

---

### 29. Microservices vs Monolith Decision Framework

🔄 **INTERAKTIF** — Pilih mode:

```
[1] ⚡ Recommended (optimized default)
    → Modular monolith first, extract to microservices only when needed
    → Domain boundaries defined, but deployment is single unit

[2] ✏️ Custom (tanya detail)
```

**Jika pilih Custom:**

```
Current scale:
[a] Startup (< 5 devs, < 10K users)
[b] Growing (5-20 devs, 10K-100K users)
[c] Enterprise (20+ devs, 1M+ users)

Pain points:
[ ] Scaling issues (different services need different scaling)
[ ] Team coordination (too many devs on same codebase)
[ ] Deployment coupling (one change requires full deploy)
[ ] Tech diversity (want different languages for different parts)
```

**Decision guidance:**

| Kondisi | Recommendation |
|---------|---------------|
| Startup + growing | **Modular monolith** — simple, fast to build |
| Growing + team coordination | **Modular monolith with strict boundaries** — prepare for extraction |
| Enterprise + scaling pain | **Microservices** — but start with 2-3 services, not 20 |
| Enterprise + tech diversity | **Microservices** — each service independent |

```python
# Modular Monolith — best for most cases
# Domain boundaries jelas, tapi deploy masih 1 unit
src/
  modules/
    users/      # Own models, services, routes
    orders/     # Own models, services, routes
    payments/   # Own models, services, routes
  shared/       # Shared kernel — minimal!
    exceptions.py
    dto.py
```

```python
# Extract to microservice ONLY when:
# 1. Team size justifies it (10+ devs per service)
# 2. Different scaling needs (payments needs more CPU than users)
# 3. Different release cycles (payments deploys daily, users weekly)
# 4. Technology differentiators (real-time requires Go, rest is Python)
```

---

### 30. API Gateway & Rate Limiting by Tier

**WAJIB** implement rate limiting per tier (Free / Premium / Enterprise). Gunakan API gateway untuk centralized routing.

```python
# Rate limiting strategy per tier
RATE_LIMITS = {
    'free': {
        'requests_per_minute': 10,
        'requests_per_hour': 100,
        'burst': 5,
    },
    'premium': {
        'requests_per_minute': 60,
        'requests_per_hour': 1000,
        'burst': 20,
    },
    'enterprise': {
        'requests_per_minute': 300,
        'requests_per_hour': 10000,
        'burst': 100,
    },
}

class RateLimiter:
    def __init__(self, redis_client):
        self.redis = redis_client
    
    def check_rate_limit(self, api_key: str, tier: str) -> bool:
        limits = RATE_LIMITS[tier]
        key = f'ratelimit:{api_key}:minute'
        
        current = self.redis.incr(key)
        if current == 1:
            self.redis.expire(key, 60)  # Reset after 1 minute
        
        if current > limits['requests_per_minute']:
            raise RateLimitExceededException('Rate limit exceeded')
        
        return True
```

**API Gateway responsibilities:**
```
[Client] → [API Gateway] → [Microservices]
               │
               ├── Rate limiting per tier
               ├── Authentication (JWT validation)
               ├── Request routing
               ├── Response caching
               ├── Request/response transformation
               └── Monitoring / access logs
```

---

### 31. Multi-tenancy Strategies

Pilih strategi isolasi tenant berdasarkan kebutuhan security dan scale.

| Strategy | Isolation | Complexity | Cost | Use Case |
|----------|-----------|------------|------|----------|
| **Shared DB, shared table** | Lowest | Simplest | Lowest | B2C, no sensitive data |
| **Shared DB, tenant_id column** | Medium | Low | Low | SaaS with row-level isolation |
| **Schema per tenant** | High | Medium | Medium | Enterprise SaaS |
| **Database per tenant** | Highest | Complex | Highest | Finance, healthcare |

```python
# Strategy: Shared DB + tenant_id column (most common)
class TenantAwareModel:
    tenant_id: UUID
    
    @classmethod
    def for_current_tenant(cls):
        tenant_id = get_current_tenant_id()
        return cls.where('tenant_id', tenant_id)

# Middleware — inject tenant context
def tenant_middleware(request):
    tenant_id = extract_tenant_from_request(request)  # Subdomain, header, JWT claim
    if not tenant_id:
        raise AuthenticationException('Tenant identifier required')
    
    # Validate tenant access
    if not user_can_access_tenant(current_user, tenant_id):
        raise ForbiddenException('No access to this tenant')
    
    request.tenant_id = tenant_id
    return request
```

---

### 32. Feature Flags (Server-side)

Gunakan server-side feature flags untuk gradual rollout dan A/B testing.

```python
# Feature flag service
class FeatureFlagService:
    def __init__(self, redis_client):
        self.redis = redis_client
        self.flags = self._load_flags()
    
    def is_enabled(self, feature: str, user: User = None) -> bool:
        flag = self.flags.get(feature)
        if not flag:
            return False
        
        if flag['status'] == 'off':
            return False
        
        if flag['status'] == 'on':
            return True
        
        if flag['status'] == 'gradual':
            # %-based rollout
            if flag.get('percentage', 0) >= 100:
                return True
            
            # Consistent hashing based on user_id
            if user:
                hash_value = hash(f'{feature}:{user.id}') % 100
                return hash_value < flag['percentage']
        
        if flag['status'] == 'targeted':
            return user.id in flag.get('user_ids', [])
        
        return False

# JSON config — bisa diubah tanpa deploy
{
    "features": {
        "new_checkout": {
            "status": "gradual",
            "percentage": 25,
            "description": "New checkout flow with improved UX"
        },
        "ai_recommendations": {
            "status": "targeted",
            "user_ids": ["beta-tester-1", "beta-tester-2"],
            "description": "AI-powered product recommendations"
        },
        "dark_mode": {
            "status": "on",
            "description": "Dark mode toggle"
        },
        "legacy_api": {
            "status": "off",
            "description": "Deprecated v1 endpoints"
        }
    }
}
```

---

### 33. Search Strategy

Untuk full-text search, jangan andalkan `LIKE %keyword%` di PostgreSQL untuk production scale. Gunakan dedicated search engine.

❌ **Salah:**
```sql
-- Full table scan — slow, no relevance ranking
SELECT * FROM products WHERE name LIKE '%smartphone%' OR description LIKE '%smartphone%';
```

✅ **Benar:**
```python
# Elasticsearch / MeiliSearch
from elasticsearch import Elasticsearch

es = Elasticsearch(ELASTIC_URL)

def search_products(query: str, page: int = 1, per_page: int = 20):
    body = {
        'query': {
            'multi_match': {
                'query': query,
                'fields': ['name^3', 'description^2', 'category'],
                'type': 'best_fields',
            }
        },
        'from': (page - 1) * per_page,
        'size': per_page,
        'sort': ['_score', {'created_at': 'desc'}],
    }
    
    result = es.search(index='products', body=body)
    
    return {
        'data': [hit['_source'] for hit in result['hits']['hits']],
        'meta': {
            'total': result['hits']['total']['value'],
            'page': page,
            'per_page': per_page,
        }
    }

# Atau pakai PostgreSQL full-text search (built-in, no extra service)
SELECT * FROM products
WHERE to_tsvector('english', name || ' ' || description)
      @@ to_tsquery('english', 'smartphone');
```

---

### 34. Backup & Disaster Recovery

**WAJIB** punya backup strategy. **SELALU** test restore secara berkala.

**Backup types:**
| Type | Frequency | Retention | Use |
|------|-----------|-----------|-----|
| Full DB dump | Daily | 30 days | Restore entire DB |
| WAL / binlog | Continuous | 7 days | Point-in-time recovery |
| File storage | Daily | 30 days | Restore uploaded files |
| Configuration | Per change | Indefinite | Infra-as-code |

```bash
# PostgreSQL backup
pg_dump -Fc mydb > /backups/mydb_$(date +%Y%m%d).dump

# Restore
pg_restore -d mydb /backups/mydb_20241201.dump

# Point-in-time recovery (with WAL archiving)
# 1. Restore base backup
# 2. Apply WAL files up to target time

# Test restore — monthly!
# 0 0 1 * * /scripts/test_restore.sh
```

**DR Plan checklist:**
- RTO (Recovery Time Objective): Maksimal waktu downtime (e.g., 1 hour)
- RPO (Recovery Point Objective): Maksimal data loss (e.g., 5 minutes)
- Backup stored in different region / cloud provider
- Test restore dijadwalkan (bukan cuma ada backup filenya)

---

### 35. Resource Profiling & Optimization

Pahami di mana bottleneck aplikasi — jangan asal optimize.

```python
# Profiling checklist:
# 1. CPU-bound → scaling (more instances) atau optimize algorithm
# 2. I/O-bound → async, connection pool, caching
# 3. Memory-bound → streaming, pagination, GC tuning
# 4. Network-bound → CDN, compression, batch requests

# Example: profiling slow endpoint
@router.get('/slow-endpoint')
def slow_endpoint():
    profiler = start_profiling()
    
    # CPU-heavy
    result = expensive_computation()
    
    # I/O-heavy
    users = user_repository.find_all()
    
    profiler.stop()
    logger.info('endpoint_profile', 
        cpu_time=profiler.cpu_time,
        io_time=profiler.io_time,
        db_queries=profiler.query_count,
        total_time=profiler.total_time
    )
    
    return result
```

**Common optimizations:**
| Problem | Solution |
|---------|----------|
| N+1 queries | Eager loading, batch loading |
| Large dataset in memory | Pagination, streaming, cursor |
| Slow queries | Index, EXPLAIN, query rewrite |
| Too many DB connections | Connection pooling, max connections tuning |
| CPU-intensive sync tasks | Background queue |
| Repeated expensive computation | Caching |
| Large response payloads | Field selection, compression, pagination |

---

### 36. API Performance Monitoring (APM)

**WAJIB** monitoring: response time, error rate, throughput, slow queries.

```python
# Metrics middleware
class MetricsMiddleware:
    def __init__(self, statsd_client):
        self.statsd = statsd_client
    
    def __call__(self, request):
        start = time.time()
        
        try:
            response = self.get_response(request)
            duration = time.time() - start
            
            # Track metrics
            self.statsd.timing('api.response_time', duration * 1000,
                tags=[f'endpoint:{request.path}',
                      f'method:{request.method}',
                      f'status:{response.status_code}'])
            
            if response.status_code >= 500:
                self.statsd.increment('api.errors',
                    tags=[f'endpoint:{request.path}'])
            
            return response
        except Exception:
            self.statsd.increment('api.exceptions')
            raise
```

**What to monitor:**
```
Response time:
  p50, p95, p99 latency per endpoint
  Alert if p95 > 500ms for critical endpoints

Error rate:
  Total errors per minute
  Error rate per endpoint
  Alert if > 1% error rate

Throughput:
  Requests per second (RPS)
  Peak vs average

Database:
  Slow queries (> 100ms)
  Connection pool utilization
  Cache hit ratio

External APIs:
  Latency to payment gateway / email service
  Error rate per external service
```

---

### 37. Code Organization & Modularity (DDD-style)

**Prinsip inti:** Modular / domain-driven structure, bukan layer-based. Setiap domain punya controllers, services, repositories, models sendiri.
**Detail lengkap:** Lihat skill `code-standards` — modular architecture patterns, DDD principles, shared kernel, module boundaries.

---

### 38. Event Sourcing & CQRS

Untuk sistem yang butuh audit trail lengkap atau temporal query, gunakan Event Sourcing + CQRS.

```python
# Event Sourcing — store events, not state
class EventStore:
    def __init__(self, db):
        self.db = db
    
    def append(self, stream_id: str, event: DomainEvent):
        """Append event to stream — immutable log."""
        self.db.execute("""
            INSERT INTO event_store (stream_id, event_type, data, version, created_at)
            VALUES (%s, %s, %s, %s, %s)
        """, [stream_id, event.type, json.dumps(event.data), event.version, event.created_at])
    
    def get_stream(self, stream_id: str) -> list[DomainEvent]:
        """Reconstruct entity by replaying events."""
        rows = self.db.query(
            "SELECT * FROM event_store WHERE stream_id = %s ORDER BY version ASC",
            [stream_id]
        )
        return [DomainEvent.from_row(row) for row in rows]

# Example: Order aggregate
class OrderAggregate:
    def __init__(self):
        self.id = None
        self.status = 'pending'
        self.items = []
        self.version = 0
    
    def apply(self, event: DomainEvent):
        """Rehydrate state from event."""
        if event.type == 'order.created':
            self.id = event.data['order_id']
            self.items = event.data['items']
        elif event.type == 'order.shipped':
            self.status = 'shipped'
        # ... setiap transisi status adalah event
    
    @classmethod
    def load(cls, event_store: EventStore, order_id: str):
        """Load aggregate by replaying all events."""
        aggregate = cls()
        for event in event_store.get_stream(f'order:{order_id}'):
            aggregate.apply(event)
            aggregate.version = event.version
        return aggregate

# CQRS — separate read/write
class OrderWriteModel:
    """Handles commands — writes go to event store."""
    def ship(self, order_id: str):
        event = DomainEvent(
            stream_id=f'order:{order_id}',
            type='order.shipped',
            data={'shipped_at': datetime.utcnow().isoformat()},
            version=self.get_next_version(order_id)
        )
        event_store.append(event.stream_id, event)

class OrderReadModel:
    """Handles queries — optimized for reading, denormalized."""
    def get_shipped_orders_today(self):
        return db.query("""
            SELECT * FROM order_read_model
            WHERE status = 'shipped' AND shipped_at >= CURRENT_DATE
        """)
```

---

### 39. Database Replication & Read/Write Splitting

Untuk high-traffic aplikasi, pisahkan read dan write database.

```python
# Database router — otomatis arahkan query
class DatabaseRouter:
    def __init__(self):
        self.write_pool = ConnectionPool(WRITE_DB_URL, minconn=5)
        self.read_pools = [
            ConnectionPool(READ_REPLICA_1, minconn=10),
            ConnectionPool(READ_REPLICA_2, minconn=10),
        ]
        self.read_index = 0  # Round-robin
    
    def get_writer(self):
        return self.write_pool.getconn()
    
    def get_reader(self):
        pool = self.read_pools[self.read_index % len(self.read_pools)]
        self.read_index += 1
        return pool.getconn()

class UserRepository:
    def find_by_id(self, user_id: str) -> User:
        conn = db_router.get_reader()  # Read — ke replica
        return conn.query(...)
    
    def create(self, user: User) -> User:
        conn = db_router.get_writer()  # Write — ke master
        return conn.execute("INSERT INTO users ...")

# Replication lag handling:
# 1. After write, read from master for X seconds
# 2. Use read-after-write consistency for critical data
def get_user_after_update(user_id: str):
    if is_recently_updated(user_id):
        conn = db_router.get_writer()  # Read from master
    else:
        conn = db_router.get_reader()  # Read from replica
    return conn.query(...)
```

---

# 🚨 Self-Enforcement — WAJIB di Setiap Kode Backend

**Sebelum menulis endpoint / service / migration / query baru, WAJIB cek:**

1. **Layered architecture?** — JANGAN taruh business logic di controller. Service layer for logic, repository for data.
2. **Input validation?** — WAJIB validated di server-side. JANGAN cuma frontend.
3. **SQL injection prevention?** — Prepared statements atau ORM. JANGAN string concatenation.
4. **Error handling?** — JANGAN try-catch di tiap controller. Pake global handler + custom exceptions.
5. **Migration rollback?** — SEDIAKAN `down()` untuk setiap migration. JANGAN migration tanpa rollback.
6. **Connection leak?** — WAJIB connection pooling. JANGAN buka/tutup koneksi tiap request.
7. **N+1 queries?** — Cek eager loading. JANGAN loop query di dalam loop.
8. **Caching strategy?** — JANGAN panggil DB untuk data yang jarang berubah.
9. **Secrets exposed?** — JANGAN hardcode secrets. JANGAN commit .env.
10. **Graceful shutdown?** — Handle SIGTERM. JANGAN mati mendadak.

**Kalau ragu dengan arsitektur atau approach → tanya user dulu. JANGAN asumsi.**

---

# ✅ Final Checklist

Sebelum menyelesaikan task backend, pastikan:

```
Architecture & Design
□ Layered architecture (Controller → Service → Repository → DTO)?
□ RESTful URI conventions diikuti?
□ Input/output DTO terpisah dari entity?
□ Pagination & filtering di semua list endpoint?
□ API documentation (OpenAPI) updated?
□ API versioning strategy ditentukan?

Security
□ Input validation di server-side?
□ Prepared statements / ORM untuk semua query?
□ CORS origin specific (bukan wildcard)?
□ Rate limiting implemented per tier?
□ Authentication sesuai konteks (JWT/session/OAuth)?
□ Authorization check di setiap endpoint?
□ Secrets di vault / env, bukan hardcode?
□ Security headers terpasang?

Data & Database
□ UUID sebagai primary key?
□ Soft delete (`deleted_at`) di semua tabel?
□ Audit fields (`created_at`, `updated_at`)?
□ Migration punya rollback?
□ N+1 queries sudah di-cek?
□ Connection pooling configured?
□ Transactions untuk multi-record operations?
□ Caching strategy sesuai data volatility?
□ Backup strategy + tested restore?

Testing & Error Handling
□ Unit test untuk business logic?
□ Integration test untuk service + DB?
□ Custom exception hierarchy?
□ Global error handler?
□ Structured logging (JSON)?
□ Correlation ID untuk tracing?

DevOps & Resilience
□ Docker multi-stage build?
□ CI/CD pipeline (test → build → deploy)?
□ Graceful shutdown handler?
□ Health check endpoint?
□ Resource profiling done?
□ Monitoring metrics (response time, error rate)?
□ Database replication strategy (jika perlu)?

Architecture Decisions
□ Modular / DDD-style folder structure?
□ Microservices vs monolith — keputusan sudah tepat?
□ Message broker — sudah pilih yang sesuai?
□ Event sourcing — sudah pertimbangkan tradeoff?
□ Feature flags untuk gradual rollout?
```

