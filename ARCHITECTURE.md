# ORAND Conversation Brancher - Architecture & Logic

## Overview

ORAND Conversation Brancher is a single-file HTML application that enables non-linear conversations with Large Language Models (LLMs). It implements a tree-based conversation structure with branching, forking, lifecycle management, and full-text search capabilities.

**Version**: 1.3  
**Last Modified**: 2026-03-22  
**Architecture**: Client-side only (no backend server)

---

## Table of Contents

1. [Application Architecture](#application-architecture)
2. [Data Model](#data-model)
3. [Core Components](#core-components)
4. [Conversation Tree Logic](#conversation-tree-logic)
5. [Branch Lifecycle](#branch-lifecycle)
6. [State Management](#state-management)
7. [Search System](#search-system)
8. [Data Persistence](#data-persistence)
9. [LLM Integration](#llm-integration)
10. [Export System](#export-system)
11. [Security & Privacy](#security--privacy)

---

## Application Architecture

### High-Level Structure

The application is organized into clearly separated sections (marked with `SECTION:` comments):

```
├── META           - App metadata and version info
├── CONFIG         - Configuration constants
├── STYLE          - CSS styling with design tokens
├── STATE          - Application state management
├── UI             - User interface rendering
├── LOGIC:db       - IndexedDB operations
├── LOGIC:actionlog - Audit trail management
├── LOGIC:lmstudio - LM Studio API integration
├── LOGIC:tree     - Conversation tree management
├── LOGIC:branch   - Branch operations
├── LOGIC:search   - Full-text search functionality
├── LOGIC:export   - Markdown export functionality
├── EVENTS         - Event handlers
├── INIT           - Application initialization
└── USERGUIDE      - Built-in documentation
```

### Technology Stack

- **Frontend**: Vanilla JavaScript (ES6+), HTML5, CSS3
- **Storage**: IndexedDB (browser-native database)
- **LLM Integration**: LM Studio API (OpenAI-compatible)
- **Build**: None required (single HTML file)

---

## Data Model

### Database Schema (IndexedDB)

The application uses 5 object stores:

#### 1. **conversations**
```javascript
{
  id: string,              // Unique identifier
  title: string,           // User-defined title
  createdAt: number        // Timestamp
}
// Index: createdAt
```

#### 2. **nodes**
```javascript
{
  id: string,              // Unique identifier
  convoId: string,         // Parent conversation ID
  parentId: string|null,   // Parent node (null for trunk)
  forkAfterMsgIdx: number, // Message index where fork occurred
  label: string,           // Display name
  color: string,           // Branch color (#hex)
  isTrunk: boolean,        // Is this the main conversation?
  canBranch: boolean,      // Can create forks from this node?
  forks: Array<{           // Fork points created from this node
    forkMsgIdx: number,
    branchIds: string[]
  }>,
  status: string,          // 'active'|'committed'|'discarded'|'promoted'|'split'
  resolved: boolean,       // Is this branch resolved?
  resolvedAt: number,      // Timestamp of resolution
  commitSummary: string,   // Summary when committed (optional)
  splitToConvoId: string,  // New conversation ID when split (optional)
  createdAt: number        // Timestamp
}
// Indexes: convoId, status
```

#### 3. **messages**
```javascript
{
  id: string,              // Unique identifier
  nodeId: string,          // Node this message belongs to
  role: string,            // 'user'|'assistant'|'system'
  content: string,         // Message content
  createdAt: number        // Timestamp
}
// Index: nodeId
```

#### 4. **snapshots**
```javascript
{
  id: string,              // Unique identifier
  nodeId: string,          // Branch node ID
  summary: string,         // LLM-generated summary of parent context
  rawCount: number,        // Number of messages summarized
  model: string,           // LLM model used
  createdAt: number        // Timestamp
}
// Index: nodeId
```

#### 5. **actionLog**
```javascript
{
  id: string,              // Unique identifier
  convoId: string,         // Conversation ID
  action: string,          // Action type
  nodeId: string,          // Node acted upon (optional)
  label: string,           // Node label (optional)
  description: string,     // Human-readable description
  meta: object,            // Additional metadata
  timestamp: number        // Timestamp
}
// Index: convoId
```

---

## Core Components

### 1. Configuration (`CFG`)

```javascript
CFG = {
  DB_NAME: 'orand_brancher',
  DB_VERSION: 2,
  MAX_BRANCHES_PER_FORK: 4,
  BRANCH_CAN_BRANCH: false,          // Future: nested branching
  BRANCH_COLORS: ['#e74c3c','#27ae60','#8e44ad','#e67e22'],
  SUMMARY_MAX_TOKENS: 400,
  LM_DEFAULT_URL: 'http://localhost:1234'
}
```

### 2. Application State

```javascript
state = {
  activeConvoId: string,              // Current conversation
  activeNodeId: string,               // Current node (trunk or branch)
  nodes: {},                          // nodeId -> node object (cache)
  messages: {},                       // nodeId -> [messages] (cache)
  forkTarget: number,                 // Message index for forking
  pendingForkBranches: [],            // Branches being created
  lmConnected: boolean,               // LM Studio connection status
  searchQuery: string,                // Current search query
  searchResults: [],                  // Array of search result objects
  activeSearchHighlight: string       // Search term being highlighted
}
```

---

## Conversation Tree Logic

### Tree Structure

```
Conversation (conversations)
└── Trunk Node (nodes)
    ├── Messages (messages)
    └── Forks[]
        ├── Fork @ message N
        │   ├── Branch A (node)
        │   │   ├── Snapshot (snapshots)
        │   │   └── Messages (messages)
        │   ├── Branch B (node)
        │   └── Branch C (node)
        └── Fork @ message M
            └── Branch D (node)
```

### Node Types

1. **Trunk Node**
   - `isTrunk: true`
   - Represents the main conversation thread
   - Can have multiple forks
   - Cannot be deleted or discarded

2. **Branch Node**
   - `isTrunk: false`
   - Created from a fork point
   - Has a parent node (trunk or another branch in future)
   - Can have status: active, committed, discarded, promoted, split

### Fork Creation Process

```
1. User clicks "Add Fork" on a message
2. System captures fork point (message index)
3. User defines N branch prompts (max 4)
4. System creates:
   a. N new branch nodes
   b. Summary of conversation up to fork point
   c. Snapshot for each branch
   d. System message with context summary
   e. User message with branch prompt
5. System calls LM Studio API in parallel for all branches
6. Responses added as assistant messages
7. Fork entry added to parent node
```

### Rendering Logic (Sidebar Tree)

```javascript
// For each conversation:
//   Render trunk node
//   If active conversation:
//     For each fork in trunk.forks:
//       Count active branches:
//         - Include: no status, 'active', 'committed'
//         - Exclude: 'discarded', 'promoted', 'split'
//       If activeBranches > 0:
//         Render fork connector
//         For each branchId:
//           If not discarded/promoted/split:
//             Render branch node (with ✓ if committed)
```

**Why committed branches are hidden**: If a fork has ALL branches resolved (discarded/promoted/split), the entire fork is hidden. If at least one active or committed branch exists, committed branches show with a checkmark.

---

## Branch Lifecycle

### State Transitions

```
        [Created]
           ↓
      [active] ←────────────┐
         ↓  ↓  ↓            │
         ↓  ↓  ↓            │
    discarded promoted committed
         ↓      ↓        ↓  split
         ↓      ↓     (trunk) ↓
     [Hidden] [Trunk] [Visible] [New Tree]
```

### 1. **Discard Branch**

**Purpose**: Mark branch as not useful, hide from tree

**Process**:
```javascript
1. Set node.status = 'discarded'
2. Set node.resolvedAt = timestamp
3. Update database
4. Log action to audit trail
5. Switch to trunk node
6. Re-render tree (branch hidden)
```

**Data**: All messages preserved, visible in export and audit log

### 2. **Commit Branch**

**Purpose**: Add branch insights to trunk as context

**Process**:
```javascript
1. Generate summary of branch messages
2. Add summary as assistant message to trunk:
   "📌 Committed branch '[name]':\n[summary]"
3. Set node.status = 'committed'
4. Set node.commitSummary = summary
5. Set node.resolvedAt = timestamp
6. Log action
7. Switch to trunk
8. Re-render tree (branch still visible with ✓)
```

**Result**: Branch context available to trunk, branch remains visible

### 3. **Promote Branch**

**Purpose**: Replace trunk with branch content

**Process**:
```javascript
1. Find sibling branches at same fork
2. Soft-delete all sibling branches (status='discarded')
3. Get trunk messages
4. Delete trunk messages after fork point
5. Copy all branch messages to trunk
6. Set branch.status = 'promoted'
7. Set branch.resolvedAt = timestamp
8. Log actions for branch and siblings
9. Switch to trunk
10. Re-render tree
```

**Result**: Branch becomes the main conversation, siblings hidden

### 4. **Split to New Tree**

**Purpose**: Make branch an independent conversation

**Process**:
```javascript
1. Create new conversation with title "Split: [branch-label]"
2. Copy all branch messages to new conversation's trunk
3. Set node.status = 'split'
4. Set node.splitToConvoId = new conversation ID
5. Set node.resolvedAt = timestamp
6. Log action
7. Switch to new conversation
8. Re-render tree (branch hidden from original)
```

**Result**: Branch becomes independent conversation, preserved in audit

---

## State Management

### Caching Strategy

- **nodes**: Loaded per conversation, cached in `state.nodes`
- **messages**: Loaded on-demand, cached in `state.messages`
- **conversations**: Loaded when rendering tree switcher

### Active Node Management

```javascript
async function setActiveNode(nodeId) {
  state.activeNodeId = nodeId;
  await renderChat();        // Update message display
  updateActionButtons();     // Show/hide branch actions
}
```

### Tree Rendering

Triggered on:
- Conversation switch
- Node creation (fork, new conversation)
- Branch lifecycle action (discard, commit, promote, split)
- Node rename

---

## Search System

### Overview

Version 1.3 introduces full-text search capabilities across all conversations, nodes, and messages. The search system provides real-time filtering with debounced input, result highlighting, and persistent highlighting across navigation.

### Search UI Components

**Search Container** (`#searchContainer`):
- Input field with placeholder text
- Clear button (×) to reset search
- Results panel showing matching items
- Collapsible panel that expands when results are present

**Search Input** (`#searchInput`):
- Real-time search with 300ms debounce
- Minimum 2 characters to trigger search
- Case-insensitive matching
- Partial word matching supported

**Search Results** (`#searchResults`):
- Maximum 50 results displayed
- Result format: `"[Match]" in [Location]`
- Contextual snippets (~60 characters before/after match)
- Click to navigate to matched item

### Search Functionality

#### 1. **performSearch(query)**

Searches across multiple IndexedDB stores:

```javascript
async function performSearch(query) {
  // Searches in:
  // 1. Conversation titles
  // 2. Node labels
  // 3. Message content (user, assistant, system)
  
  // Returns array of result objects:
  {
    type: 'conversation|node|message',
    id: string,
    convoId: string,
    nodeId: string,
    content: string,
    snippet: string,     // Contextual text around match
    highlightedSnippet: string  // With <span> highlights
  }
}
```

**Performance Optimizations**:
- Limit to 50 results per search
- Uses IndexedDB cursors for efficient scanning
- Debounced input prevents excessive queries

#### 2. **renderSearchResults()**

Displays search results in UI:
- Groups results by type (conversations, nodes, messages)
- Shows contextual snippets with highlighted search terms
- Provides click handlers for navigation

#### 3. **highlightSearchTerm(text, term)**

Highlights search matches in result snippets:
```javascript
// Wraps matches in <span class="search-highlight">
// Example: "the quick brown" → "the <span>quick</span> brown"
```

#### 4. **highlightInMessage(text, searchTerm)**

Highlights search matches in conversation messages:
- Applied when viewing messages in chat panel
- Uses yellow background highlighting
- Persists across navigation when initiated from search results

#### 5. **clearSearch()** vs **hideSearchUI()**

**clearSearch()**:
- Clears search input and results
- Removes all highlighting from messages
- Re-renders chat to show unhighlighted content
- Used when user manually clears search

**hideSearchUI()**:
- Collapses search results panel
- Preserves highlighting in messages
- Used when navigating via search results
- Maintains visual context for user

### Search State Management

**state.searchQuery**:
- Current search query string
- Updated on input changes
- Empty string when no search active

**state.searchResults**:
- Array of matching result objects
- Populated by performSearch()
- Cleared when search is reset

**state.activeSearchHighlight**:
- Search term currently being highlighted
- Persists across navigation
- Cleared only by clearSearch()

### Navigation with Search Highlighting

Enhanced navigation functions support persistent highlighting:

```javascript
// Load conversation with optional highlight preservation
async function loadConversation(convoId, preserveSearchHighlight = false)

// Set active node with optional highlight preservation
async function setActiveNode(nodeId, preserveSearchHighlight = false)
```

**Workflow**:
1. User performs search
2. Clicks search result
3. System navigates to result location
4. `preserveSearchHighlight=true` passed
5. Highlighting remains visible in destination
6. Search UI collapses but highlighting persists

### Event Handlers

**Search Input (`#searchInput`)**:
- `input` event with 300ms debounce
- Minimum 2-character threshold
- Calls performSearch() and renderSearchResults()

**Clear Button (`#searchClear`)**:
- `click` event
- Calls clearSearch()
- Resets all search state

**Search Result Items (`.search-result-item`)**:
- `click` event
- Calls hideSearchUI() (not clearSearch())
- Navigates to result with preserveSearchHighlight=true

### CSS Styling

```css
.search-highlight {
  background-color: #ffeb3b;
  font-weight: bold;
  padding: 2px 0;
}

#searchContainer {
  border-bottom: 1px solid #444;
  padding: 10px;
}

.search-result-item {
  cursor: pointer;
  padding: 8px;
  transition: background-color 0.2s;
}

.search-result-item:hover {
  background-color: #2a2a2a;
}
```

### Use Cases

1. **Find Past Discussions**: Locate conversations about specific topics
2. **Navigate Large Trees**: Jump directly to relevant branches
3. **Content Review**: Find all mentions of a term across conversations
4. **Research**: Compare how different branches addressed same topic
5. **Context Recovery**: Return to important discussion points

### Performance Considerations

- **Debouncing**: 300ms delay prevents excessive queries
- **Result Limiting**: 50-item cap prevents UI overload
- **Lazy Highlighting**: Only highlights visible messages
- **Caching**: Search results cached until query changes

### Future Enhancements

Potential improvements documented in [VERSION_CONTROL_PROPOSALS.md](VERSION_CONTROL_PROPOSALS.md):
- Advanced filters (by date, node type, role)
- Regular expression search
- Fuzzy matching
- Search within specific conversations
- Search history/saved searches

---

## Data Persistence

### IndexedDB Operations

**Transaction Types**:
- `readonly`: For queries
- `readwrite`: For updates

**Core Operations**:
```javascript
dbPut(store, object)              // Insert or update
dbGet(store, key)                 // Get by primary key
dbGetAllByIndex(store, idx, val)  // Query by index
dbDelete(store, key)              // Delete (rarely used)
dbTx(store)                       // Get transaction for raw access
```

### Database Recovery

If IndexedDB fails to open:
1. Delete existing database
2. Recreate with current schema
3. Data is lost (user should export regularly)

### Migration Strategy

When `DB_VERSION` increases:
- `onupgradeneeded` handler creates new stores/indexes
- Existing data preserved
- New indexes added to existing stores

---

## LLM Integration

### LM Studio Connection

**Default**: `http://localhost:1234`  
**Protocol**: OpenAI-compatible API

### API Workflow

```javascript
1. Fetch available models: GET /v1/models
2. User selects model
3. For each message:
   a. Build conversation context
   b. Add system prompt (if summary exists)
   c. POST /v1/chat/completions
   d. Stream or receive full response
   e. Store as assistant message
```

### Context Management

**For Trunk Messages**:
- Full conversation history (all messages)
- Optional system prompt from config

**For Branch Messages**:
- System message with summary of parent up to fork
- Branch-specific messages only
- Prevents context pollution between branches

### Error Handling

- **Connection refused**: Show diagnostic with checklist
- **CORS error**: Explain LM Studio CORS settings
- **HTTP error**: Display status code
- **Timeout**: Configurable, default 60s

---

## Export System

### Markdown Export Format

```markdown
# [Conversation Title]

**Exported**: [Timestamp]  
**Model**: [LLM Model]  
**Message Count**: X trunk + Y branches = Z total

---

## Trunk: Main Conversation

**Message N** | User | [Time]
[Message content]

**Message N+1** | Assistant | [Time]
[Message content]

### ⎇ Fork 1 @ message X

---

#### Branch A: [Label]

| Attribute | Value |
|-----------|-------|
| **Color** | #hex |
| **Status** | [status] |
| **Created** | [Date] |
| **Fork point** | After message X |

**Context Snapshot**  
[Summary]

[Branch messages...]

---

#### Branch B: [Label]
...

---

## Action Log

| # | Timestamp | Action | Description |
|---|-----------|--------|-------------|
| 1 | [Time] | fork_created | ... |
| 2 | [Time] | committed | ... |

---

## Summary

- **Conversation**: [Title]
- **Duration**: [Time range]
- **Branches**: X active, Y resolved
```

### Export Process

```javascript
1. Load conversation data
2. Load all nodes (trunk + branches)
3. Load all messages per node
4. Load action log
5. Build Markdown structure
6. Trigger browser download
   - Modern: File System Access API
   - Fallback: Blob URL download
```

---

## Security & Privacy

### Data Storage

- **Location**: Browser's IndexedDB (origin-isolated)
- **Encryption**: None (data stored in plaintext locally)
- **Access**: Only accessible by same origin (protocol + domain + port)

### Network Communication

- **LM Studio**: Local HTTP requests to localhost
- **No External APIs**: All data stays on device
- **No Analytics**: No tracking or telemetry

### Privacy Considerations

1. **Shared Computers**: Data persists in browser profile
2. **Browser Data Clearing**: Deletes all conversations
3. **Private/Incognito Mode**: Data lost when session ends
4. **Export Recommended**: Regular backups via Markdown export

### CORS Requirements

For LM Studio integration:
```
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST
Access-Control-Allow-Headers: Content-Type
```

---

## Performance Considerations

### Optimization Strategies

1. **Lazy Loading**: Messages loaded per node on-demand
2. **Caching**: Nodes and messages cached in memory
3. **Indexes**: Database queries use indexes (convoId, nodeId, status)
4. **Parallel API Calls**: Branch creation sends all LLM requests simultaneously
5. **Transaction Batching**: Related DB operations in single transaction

### Scalability Limits

- **IndexedDB**: ~50MB typical limit (browser-dependent)
- **Large Conversations**: Performance degrades with 100+ branches
- **Long Messages**: No chunking (single message = single DB record)
- **Concurrent Branches**: Max 4 per fork (configurable)

### Future Optimizations

- Message pagination for very long conversations
- Virtual scrolling for large tree views
- Background summarization for old messages
- Selective node loading (load branches on expand)

---

## Error Handling

### Database Errors

- Open failure → Delete and recreate DB
- Transaction errors → Logged to console, user notified
- Quota exceeded → Alert user to clear data or export

### LLM Errors

- Connection refused → Diagnostic modal
- CORS blocked → Instructions to fix LM Studio settings
- API errors → Display in chat as `[Error: ...]`
- Timeout → Retry option available

### UI Errors

- Invalid state → Reset to trunk node
- Missing nodes → Log warning, skip in tree
- Render failures → Fallback to empty state message

---

## Future Architecture Considerations

### Implemented in v1.3 ✅

- **Full-Text Search**: Search across conversations, nodes, and messages with highlighting

### Potential Enhancements

1. **Nested Branching**: Enable branches to fork (requires `BRANCH_CAN_BRANCH`)
2. **Multi-Model Comparison**: Run same prompt across multiple models
3. **Real-time Collaboration**: SharedWorker or WebRTC for multi-tab sync
4. **Cloud Sync**: Optional encrypted backup to cloud storage
5. **Import/Export JSON**: Structured data format for sharing
6. **Templates**: Predefined conversation starters with branching
7. **Advanced Search Filters**: Filter by date, node type, role; regex and fuzzy matching
8. **Diff Viewer**: Compare responses across branches side-by-side
9. **Version Control**: Git-like commit/branch/merge workflows (see [VERSION_CONTROL_PROPOSALS.md](VERSION_CONTROL_PROPOSALS.md))
10. **Plugin System**: Extend with custom actions and integrations

### Architectural Constraints

- **Single File Philosophy**: Keep zero-dependency, portable
- **No Build Step**: Vanilla JavaScript only
- **Browser Compatibility**: ES6+ features, IndexedDB required
- **Local-First**: Never require external server for core functionality

---

## Glossary

- **Trunk**: The main conversation thread (root node)
- **Branch**: An alternative conversation path from a fork point
- **Fork**: A branching point in the conversation with multiple paths
- **Node**: Generic term for either trunk or branch in the tree
- **Snapshot**: LLM-generated summary of conversation context
- **Commit**: Adding branch insights to trunk as context
- **Promote**: Making a branch the new trunk
- **Discard**: Hiding a branch from the tree (soft delete)
- **Split**: Converting a branch into a new independent conversation
- **Resolved**: A branch that has been committed, discarded, promoted, or split
- **Active**: A branch that is still in use and hasn't been resolved
- **Search Highlight**: Visual emphasis (yellow background) on search term matches in messages
- **Debouncing**: Delaying search execution until user stops typing (300ms delay)
- **Search Result**: A match found in conversation titles, node labels, or message content

---

## Code Organization Best Practices

### Section Markers

All major sections are marked with clear comments:
```javascript
// ── SECTION NAME ────────────────────────────────────────────────────
```

### Function Naming Conventions

- **Async operations**: Prefix with `async function`
- **Database ops**: Prefix with `db` (e.g., `dbPut`, `dbGet`)
- **Rendering**: Prefix with `render` (e.g., `renderTree`, `renderChat`)
- **Actions**: Verb-first (e.g., `discardBranch`, `commitBranch`)

### Event Handler Pattern

```javascript
// Defined in EVENTS section
document.getElementById('elem').addEventListener('click', handlerFunction);

// Handler implemented above in appropriate LOGIC section
async function handlerFunction() { ... }
```

---

**Document Version**: 1.1 (for App v1.3)  
**Last Updated**: 2026-03-22  
**Maintainer**: ORAND Conversation Brancher Project
