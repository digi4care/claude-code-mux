# Multi-Tenant Authentication System - Epic Breakdown

## Overview

Building a multi-tenant authentication system with per-user API keys, usage tracking, and admin UI.

**Current**: Single-user system (any API key works)
**Proposed**: Multi-user system with per-user API keys and authentication

---

## Epic 1: Database Foundation

### Must-Have (MVP)
- [ ] SQLx + SQLite setup
- [ ] Database schema (users, api_keys, usage_stats, sessions)
- [ ] SQLx migrations system (up + down.sql files)
- [ ] Connection pooling with proper SQLite settings
- [ ] PRAGMA optimization (WAL mode, cache size, synchronous)
- [ ] Indexes for query performance

### Implementation Details

**SQLite Connection String**:
```rust
// Enable WAL mode for concurrent reads
const DATABASE_URL: &str = "sqlite:claude-code-mux.db?mode=rwc";
```

**PRAGMA Settings (First Run)**:
```sql
PRAGMA journal_mode = WAL;          -- Enable Write-Ahead Logging
PRAGMA synchronous = NORMAL;        -- Safe but faster (vs FULL)
PRAGMA cache_size = -64000;         -- 64MB cache (negative = KB)
PRAGMA temp_store = MEMORY;         -- Temp tables in RAM
PRAGMA foreign_keys = ON;           -- Enable FK constraints
PRAGMA busy_timeout = 5000;         -- 5 second lock timeout
```

**Connection Pool Settings**:
```rust
SqlitePoolOptions::new()
    .max_connections(5)           // SQLite: max 1 writer, 5+ readers fine
    .min_connections(1)           // Keep 1 warm
    .acquire_timeout(Duration::from_secs(30))
    .idle_timeout(Duration::from_secs(600))   // 10 min
    .max_lifetime(Duration::from_secs(1800))  // 30 min
```

**Migration Structure**:
```
migrations/
├── 001_initial_schema.up.sql        -- Create tables + indexes
├── 001_initial_schema.down.sql      -- Drop tables + indexes
├── 002_add_sessions_table.up.sql
├── 002_add_sessions_table.down.sql
└── ...
```

### Nice-to-Have (V2)
- [ ] Automatic backups (daily/weekly)
- [ ] Database cleanup jobs (archive old stats > 6 months)
- [ ] Admin UI for database management
- [ ] Export/Import functionality (CSV/JSON)

---

## Epic 2: Authentication System

### Must-Have (MVP)
- [ ] Password hashing (bcrypt)
- [ ] User CRUD (Create, Read, Update, Delete)
- [ ] Session management (JWT tokens)
- [ ] Login/logout endpoints
- [ ] Admin user (seeded on first run)

### Nice-to-Have (V2)
- [ ] Password reset flow
- [ ] Two-factor authentication (2FA)
- [ ] Session expiration settings
- [ ] OAuth login (Google, GitHub)
- [ ] Audit logging (login attempts, password changes)

---

## Epic 3: API Key Management

### Must-Have (MVP)
- [ ] API key generation (`sk-user-<random>`)
- [ ] API key storage (hashed + salted)
- [ ] API key verification middleware
- [ ] Per-user API key CRUD
- [ ] API key deactivation

### Nice-to-Have (V2)
- [ ] API key expiration dates
- [ ] API key usage limits (rate limiting)
- [ ] API key scopes/permissions (read-only, admin, etc.)
- [ ] API key rotation
- [ ] API key usage analytics

---

## Epic 4: Usage Tracking

### Must-Have (MVP)
- [ ] Log requests per user (model, provider, tokens)
- [ ] Store usage stats in database
- [ ] Basic stats API (total tokens, total cost)
- [ ] Per-user stats overview

### Nice-to-Have (V2)
- [ ] Real-time stats dashboard
- [ ] Cost graphs (daily/weekly/monthly)
- [ ] Per-model breakdown
- [ ] Per-provider breakdown
- [ ] Export stats to CSV/JSON
- [ ] Alerts/budget limits

---

## Epic 5: Admin UI (Sailfish Templates)

### Must-Have (MVP)
- [ ] Login page (`/login`)
- [ ] Dashboard (`/dashboard`)
- [ ] User management (`/admin/users`)
- [ ] API key management (`/admin/api-keys`)
- [ ] Stats overview (`/admin/stats`)
- [ ] Auth middleware (protect admin routes)

### Nice-to-Have (V2)
- [ ] User registration page
- [ ] User profile page
- [ ] Password change page
- [ ] Dark mode support
- [ ] Responsive design (mobile)
- [ ] Activity logs
- [ ] Settings page

---

## Epic 6: API Changes

