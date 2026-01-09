# Provider Reference

Complete reference for all supported AI providers - API documentation, how they work, and what to check when something breaks.

---

## Quick Reference

| Provider | Type | Format | Docs Link |
|----------|------|--------|-----------|
| **Anthropic** | Anthropic-compatible | Native | [docs.anthropic.com](https://docs.anthropic.com/en/api/messages) |
| **z.ai** | Anthropic-compatible | Native | [api.z.ai](https://api.z.ai/docs) |
| **ZenMux** | Anthropic-compatible | Native | [zenmux.ai](https://zenmux.ai/docs) |
| **Minimax** | Anthropic-compatible | Native | [minimax.io](https://minimax.io/document/anthropic) |
| **Kimi For Coding** | Anthropic-compatible | Native | [kimi.com](https://api.kimi.com) |
| **OpenAI** | OpenAI-compatible | Conversion | [platform.openai.com](https://platform.openai.com/docs/api-reference) |
| **OpenRouter** | OpenAI-compatible | Conversion | [openrouter.ai](https://openrouter.ai/docs) |
| **Google Gemini** | Custom | Conversion | [ai.google.dev](https://ai.google.dev/gemini-api/docs) |
| **Vertex AI** | Custom | Conversion | [cloud.google.com/vertex-ai](https://cloud.google.com/vertex-ai/docs) |
| **Groq** | OpenAI-compatible | Conversion | [groq.com](https://console.groq.com/docs) |
| **Together AI** | OpenAI-compatible | Conversion | [together.ai](https://docs.together.ai/docs) |
| **Fireworks AI** | OpenAI-compatible | Conversion | [fireworks.ai](https://docs.fireworks.ai) |
| **Deepinfra** | OpenAI-compatible | Conversion | [deepinfra.com/docs](https://deepinfra.com/docs) |
| **Cerebras** | OpenAI-compatible | Conversion | [inference.cerebras.ai](https://inference.cerebras.ai/api) |
| **Nebius** | OpenAI-compatible | Conversion | [nebius.ai/docs](https://nebius.ai/docs) |
| **Moonshot AI** | OpenAI-compatible | Conversion | [api.moonshot.cn](https://api.moonshot.cn/docs) |
| **NovitaAI** | OpenAI-compatible | Conversion | [novita.ai](https://docs.novita.ai) |
| **Baseten** | OpenAI-compatible | Conversion | [baseten.co/docs](https://baseten.co/docs) |

---

## Provider Types

### Anthropic-Compatible (Pass-Through)

**How it works:**
- Send Anthropic format → Receive Anthropic format
- No conversion needed - direct pass-through
- Full feature parity with Anthropic API

**Supported features:**
- ✅ Messages API (text, images)
- ✅ Tool use (function calling)
- ✅ Tool results
- ✅ Thinking blocks (extended thinking)
- ✅ Web search results
- ✅ Streaming (SSE)
- ✅ Token counting (Anthropic native only)

**Code location:** `src/providers/anthropic_compatible.rs`

**What to check when broken:**
1. **API changes** - Check provider's Messages API docs
2. **New content types** - Add to `src/models/mod.rs` `ContentBlock` enum
3. **Endpoint changes** - Update base_url in provider constructor
4. **Authentication** - Check if they changed headers/auth format

---

### OpenAI-Compatible (Conversion)

**How it works:**
- Convert Anthropic format → OpenAI format (request)
- Convert OpenAI format → Anthropic format (response)
- Bidirectional translation

**Supported features:**
- ✅ Messages API (text, images)
- ✅ Tool use (function calling)
- ✅ Tool results
- ✅ Streaming (SSE)
- ❌ Thinking blocks (not supported - skipped)
- ❌ Web search results (not supported - skipped)

**Code location:** `src/providers/openai.rs`

**What to check when broken:**
1. **OpenAI API changes** - [platform.openai.com/docs](https://platform.openai.com/docs/api-reference/chat/create)
2. **New response types** - Update `OpenAIResponse` struct
3. **New content types** - Add to `OpenAIContentPart` enum
4. **Tool format changes** - Update `OpenAIToolCall` / `OpenAIFunctionCall`

**Conversion limitations:**
- Web search results → skipped (OpenAI has no equivalent)
- Thinking blocks → skipped (OpenAI has no equivalent)
- Server tool use → skipped (OpenAI has no equivalent)

---

### Google Gemini (Custom)

**How it works:**
- Convert Anthropic format → Gemini format (request)
- Convert Gemini format → Anthropic format (response)
- Custom translation layer

**Supported features:**
- ✅ Messages API (text, images)
- ✅ Tool use (function calling)
- ✅ Tool results
- ✅ Streaming (SSE)
- ❌ Thinking blocks (not supported)
- ❌ Web search results (not supported)

**Code location:** `src/providers/gemini.rs`

**What to check when broken:**
1. **Gemini API changes** - [ai.google.dev/gemini-api/docs](https://ai.google.dev/gemini-api/docs)
2. **New content types** - Update `GeminiContent` enum
3. **Tool format changes** - Update `GeminiTool` / `GeminiFunctionCall`
4. **Streaming format** - Check SSE event structure

---

## Content Block Types

### Anthropic Content Blocks

Located in: `src/models/mod.rs:120`

```rust
pub enum ContentBlock {
    Text { text: String },
    Image { source: ImageSource },
    ToolUse { id, name, input },
    ServerToolUse { id, name, input },
    ToolResult { tool_use_id, content },
    Thinking { thinking, signature },
    WebSearchToolResult { id, content },  // ← Added for z.ai
}
```

**Docs:** [docs.anthropic.com/en/api/messages](https://docs.anthropic.com/en/api/messages)

### OpenAI Content Types

Located in: `src/providers/openai.rs:78`

```rust
pub enum OpenAIContentPart {
    Text { text: String },
    ImageUrl { image_url: OpenAIImageUrl },
}
```

**Note:** OpenAI has tool calls in a separate array, not in content.

**Docs:** [platform.openai.com/docs/api-reference/chat/create](https://platform.openai.com/docs/api-reference/chat/create)

### Gemini Content Types

Located in: `src/providers/gemini.rs`

```rust
pub enum GeminiContent {
    Text { text: String },
    Image { inline_data: GeminiInlineData },
    FunctionCall { name, args },
    FunctionResponse { name, response },
}
```

**Docs:** [ai.google.dev/gemini-api/docs](https://ai.google.dev/gemini-api/docs)

---

## Thinking / Reasoning Modes

**IMPORTANT:** Different providers use different names and approaches for thinking/reasoning!

### Quick Comparison

| Provider | Name | Thinking Visible? | Parameter | Docs |
|----------|------|-------------------|-----------|------|
| **Anthropic** | Extended Thinking | ✅ YES (public) | `thinking: { budget_tokens }` | [docs](https://docs.anthropic.com/en/api/thinking) |
| **OpenAI** | Reasoning Models | ❌ NO (internal) | `max_completion_tokens` | [docs](https://platform.openai.com/docs/guides/reasoning) |
| **Gemini** | Thinking Mode | ❌ NO (internal) | `thinking_budget` | [docs](https://ai.google.dev/gemini-api/docs/thinking) |
| **z.ai** | Thinking | ✅ YES | `thinking: { budget_tokens }` | Anthropic-compatible |
| **MiniMax** | Thinking | ✅ YES | `thinking: { budget_tokens }` | Anthropic-compatible |
| **ZenMux** | Thinking | ✅ YES | `thinking: { budget_tokens }` | Anthropic-compatible |

### Anthropic: Extended Thinking (PUBLIC)

- **Models:** claude-sonnet-4, claude-opus-4
- **Response includes:** `ContentBlock::Thinking { thinking, signature }`
- **Usage:** See thinking process in response

### OpenAI: Reasoning Models (INTERNAL)

- **Models:** o1, o1-mini, o3-mini, o3, GPT-5 series
- **Response:** NO thinking blocks (internal only)
- **Parameter:** `max_completion_tokens` controls reasoning time
- **Our code:** Correctly skips thinking blocks (not returned by OpenAI)

### Gemini: Thinking Mode (INTERNAL)

- **Models:** Gemini 2.5 Pro, Gemini 2.5 Flash
- **Response:** NO thinking blocks (internal only)
- **Parameter:** `thinking_budget` controls thinking time
- **Our code:** Correctly skips thinking blocks (not returned by Gemini)

**Key insight:** OpenAI and Gemini DO have thinking, but it's **internal/hidden** - not returned in API response. Only Anthropic shows thinking publicly.

---

## Recent API Changes

### 2025-01-09: z.ai Web Search Results

**Issue:** Parse error `unknown variant 'web_search_tool_result'`

**Cause:** z.ai added web search tool results (Anthropic feature)

**Fix:** Added `WebSearchToolResult` variant to `ContentBlock` enum

**Files changed:**
- `src/models/mod.rs` - Added variant
- `src/providers/openai.rs` - Skip conversion to OpenAI
- `src/providers/anthropic_compatible.rs` - Token counting

**Reference:**
- Anthropic docs: [Content blocks - Tool results](https://docs.anthropic.com/en/api/messages#tool-results)
- z.ai docs: Check provider for web search documentation

---

## Testing Provider Changes

When a provider updates their API:

1. **Check the provider's API docs** (see links above)
2. **Look for:**
   - New request/response fields
   - New content types
   - Changed authentication
   - New endpoints
3. **Update code:**
   - Add new structs/enums for new types
   - Update conversion logic
   - Add match arms for new variants
4. **Test:**
   - Use Admin UI "Test" tab
   - Check logs for parse errors
   - Verify streaming works
5. **Update this doc** with new info

---

## OAuth Providers

Providers with OAuth support (no API key needed):

| Provider | OAuth Flow | Docs |
|----------|-----------|------|
| **Anthropic** | Authorization Code + PKCE | [docs.anthropic.com/en/docs/oauth](https://docs.anthropic.com/en/docs/oauth) |
| **Google Gemini** | OAuth 2.0 | [ai.google.dev/gemini-api/docs/oauth](https://ai.google.dev/gemini-api/docs/oauth) |
| **ChatGPT/Codex** | Authorization Code + PKCE | Custom |

**Code location:** `src/auth/oauth.rs`

**What to check when broken:**
1. OAuth endpoints changed
2. Token refresh logic changed
3. Scopes or authorization headers changed

---

## Performance Notes

### Pass-Through Providers (Fastest)
- Anthropic-compatible
- ~0ms overhead
- Just proxy requests

### Conversion Providers (Minimal overhead)
- OpenAI-compatible: ~0.5ms per request
- Gemini: ~1ms per request
- JSON conversion is fast in Rust

### Token Counting

| Provider | Method | Accuracy |
|----------|--------|----------|
| **Anthropic** | Official API | 100% |
| **Anthropic-compatible** | Char-based estimate | ~75% |
| **OpenAI-compatible** | tiktoken-rs | ~95% |
| **Gemini** | Char-based estimate | ~75% |

---

## Troubleshooting

### Parse Errors

**Error:** `unknown variant 'X'`
- **Cause:** Provider added new content type
- **Fix:** Add variant to `ContentBlock` enum
- **Check:** Provider's API docs for new features

### Authentication Errors

**Error:** `401 Unauthorized`
- **Cause:** API key invalid or OAuth token expired
- **Fix:**
  - API key: Regenerate from provider dashboard
  - OAuth: Re-authorize via Admin UI
- **Check:** Provider's auth docs

### Streaming Errors

**Error:** SSE connection drops
- **Cause:** Provider changed streaming format
- **Fix:** Update SSE parser in provider code
- **Check:** Provider's streaming docs

### Tool Use Errors

**Error:** Tools not working
- **Cause:** Provider uses different tool format
- **Fix:** Update tool conversion logic
- **Check:** Provider's function calling docs

---

## Adding a New Provider

### Step 1: Determine Type
- **Anthropic-compatible?** Use `AnthropicCompatibleProvider`
- **OpenAI-compatible?** Extend `OpenAIProvider`
- **Custom API?** Create new provider impl

### Step 2: Add Provider Config
Edit `config.toml`:
```toml
[[providers]]
name = "new-provider"
type = "anthropic-compatible"  # or "openai-compatible"
base_url = "https://api.example.com"
api_key = "sk-..."
```

### Step 3: Add to Code
- `src/providers/mod.rs` - Register provider
- `src/server/config.rs` - Parse config
- `src/main.rs` - Initialize provider

### Step 4: Test
1. Add provider in Admin UI
2. Test with simple message
3. Test with tools
4. Test streaming

### Step 5: Document
Update this file with provider info!

---

## Resources

### Provider Aggregators
- **OpenRouter:** [openrouter.ai/docs](https://openrouter.ai/docs) - 500+ models
- **Together AI:** [docs.together.ai](https://docs.together.ai) - Open source models

### Model Hubs
- **Hugging Face:** [huggingface.co/models](https://huggingface.co/models) - Model cards
- **Replicate:** [replicate.com/models](https://replicate.com/models) - Serverless models

### API Documentation
- **Anthropic:** [docs.anthropic.com](https://docs.anthropic.com/en/api/messages)
- **OpenAI:** [platform.openai.com/docs](https://platform.openai.com/docs/api-reference)
- **Google:** [ai.google.dev/gemini-api/docs](https://ai.google.dev/gemini-api/docs)

### Monitoring
- Check provider status pages for outages
- Monitor response times in Admin UI
- Check logs for errors

---

**Last updated:** 2025-01-09

**To update:** When adding/changing providers or when APIs change
