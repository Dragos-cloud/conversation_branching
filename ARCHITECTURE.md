# ORAND Praxis - Architecture & Logic

## Overview

ORAND Praxis is a single-file HTML application that enables non-linear conversations with Large Language Models (LLMs). It implements a tree-based conversation structure with branching, forking, lifecycle management, RAG document integration, and full-text search capabilities.

**Version**: 2.0.0  
**Last Modified**: 2026-03-24  
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
8. [RAG Document System](#rag-document-system-v20)
9. [Data Persistence](#data-persistence)
10. [LLM Integration](#llm-integration)
11. [Export System](#export-system)
12. [Security & Privacy](#security--privacy)

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
├── LOGIC:documents - RAG document processing
├── EVENTS         - Event handlers
├── INIT           - Application initialization
└── USERGUIDE      - Built-in documentation
```

### Technology Stack

- **Frontend**: Vanilla JavaScript (ES6+), HTML5, CSS3
- **Storage**: IndexedDB (browser-native database)
- **LLM Integration**: LM Studio API (OpenAI-compatible)
- **Document Processing**: PDF.js (v3.11.174), Mammoth.js (v1.6.0)
- **Build**: None required (single HTML file)

---

## Data Model

### Database Schema (IndexedDB)

The application uses 6 object stores:

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
  messages: Array<{        // Exact copies of parent context messages (v1.5)
    role: string,          // 'user'|'assistant'
    content: string,       // Message content
    createdAt: number      // Message timestamp
  }>,
  rawCount: number,        // Number of messages in snapshot
  model: string,           // LLM model used
  createdAt: number        // Timestamp
}
// Index: nodeId
// Note: v1.5 changed from summary string to messages array for exact context preservation
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

#### 6. **documents** (v2.0)
```javascript
{
  id: string,              // Unique identifier
  convoId: string,         // Parent conversation ID
  filename: string,        // Original file name
  content: string,         // Extracted text content
  format: string,          // File extension (pdf, docx, txt, md, json, csv)
  uploadedAt: number       // Timestamp
}
// Indexes: convoId, uploadedAt
```

---

## Core Components

### 1. Configuration (`CFG`)

```javascript
CFG = {
  DB_NAME: 'orand_brancher',
  DB_VERSION: 3,                     // v2.0: Added documents store
  MAX_BRANCHES_PER_FORK: 4,
  BRANCH_CAN_BRANCH: false,          // Future: nested branching
  BRANCH_COLORS: ['#e74c3c','#27ae60','#8e44ad','#e67e22'],
  SUMMARY_MAX_TOKENS: 400,           // Deprecated in v1.5 (exact context used instead)
  LM_DEFAULT_URL: 'http://localhost:1234',
  MAX_DOCUMENT_FILE_SIZE: 10485760,  // 10MB raw file upload limit
  MAX_DOCUMENT_CONTENT_SIZE: 5242880 // 5MB extracted text content limit
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
  activeSearchHighlight: string,      // Search term being highlighted
  activeDocuments: []                 // Documents for active conversation (v2.0)
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
   b. Exact copies of conversation messages up to fork point (v1.5)
   c. Snapshot for each branch containing full message array
   d. User message with branch prompt
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
//       For each branchId:
//         Render branch node (includes all resolved branches: v1.4 discarded, v1.6.2 promoted, v1.6.3 split)
//         Branch styling based on status:
//           - 'active': normal styling
//           - 'committed': checkmark indicator
//           - 'discarded': dimmed with strikethrough (v1.4)
//           - 'promoted': visible with ↑ indicator and blue gradient (v1.6.2)
//           - 'split': visible with ⎇ indicator and purple/green gradient (v1.6.3)
```

**Tree Visibility Rules (v1.6.3)**: All branch statuses are visible for complete audit trail. Fork entries are never hidden. Active, committed, discarded, promoted, and split branches all remain visible with distinct styling. Promoted branches show with ⬆ badge and blue styling. Split branches show with ⎇ badge and purple styling plus target conversation ID.

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

### 2. **Commit Branch (v1.6 - Selective Injection)**

**Purpose**: Selectively inject chosen branch messages into trunk

**Process**:
```javascript
1. Show interactive modal with all branch messages (checkboxes, role badges, previews)
2. User selects which messages to inject (all selected by default)
3. User clicks "Commit Selected" (or cancels)
4. Add header message to trunk:
   "📌 Committed N messages from branch '[name]':"
5. Inject selected messages directly to trunk (preserving role and content)
6. Set node.status = 'committed'
7. Set node.commitSummary = "N of M messages injected"
8. Set node.resolvedAt = timestamp
9. Log action with message indices for audit trail
10. Switch to trunk
11. Re-render tree (branch still visible with ✓)
```

**v1.6 Changes**:
- Replaced LLM summary generation with user-controlled message selection
- Interactive modal UI with checkbox selection, "Select All" toggle, selection counter
- Direct message injection instead of summarized context
- Detailed action log with selected message indices
- Faster commit operation (no LLM call)

**Result**: Selected branch messages directly available in trunk, branch remains visible with commit indicator

### 3. **Promote Branch**

**Purpose**: Create new conversation with full original context plus branch direction (v1.7)

**Process**:
```javascript
1. Get all trunk messages and branch messages
2. Create new conversation with title "Promote: [branch-label]"
3. Copy ALL trunk messages up to fork point to new conversation
4. Copy all branch messages to new conversation (after trunk messages)
5. Set node.status = 'promoted'
6. Set node.promotedToConvoId = new conversation ID
7. Set node.resolvedAt = timestamp
8. Soft-delete all sibling branches (status='discarded')
9. Log actions for branch and siblings
10. Switch to new conversation
11. Re-render tree
```

**Context Model**: Full trunk history + branch direction (complete original context)

**Result**: New conversation created with full context, original conversation intact, promoted branch visible in original tree with ↑ indicator and target conversation ID

### 4. **Split to New Tree**

**Purpose**: Create new independent conversation from fork point only

**Process**:
```javascript
1. Create new conversation with title "Split: [branch-label]"
2. Copy fork-point context messages to new conversation (from node.contextMessages)
3. Copy all branch messages to new conversation's trunk
4. Set node.status = 'split'
5. Set node.splitToConvoId = new conversation ID
6. Set node.resolvedAt = timestamp
7. Log action
8. Switch to new conversation
9. Re-render tree
```

**Context Model**: Fork-point snapshot only (fresh start from fork)

**Result**: New independent conversation created, original conversation intact, split branch visible in original tree with ⎇ indicator and target conversation ID (v1.6.3)

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

## RAG Document System (v2.0)

### Overview

Version 2.0 introduces Retrieval-Augmented Generation (RAG) capabilities, allowing users to upload documents that ground LLM responses in specific content. Documents are linked to conversations and automatically injected into LLM requests as system prompts.

### Document Processing Pipeline

```
User Upload → File Validation → Format Detection → Parsing → Storage → Context Injection
```

### Document Storage Model

Documents are stored per conversation in the `documents` object store:

```javascript
{
  id: string,              // Unique document ID (UUID)
  convoId: string,         // Parent conversation ID (foreign key)
  filename: string,        // Original file name with extension
  content: string,         // Extracted plain text content
  format: string,          // File extension (pdf, docx, txt, md, json, csv)
  uploadedAt: number       // Timestamp of upload
}
```

**Key Design Decisions**:
- Documents linked to conversations, not nodes (branches inherit parent conversation's docs)
- One-to-many relationship: conversation → multiple documents
- Text-only storage (no binary data retained after parsing)
- No versioning (replaced on re-upload)

### Document Parsing Functions

#### 1. **parseDocumentFile(file)**

Master parsing function that routes to format-specific handlers:

```javascript
async function parseDocumentFile(file) {
  const filename = file.name.toLowerCase();
  
  if (filename.endsWith('.pdf')) {
    return await parsePDF(file);
  } else if (filename.endsWith('.docx')) {
    return await parseDOCX(file);
  } else {
    return await parseTextFile(file);
  }
}
```

#### 2. **PDF Parsing (PDF.js)**

```javascript
// Uses PDF.js v3.11.174 from CDN
// Worker: pdf.worker.min.js

async function parsePDF(file) {
  const arrayBuffer = await file.arrayBuffer();
  pdfjsLib.GlobalWorkerOptions.workerSrc = '[CDN_URL]';
  
  const pdf = await pdfjsLib.getDocument({ data: arrayBuffer }).promise;
  let fullText = '';
  
  // Process each page sequentially
  for (let i = 1; i <= pdf.numPages; i++) {
    const page = await pdf.getPage(i);
    const textContent = await page.getTextContent();
    const pageText = textContent.items.map(item => item.str).join(' ');
    fullText += pageText + '\n\n';
  }
  
  return fullText.trim();
}
```

**Capabilities**:
- Multi-page text extraction
- Handles text-based PDFs (not scanned images)
- Preserves paragraph breaks
- No OCR support

#### 3. **DOCX Parsing (Mammoth.js)**

```javascript
// Uses Mammoth.js v1.6.0 from CDN

async function parseDOCX(file) {
  const arrayBuffer = await file.arrayBuffer();
  const result = await mammoth.extractRawText({ arrayBuffer });
  return result.value.trim();
}
```

**Capabilities**:
- Raw text extraction (no formatting)
- Tables converted to plain text
- Images ignored
- Fast processing

#### 4. **Plain Text Files**

```javascript
async function parseTextFile(file) {
  return await file.text();
}
```

Supports: `.txt`, `.md`, `.json`, `.csv`

### Document Management Functions

#### **getConversationDocuments(convoId)**

Retrieves all documents for a conversation:

```javascript
async function getConversationDocuments(convoId) {
  if (!convoId) return [];
  return await dbGetAllByIndex('documents', 'convoId', convoId);
}
```

#### **uploadDocument(convoId, file)**

Uploads and stores a document:

```javascript
async function uploadDocument(convoId, file) {
  const content = await parseDocumentFile(file);
  
  // Validate extracted content size (5MB limit)
  if (content.length > CFG.MAX_DOCUMENT_CONTENT_SIZE) {
    throw new Error('Extracted text too large');
  }
  
  const doc = {
    id: uid(),
    convoId,
    filename: file.name,
    content: content,
    format: file.name.split('.').pop().toLowerCase(),
    uploadedAt: Date.now(),
  };
  await dbPut('documents', doc);
  return doc;
}
```

**Validation**:
- File type checking: `.pdf`, `.docx`, `.txt`, `.md`, `.json`, `.csv`
- **File size limit**: 10MB maximum for raw upload
- **Content size limit**: 5MB maximum for extracted text
- Active conversation required
- Error handling for parsing failures and size violations

#### **deleteDocument(docId)**

Removes a single document:

```javascript
async function deleteDocument(docId) {
  await dbDelete('documents', docId);
}
```

#### **clearAllDocuments(convoId)**

Removes all documents from a conversation:

```javascript
async function clearAllDocuments(convoId) {
  const docs = await getConversationDocuments(convoId);
  for (const doc of docs) {
    await deleteDocument(doc.id);
  }
}
```

### RAG Context Injection

#### System Prompt Construction

When documents are present, a RAG system prompt is injected before all other messages:

```javascript
const ragSystemPrompt = `You are a helpful AI assistant with access to the following documents. When answering questions, prioritize information from these documents. If the answer is not in the documents, you may use your general knowledge but clearly indicate when you're doing so.

### KNOWLEDGE BASE ###

${knowledgeBase}

### END KNOWLEDGE BASE ###

Please answer the user's questions based on the documents provided above when relevant.`;
```

**Knowledge Base Format**:
```
### Document: filename1.pdf ###
[Full extracted content]

---

### Document: filename2.docx ###
[Full extracted content]
```

#### Integration Points

**1. sendMessage() - Trunk & Branch Messages**

```javascript
async function sendMessage() {
  // ... existing code ...
  
  let apiMsgs = [];
  
  // RAG: Inject documents if present
  const docs = await getConversationDocuments(node.convoId);
  if (docs && docs.length > 0) {
    const knowledgeBase = docs.map(doc => 
      `### Document: ${doc.filename} ###\n${doc.content}`
    ).join('\n\n---\n\n');
    
    apiMsgs.push({ role: 'system', content: ragSystemPrompt });
  }
  
  // Continue building context...
  // For branches: add snapshot messages
  // Add current messages
  
  const reply = await callLM(apiMsgs);
  // ... store reply ...
}
```

**2. launchBranches() - Fork Creation**

```javascript
async function launchBranches() {
  // ... existing code ...
  
  const lmCalls = branchIds.map(async (bid) => {
    let apiMsgs = [];
    
    // RAG: Inject documents for each branch
    const docs = await getConversationDocuments(node.convoId);
    if (docs && docs.length > 0) {
      apiMsgs.push({ role: 'system', content: ragSystemPrompt });
    }
    
    // Add snapshot context
    // Add branch messages
    // Call LLM
  });
}
```

### UI Components

#### Documents Panel

Located in sidebar between search and tree:

```html
<div id="documentsPanel">
  <h4>📄 Documents (RAG)</h4>
  <label for="docFileInput" class="doc-upload-btn">
    + Upload Document (PDF, DOCX, TXT, MD)
  </label>
  <input type="file" id="docFileInput" accept=".pdf,.docx,.txt,.md,.json,.csv" style="display:none;" />
  <div class="doc-list" id="docList">
    <!-- Document items or empty state -->
  </div>
  <button class="doc-clear-all" id="docClearAllBtn">Clear All Documents</button>
</div>
```

**States**:
- No conversation selected: "Select a conversation first"
- No documents: "No documents uploaded"
- With documents: List with remove buttons + clear all button

#### RAG Status Badge

Appears in chat header when documents are present:

```html
<div id="ragStatusBadge" class="active">
  🟢 RAG Active: 2 docs
</div>
```

**Styling**:
- Orange badge (`--orange-light` background)
- Auto-shown/hidden based on document count
- Updates when documents change

#### Document List Items

```html
<div class="doc-item">
  <span class="doc-item-icon">📄</span>
  <span class="doc-item-name" title="document.pdf">document.pdf</span>
  <button class="doc-item-remove" onclick="handleDocumentRemove('doc-id')">×</button>
</div>
```

### Event Handlers

```javascript
// Document upload
document.getElementById('docFileInput').addEventListener('change', handleDocumentUpload);

// Document clear all
document.getElementById('docClearAllBtn').addEventListener('click', handleDocumentsClearAll);

// Individual document remove (inline onclick)
async function handleDocumentRemove(docId) {
  if (!confirm('Remove this document from the conversation?')) return;
  await deleteDocument(docId);
  await renderDocumentsList();
}
```

### State Management

**state.activeDocuments**:
- Array of document objects for current conversation
- Updated when switching conversations (setActiveNode)
- Updated after upload/delete operations
- Used for RAG badge rendering

**Lifecycle**:
```javascript
// On conversation switch
async function setActiveNode(nodeId) {
  // ... existing code ...
  state.activeConvoId = node.convoId;
  await renderDocumentsList();  // Updates state.activeDocuments
  updateRagStatus();
}

// On document upload
async function handleDocumentUpload(event) {
  await uploadDocument(state.activeConvoId, file);
  await renderDocumentsList();  // Refreshes UI and state
}
```

### RAG + Branching Integration

**Context Inheritance**:
- Documents linked to conversation, not individual nodes
- All branches (trunk, active branches, forks) share same documents
- When forking, branches automatically get RAG context
- When promoting/splitting, new conversation does NOT inherit documents (fresh slate)

**Use Cases**:
1. **Explore document questions**: Fork to try different questions on same content
2. **Compare interpretations**: Multiple branches with different prompts on same docs
3. **Refine understanding**: Commit valuable insights to trunk while exploring tangents
4. **Document-specific tangents**: Split to new conversation for unrelated follow-ups

### Performance Considerations

**Upload Performance**:
- PDF parsing: ~50-200ms per page (depends on content)
- DOCX parsing: ~100-500ms per document
- Text files: Near-instant

**File Size Limits**:
- **Raw file upload**: 10MB maximum
- **Extracted text content**: 5MB maximum
- Rationale:
  - Prevents IndexedDB quota issues (typically 50MB-500MB available)
  - Ensures LLM token limits not exceeded (~128K-200K tokens = ~500KB-800KB)
  - Maintains responsive parsing and UI performance
  - Large documents should be split for better RAG accuracy anyway

**Context Size Impact**:
- Documents injected as system message (counted in context)
- 5MB text limit ≈ 1.25M tokens (well below most model limits)
- Context counter includes document size
- Very large knowledge bases may approach model context windows

**Storage**:
- Plain text storage is space-efficient
- IndexedDB typically 50MB+ limit per origin
- With 5MB per doc limit: ~10-100 documents feasible (depending on browser)
- Recommend exporting conversations periodically as backup

### Security Considerations

**Client-Side Only**:
- All parsing happens in browser
- No document data sent to servers (except LLM via LM Studio)
- Documents stored in browser IndexedDB only

**File Validation**:
- Extension-based type checking
- No server-side validation needed
- Malicious file risk limited to client-side parsing libraries

**Data Privacy**:
- Documents never leave user's machine (except LM Studio)
- No cloud storage or external API calls
- User controls all data lifecycle

### Future Enhancements

Potential improvements:
- **Chunking**: Split large documents for better context management
- **Document summaries**: Pre-generate summaries for context efficiency
- **Vector search**: Semantic retrieval instead of full-text injection
- **File viewers**: Preview documents before upload
- **OCR support**: Extract text from scanned PDFs
- **Export with documents**: Include document references in MD export
- **Document versioning**: Track document history and changes

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

**For Branch Messages (v1.5 - Exact Context Preservation)**:
- Complete parent conversation context (exact message copies from snapshot)
- Branch-specific messages
- No summarization - perfect semantic fidelity
- Context reconstruction: `[...snapshotMessages, ...branchMessages]`
- Prevents context pollution between branches while maintaining full transparency

**v1.5 Changes**:
- Eliminated LLM-generated summaries during fork creation
- Snapshots now store complete message arrays instead of summary strings
- Faster branch creation (no extra LLM call required)
- Better conversation continuity and predictable LLM behavior
- Context size indicator displays real-time message count with efficiency color coding

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

**Context Snapshot (v1.5)**  
Exact copy of [N] parent messages

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
**Maintainer**: ORAND Praxis Project