### Must-Have (MVP)
- [ ] API key verification in request header
- [ ] Return 401 for invalid API keys
- [ ] Per-user rate limiting (basic)
- [ ] Usage tracking in request flow
- [ ] Maintain backward compatibility (fallback: "any-string" API key)

### Nice-to-Have (V2)
- [ ] Rate limiting per tier (free/paid/admin)
- [ ] Usage quota enforcement
- [ ] API response headers (X-RateLimit-Remaining, etc.)
- [ ] Webhooks for usage events
- [ ] API versioning

---

## Epic 7: Security Hardening

### Must-Have (MVP)
- [ ] Password requirements (min length, complexity)
- [ ] SQL injection prevention (parameterized queries)
- [ ] XSS protection in templates
- [ ] CSRF tokens for forms
- [ ] Secure headers (CSP, X-Frame-Options, etc.)

### Nice-to-Have (V2)
- [ ] Brute force protection (rate limit login attempts)
- [ ] IP whitelist/blacklist
- [ ] Security audit logging
- [ ] Penetration testing
- [ ] Security headers scanner integration

---

## Epic 8: Documentation

### Must-Have (MVP)
- [ ] API documentation (API key authentication)
- [ ] User guide (how to create API keys)
- [ ] Admin guide (user management)
- [ ] Migration guide (existing users)

### Nice-to-Have (V2)
- [ ] Integration examples (cURL, Python, JavaScript)
- [ ] Video tutorials
- [ ] FAQ section
- [ ] Architecture diagrams
- [ ] Security best practices guide

---

## MVP Timeline (Must-Haves Only)

**Epic Order for MVP**:
1. **Epic 1** - Database Foundation (3-5 days)
2. **Epic 2** - Authentication System (5-7 days)
3. **Epic 3** - API Key Management (3-5 days)
4. **Epic 4** - Usage Tracking (2-3 days)
5. **Epic 5** - Admin UI (Sailfish) (7-10 days)
6. **Epic 6** - API Changes (3-5 days)
7. **Epic 7** - Security Hardening (3-5 days)
8. **Epic 8** - Documentation (2-3 days)

**Total MVP**: ~4-6 weeks full-time development

---

## Nice-to-Have Priority

### High Priority (after MVP)
- Password reset flow
- API key usage limits
- Real-time stats dashboard

### Medium Priority
- API key expiration
- User registration page
- Export stats

### Low Priority
- 2FA
- OAuth login
- Mobile responsive

---

## Database Schema (MVP) - Reviewed by Database Architect

```sql
-- users table (improved)
CREATE TABLE users (
    id TEXT PRIMARY KEY,              -- UUID v4
    username TEXT UNIQUE NOT NULL,
    email TEXT UNIQUE NOT NULL,
    password_hash TEXT NOT NULL,      -- bcrypt (cost 12)
    created_at INTEGER NOT NULL,      -- Unix timestamp (milliseconds)
    updated_at INTEGER NOT NULL,      -- For audit trails
    is_admin BOOLEAN NOT NULL DEFAULT 0,
    is_active BOOLEAN NOT NULL DEFAULT 1  -- User disable without deletion
);

-- api_keys table (improved)
CREATE TABLE api_keys (
    id TEXT PRIMARY KEY,              -- UUID v4
    user_id TEXT NOT NULL,
    name TEXT NOT NULL,               -- Display name: "Production key", "Dev key"
    key_hash TEXT NOT NULL UNIQUE,    -- SHA-256 hash, UNIQUE for O(1) lookup
    key_prefix TEXT NOT NULL,         -- "sk_user_abc1..." (first 12 chars) - UI display
    created_at INTEGER NOT NULL,
    last_used_at INTEGER,
    expires_at INTEGER,               -- NULL = no expiration
    is_active BOOLEAN NOT NULL DEFAULT 1,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- usage_stats table (improved)
CREATE TABLE usage_stats (
    id TEXT PRIMARY KEY,              -- UUID v4
    user_id TEXT NOT NULL,
    api_key_id TEXT,                  -- NULL for deleted keys (retain stats)
    model TEXT NOT NULL,
    provider TEXT NOT NULL,
    tokens_input INTEGER NOT NULL,
    tokens_output INTEGER NOT NULL,
    cost_usd REAL NOT NULL DEFAULT 0,
    timestamp INTEGER NOT NULL,       -- Unix timestamp (milliseconds)
    request_id TEXT,                  -- Correlate multiple events
    status TEXT NOT NULL DEFAULT 'success',  -- 'success', 'error', 'timeout'
    error_type TEXT,                  -- For failure analysis
    FOREIGN KEY (user_id) REFERENCES users(id),
    FOREIGN KEY (api_key_id) REFERENCES api_keys(id) ON DELETE SET NULL
);

-- sessions table (JWT revocation/blacklist)
CREATE TABLE sessions (
    id TEXT PRIMARY KEY,              -- Session ID (jti claim)
    user_id TEXT NOT NULL,
    created_at INTEGER NOT NULL,
    expires_at INTEGER NOT NULL,
    revoked_at INTEGER,               -- NULL = active
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- Indexes (CRITICAL for performance)
CREATE INDEX idx_users_username ON users(username);
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_is_active ON users(is_active);

CREATE INDEX idx_api_keys_user_id ON api_keys(user_id);
CREATE INDEX idx_api_keys_key_hash ON api_keys(key_hash);  -- PRIMARY lookup path!
CREATE INDEX idx_api_keys_is_active ON api_keys(is_active);

CREATE INDEX idx_usage_stats_user_id_timestamp ON usage_stats(user_id, timestamp DESC);
CREATE INDEX idx_usage_stats_api_key_id ON usage_stats(api_key_id);
CREATE INDEX idx_usage_stats_timestamp ON usage_stats(timestamp DESC);  -- Time-series queries
CREATE INDEX idx_usage_stats_model ON usage_stats(model);  -- Per-model aggregations

CREATE INDEX idx_sessions_user_id ON sessions(user_id);
CREATE INDEX idx_sessions_expires_at ON sessions(expires_at);  -- Cleanup job
```

