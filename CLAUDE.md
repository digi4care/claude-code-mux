# RapidSpec Instructions

AI assistant instructions for this project using RapidSpec workflow.

Always reference `@/rapidspec/AGENTS.md` when:
- Planning features or changes
- Creating proposals or specs
- Making architectural decisions
- Need clarification on workflow

See `@/rapidspec/AGENTS.md` for:
- RapidSpec workflow and commands
- Spec-driven development process
- AI agents and their usage
- Project conventions

Keep this file so `rapid update` can refresh instructions.

## Version Management

**Always bump the version according to Semantic Versioning (SemVer) before every release/build:**

- **Cargo.toml** - Update `version` field (currently `0.6.3`)
- **CHANGELOG.md** - Add new entry following [Keep a Changelog](https://keepachangelog.com/en/1.0.0/) format
  - Use sections: Added, Changed, Deprecated, Removed, Fixed, Security
  - Add version comparison link at bottom (e.g., `[0.6.4]: https://github.com/9j/claude-code-mux/compare/v0.6.3...v0.6.4`)

**SemVer Rules:**
- **MAJOR** (0.x.0 → 1.0.0): Breaking changes
- **MINOR** (0.6.x → 0.7.0): New features (backwards compatible)
- **PATCH** (0.6.3 → 0.6.4): Bug fixes (backwards compatible)

## Architecture Guidelines

### Admin UI State Management

The admin UI (`src/server/admin.html`) uses two complementary state management patterns:

#### 1. URL-based State Management
See `@/docs/url-state-management.md` for detailed documentation.

- Navigation state (tabs, views) is stored in URL parameters
- Enables shareable URLs, browser history, and bookmarking
- Example: `?tab=providers&view=add`

#### 2. LocalStorage-based State Management
See `@/docs/localstorage-state-management.md` for detailed documentation.

**Critical Architecture Decision**: The server loads TOML config on startup and **does not reload until restart**. Therefore:

- ✅ **Correct**: Use localStorage as client-side cache
  - Page load: Fetch from server → save to localStorage
  - All operations (add/delete/edit): Update localStorage only
  - "Save" button: Sync localStorage → server
  - "Save & Restart" button: Sync → restart server

- ❌ **Wrong**: Fetch from server after each operation
  - Server returns stale data until restart
  - Causes inconsistent UI state

**Key Functions**:
- `loadConfig()` - Fetch from server (only on page load)
- `saveToLocalStorage(config)` - Save to localStorage
- `syncToServer()` - Sync localStorage to server (only called by save buttons)
- All CRUD operations update localStorage only, then call `saveToLocalStorage()`

**When modifying admin UI**:
1. Never add server fetches in CRUD operations
2. All operations must update `appState.config` → `saveToLocalStorage()`
3. Only save buttons call `syncToServer()`
4. Always notify user: "(Press the Save button to apply)"
