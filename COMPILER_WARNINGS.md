# Compiler Warnings Report

Generated: 2026-01-09
Version: 0.6.5

## Summary
- **Total Warnings**: 60

---

## 1. Unused Imports (9)

### `src/auth/oauth.rs`
- Line 6: `DateTime`

### `src/auth/admin_auth.rs`
- Line 5: `Algorithm`, `DecodingKey`, `EncodingKey`, `Header`, `Validation`, `decode`, `encode`

### `src/auth/mod.rs`
- Line 5: `AuthorizationUrl`, `PKCEVerifier`
- Line 6: `OAuthToken`
- Line 7: `AuthError`

### `src/server/oauth_handlers.rs`
- Line 11: `TokenStore`

### `src/server/mod.rs`
- Line 6: `RouteDecision`, `RouteType`, `SystemPrompt`
- Line 10: `Query`
- Line 20: `Serialize`

---

## 2. Unused Variables (6)

### Mutable variables that don't need mut
- `src/server/mod.rs:163` - variable doesn't need to be mutable
- `src/server/mod.rs:211` - variable doesn't need to be mutable

### Unused variables
- `src/auth/admin_auth.rs:188` - `client_ip`
- `src/auth/admin_auth.rs:199` - `password_hash`
- `src/auth/admin_auth.rs:522` - `i`
- `src/server/mod.rs:349` - `auth_service`
- `src/server/mod.rs:559` - `status`
- `src/server/mod.rs:654` - `status`

---

## 3. Dead Code - Methods (5)

### `src/auth/oauth.rs`
- `get_valid_token` (line 522)
- `create_api_key` (line 535)

### `src/auth/admin_auth.rs`
- `is_auth_enabled`
- `get_users`
- `get_user_sessions`
- `save_password_hash`
- `cleanup_expired_sessions` (all line 130)

### `src/providers/mod.rs`
- `get_auth_credential` (line 112)

### `src/providers/openai.rs`
- `transform_responses_response` (line 805)

### `src/providers/anthropic_compatible.rs`
- `with_headers` (line 49)
- `anthropic` (line 49)
- `openrouter` (line 49)

### `src/providers/streaming.rs`
- `to_sse_string` (line 16)

---

## 4. Dead Code - Structs (6)

### `src/auth/admin_auth.rs`
- `TokenClaims` (line 41)
- `TokenInfoResponse` (line 95)
- `SessionInfo` (line 101)

### `src/providers/openai.rs`
- `OpenAIResponsesResponse` (line 163)
- `ResponsesOutput` (line 171)
- `ResponsesContentBlock` (line 179)
- `ResponsesUsage` (line 187)

---

## 5. Dead Code - Enum Variants (3)

### `src/auth/admin_auth.rs`
- `InvalidToken` (line 23)
- `IoError` (line 23)

### `src/providers/openai.rs`
- `Text` (line 55)

---

## 6. Dead Code - Constants (13)

### `src/cli/mod.rs`
- `DEFAULT_PORT` (line 10)
- `DEFAULT_HOST` (line 11)
- `DEFAULT_LOG_LEVEL` (line 12)
- `DEFAULT_AUTH_ENABLED` (line 13)
- `DEFAULT_SESSION_DURATION_SECS` (line 14)
- `DEFAULT_TOKEN_ISSUER` (line 15)
- `DEFAULT_MAX_SESSIONS` (line 16)
- `DEFAULT_MAX_LOGIN_ATTEMPTS` (line 17)
- `DEFAULT_RATE_LIMIT_WINDOW_SECS` (line 18)
- `DEFAULT_RATE_LIMIT_LOCKOUT_SECS` (line 19)
- `DEFAULT_API_TIMEOUT_MS` (line 20)
- `DEFAULT_CONNECT_TIMEOUT_MS` (line 21)

### `src/server/mod.rs`
- `DEFAULT_PORT` (line 29)
- `OAUTH_CALLBACK_PORT` (line 30)
- `DEFAULT_UNKNOWN_MODEL` (line 31)

---

## 7. Dead Code - Functions (2)

### `src/server/mod.rs`
- `find_model_config` (line 82)
- `get_sorted_mappings` (line 90)
- `execute_provider_request_with_fallback` (line 106)

---

## 8. Dead Code - Associated Functions (1)

### `src/providers/streaming.rs`
- `new` (line 75)

---

## 9. Unused Fields (11)

### Struct fields never read

#### `src/auth/admin_auth.rs`
- `Session.created_at` (line 55)
- `Session.is_superadmin` (line 57)
- `Session.last_activity` (line 58)

#### `src/providers/openai.rs`
- `OpenAIUsage.total_tokens` (line 158)

#### `src/providers/gemini.rs`
- `GeminiProvider.name` (line 14)
- `GeminiUsageMetadata.total_token_count` (line 868)
- `CodeAssistResponse.trace_id` (line 902)
- `GeminiError.code` (line 914)
- `GeminiError.message` (line 915)
- `GeminiError.status` (line 916)

#### `src/providers/streaming.rs`
- `SseEvent.event` (line 10)
- `SseEvent.data` (line 11)

#### `src/server/oauth_handlers.rs`
- `OAuthCallbackQuery.state` (line 293)

#### `src/server/openai_compat.rs`
- `OpenAIRequest.tools` (line 21)
- `OpenAIRequest.tool_choice` (line 23)
- `OpenAIMessage.name` (line 32)