### Key Improvements from Original Schema:

1. **User table**: Added `updated_at`, `is_active` for soft deletes
2. **API Keys table**: Added `name` (display), `expires_at`, UNIQUE constraint on `key_hash`
3. **Usage Stats table**: Added `request_id`, `status`, `error_type` for better analytics
4. **Sessions table**: NEW - for JWT token revocation support
5. **Indexes**: COMPOUND indexes on `(user_id, timestamp)` for query performance
6. **Foreign Keys**: Added `ON DELETE CASCADE` and `ON DELETE SET NULL` for data integrity

---

## Proposed Architecture

```
Current (Single-user):
Claude Code → ccm (API key: "any-string") → Providers

Proposed (Multi-user):
User 1 → ccm (API key: sk-user1-abc123) → Providers
User 2 → ccm (API key: sk-user2-xyz789) → Providers
Admin → ccm (Admin UI → user management)
```

---

## Technology Choices

### Database: SQLx + SQLite
- **Why SQLx**: Compile-time checked queries, battle-tested, async support
- **Why SQLite**: Lightweight, embedded, perfect for single-server deployments
- **Alternative**: Limbo (lighter, but less mature)

### Template Engine: Sailfish
- **Why Sailfish**: Compile-time rendering, type-safe, zero runtime overhead
- **Use case**: Login page, dashboard, user management UI

### Authentication: JWT
- **Why JWT**: Stateless, scalable, industry standard
- **Alternative**: Session tokens (simpler, but requires server-side storage)
- **Storage**: JWT tokens in sessions table for revocation support

---

## Security Implementation Guidelines

### Password Hashing
- **Algorithm**: bcrypt (cost 12)
- **Library**: `bcrypt` crate (v0.16+)
- **Rationale**: Slow hash for password bruteforce protection
- **Example**:
```rust
bcrypt::hash(password, 12)?  // Hash
bcrypt::verify(password, hash)?  // Verify
```

### API Key Storage
- **Algorithm**: SHA-256 (NOT bcrypt!)
- **Rationale**: API keys are random (no bruteforce risk), faster verification
- **Library**: `sha2` crate + `hex` encoding
- **Example**:
```rust
let hash = hex::encode(Sha256::digest(key.as_bytes()));
let key = format!("sk_user_{}", hex::encode(random_32_bytes));
```

### SQL Injection Prevention
- **Primary Defense**: SQLx `query!` and `query_as!` macros (compile-time checked)
- **Never**: String concatenation in queries
- **Always**: Bind parameters with `?` placeholders
- **Example**:
```rust
// ✅ CORRECT
sqlx::query_as!(User, "SELECT * FROM users WHERE email = ?", email)

// ❌ WRONG - SQL injection risk
sqlx::query(&format!("SELECT * FROM users WHERE email = '{}'", email))
```

### Additional Security Measures
- **Rate Limiting**: In-memory DashMap per API key (60 req/min window)
- **HTTPS Only**: Never transmit API keys or passwords over HTTP
- **Password Requirements**: Min 12 chars, complexity optional but recommended
- **Session Management**: JWT with jti claim stored in sessions table for revocation

---

## Scalability: SQLite vs PostgreSQL Migration Triggers

### When to Stay on SQLite

| Metric | SQLite Limit | Your Need (MVP) | Status |
|--------|--------------|-----------------|--------|
| Database size | 281 TB | < 1 GB (year 1) | ✅ |
| Concurrent readers | Unlimited | 10-100 | ✅ |
| Concurrent writers | 1 (WAL serialized) | < 10/s | ✅ |
| Table rows | 2^64 (unlimited) | 1M+ stats | ✅ |

**SQLite is sufficient for 12-18 months** with 1000+ users.

### When to Migrate to PostgreSQL

**Trigger Criteria**:
1. **Concurrent writes** > 10/second (sustained)
2. **Database size** > 10 GB
3. **Multi-server deployment** needed (horizontal scaling)
4. **Need advanced features**: PARTITION, replication, complex joins

**Migration Indicators**:
| Metric | Stay on SQLite | Migrate to PostgreSQL |
|--------|----------------|----------------------|
| Users | < 1000 | > 1000 |
| API requests/day | < 50K | > 50K |
| Concurrent users | < 50 | > 50 |
| Servers | 1 | > 1 |

### Migration Strategy (When Needed)

**Step 1: SQLx Abstraction**
```rust
// Use generic Sqlite vs Pg types
pub async fn get_user(pool: &Pool) -> Result<User>
where Pool: Database + FromRow
```

**Step 2: Export SQLite → PostgreSQL**
```bash
pgloader sqlite:claude-code-mux.db postgresql://user@host/db
```

**Step 3: Switch DATABASE_URL**
```rust
DATABASE_URL=postgresql://user@host/claude-code-mux
```

**Timeline**: 1-2 weeks migration when needed (not urgent for MVP)

---

## Performance Optimization Guidelines

### Usage Stats Query Optimization

**Problem**: Full table scans on usage_stats (heavy write table)

**Solution 1: Compound Indexes** (already in schema)
```sql
CREATE INDEX idx_usage_stats_user_id_timestamp ON usage_stats(user_id, timestamp DESC);
```

**Solution 2: Always Bound Time Ranges**
```rust
// ✅ FAST - uses index
WHERE user_id = ? AND timestamp >= ? AND timestamp < ?

// ❌ SLOW - full table scan
WHERE user_id = ?  // No time bound!
```

**Solution 3: Aggregation Queries**
```sql
-- Efficient daily stats with index support
SELECT
    (timestamp / 86400000) * 86400000 as day_bucket,
    SUM(tokens_input) as total_input,
    SUM(tokens_output) as total_output,
    SUM(cost_usd) as total_cost,
    COUNT(*) as request_count
FROM usage_stats
WHERE user_id = ? AND timestamp >= ? AND timestamp < ?
GROUP BY day_bucket
ORDER BY day_bucket DESC;
```

### Archival Strategy for Old Stats

**Problem**: usage_stats grows infinitely (every API request)

**Solution**: Monthly archival job
```rust
// Archive stats older than 6 months
pub async fn archive_old_stats(pool: &Pool, months: i64) -> Result<()> {
    let cutoff = now() - (months * 30 * 24 * 60 * 60 * 1000);

    // 1. Copy to archive database
    sqlx::query("ATTACH DATABASE 'archive.db' AS archive;
                INSERT INTO archive.usage_stats SELECT * FROM usage_stats WHERE timestamp < ?;
                DETACH DATABASE archive;")
        .bind(cutoff)
        .execute(pool)
        .await?;

    // 2. Delete from main
    sqlx::query("DELETE FROM usage_stats WHERE timestamp < ?")
        .bind(cutoff)
        .execute(pool)
        .await?;

    // 3. Reclaim space
    sqlx::query("VACUUM").execute(pool).await?;

    Ok(())
}
```

**Schedule**: Run monthly via tokio::spawn + cron

### Caching Strategy

**What to Cache**:
- User stats aggregations (5 min TTL)
- API key verification results (1 min TTL)
- User permissions (10 min TTL)

**What NOT to Cache**:
- Individual usage records (too volatile)
- Real-time stats (defeats purpose)

**Implementation**:
```rust
use lru::LruCache;

pub struct StatsCache {
    cache: Mutex<LruCache<String, (UserStats, i64)>>, // (user_id, (stats, expires_at))
}

// TTL: 5 minutes
// Invalidation: After new usage_stats INSERT
```

---

## Next Steps

**Decision Required**:
1. **Full MVP** - Build all must-haves (4-6 weeks)
2. **Phased Approach** - Epic 1-3, deploy, Epic 4-6, etc.
3. **Simplified MVP** - Subset of must-haves

**Choose direction** before implementation begins.
