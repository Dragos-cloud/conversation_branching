# Changelog

All notable changes to ORAND Praxis will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2.0.0] - 2026-03-24

### 🎉 Major Feature: RAG Document Integration

### Added
- **Document Upload & Processing**: Upload documents directly to conversations for document-grounded discussions
  - Supports PDF (via PDF.js), DOCX (via Mammoth.js), TXT, MD, JSON, CSV formats
  - Automatic document parsing and text extraction
  - Documents stored per conversation in IndexedDB
- **RAG Context Injection**: Documents automatically injected as system prompts for all LLM requests
  - Knowledge base formatted with document titles and content
  - Clear instructions for LLM to prioritize document information
  - Works with both trunk messages and branch forks
  - RAG context inherited by all fork branches from same conversation
- **Documents Panel**: New sidebar section for document management
  - Upload button with file type validation
  - Document list showing all uploaded files for active conversation
  - Individual document removal (×) button
  - Clear all documents button
  - Empty state guidance ("Select a conversation first")
- **RAG Status Indicators**: Visual feedback showing when RAG is active
  - Orange RAG badge in chat header: "🟢 RAG Active: N docs"
  - Badge hidden when no documents present
  - Document count updates in real-time
- **CDN Dependencies**: Added PDF.js (v3.11.174) and Mammoth.js (v1.6.0) for document processing

### Changed
- **Database Schema**: Upgraded to v3 with new `documents` object store
  - Schema: `{id, convoId, filename, content, format, uploadedAt}`
  - Indexes on `convoId` and `uploadedAt` for efficient queries
  - Automatic migration from v2 → v3 for existing users
- **State Management**: Added `activeDocuments` array to track documents for current conversation
- **Clear All Data**: Now includes documents when clearing all conversations
- **App Description**: Updated to include "RAG document integration"

### Technical
- **Database Functions**: Added `getConversationDocuments()`, `uploadDocument()`, `deleteDocument()`, `clearAllDocuments()`
- **Document Parsing**: `parseDocumentFile()` handles all supported formats with appropriate library
- **UI Rendering**: `renderDocumentsList()` updates document panel when switching conversations
- **RAG Prompt Construction**: System message with knowledge base injected before all LLM API calls
- **Event Handlers**: Document upload input and clear all button integrated into event binding
- **setActiveNode Enhancement**: Now updates `state.activeConvoId` and calls `renderDocumentsList()`
- **File Size Limits**: 10MB raw upload limit, 5MB extracted text limit to prevent quota issues and ensure optimal LLM performance

### User Benefits
- **Document-Grounded Conversations**: Ask questions about uploaded documents with LLM responses based on content
- **Branching + RAG**: Fork conversations to explore different questions on same documents
- **Context Preservation**: Documents linked to conversations, not global - prevents cross-contamination
- **Complete Integration**: RAG works seamlessly with all existing features (commit, promote, split, export)
- **Audit Trail**: Document references preserved in conversation snapshots and exports

### Breaking Changes
None - fully backward compatible. Existing conversations work unchanged; new feature is opt-in per conversation.

## [1.7.1] - 2026-03-24

### Fixed
- **Read-Only UI Consistency**: Split, Promoted, and Committed branches now have consistent read-only UI like Discarded branches
  - All resolved branches (discarded, committed, promoted, split) now show "🔒 READ-ONLY" badge with gray styling
  - All action buttons (Commit, Discard, Promote, Split) are hidden for resolved branches
  - Input field, send button, and fork button are disabled when viewing resolved branches
  - Maintains complete audit trail visibility while preventing accidental modifications

## [1.7.0] - 2026-03-24

### 🔥 Breaking Changes
- **Promote Action Behavior**: Promote now creates a new conversation (like Split) instead of modifying the original conversation
  - Old behavior: Replaced trunk messages with branch messages, stayed in same conversation
  - New behavior: Creates new conversation with full trunk history + branch direction, original remains intact
  - User benefit: Original conversations are preserved, complete audit trail maintained

### Added
- **Full Context Preservation for Promote**: Promoted branches create new conversations with complete original context
  - All trunk messages up to fork point are copied to new conversation
  - Branch messages are appended to show new direction
  - Original conversation remains completely unchanged
- **promotedToConvoId Field**: New database field tracks target conversation ID (parallel to splitToConvoId)
- **Badge Shows Target**: "⬆ PROMOTED → #xxxx" badge displays target conversation ID (last 4 chars)
- **Chat Header Text**: Promoted resolution shows "Resolved: Promoted to New Conversation → Conv #xxxx"
- **Export Metadata**: Added "Promoted to | Conversation `xxxx-xxxx-xxxx`" row in node metadata table

### Changed
- **Promote Dialog**: Updated text to explain "This branch becomes a new conversation with full original context"
- **Context Semantics**: 
  - Promote = Full original history + new direction (answers: "Should conversation go this way?")
  - Split = Fork point only + tangent (answers: "Should this be separate conversation?")
- **Symmetry**: Both Promote and Split now create new conversations, leave audit trail, show target IDs

### Technical
- Rewrote `promoteBranch()` function to use `createConversation()` instead of replacing trunk messages
- Both Promote and Split now preserve original conversation structure for complete audit trail
- Context inheritance models differentiated: Promote gets full trunk, Split gets fork snapshot only

## [1.6.3] - 2026-03-24

### Added
- **Split Branch Visibility**: Split branches now remain visible in the tree with special ⎇ indicator
- Purple/green gradient background styling for split branches to distinguish from active branches
- ⎇ arrow suffix in split branch labels for at-a-glance recognition
- "⎇ SPLIT → #xxxx" badge showing target conversation ID (last 4 chars)
- CSS styling: `.tree-node.split` with gradient background and purple text

### Changed
- **Tree Visibility Filter**: All branch statuses now visible (active, committed, discarded, promoted, split)
- **Split Dialog**: Changed text from "hidden from this tree" to "remain visible in this tree for audit purposes"
- **Chat Header Text**: Split resolution now shows target conversation ID: "Resolved: Split to New Conversation → Conv #xxxx"
- **User Guide**: Updated status legend to reflect split branch visibility with ⎇ indicator

### Technical
- Modified `visibleBranches` filter to return all branches: `return bn;` (no exclusions)
- Extended branch rendering logic to handle `isSplit` state with special badge and styling
- Split branches already preserved in database (v1.6.3 only adds visibility)

### User Benefits
- **Complete Audit Trail**: All four resolution types visible (committed, discarded, promoted, split)
- **Decision Transparency**: See which branches were split off into new conversations
- **Navigation**: Click split branch to review content before it became independent
- **Traceability**: Branch badge shows target conversation ID for easy cross-reference

## [1.6.2] - 2026-03-24

### Added
- **Promoted Branch Visibility**: Promoted branches now remain visible in the tree with special ⬆ indicator
- Blue gradient background styling for promoted branches to distinguish from active branches
- ↑ arrow suffix in promoted branch labels for at-a-glance recognition
- "⬆ PROMOTED" badge with blue highlighting to match visual hierarchy
- CSS styling: `.tree-node.promoted` with gradient background and bold blue text

### Changed
- **Tree Visibility Filter**: Promoted branches now visible alongside active, committed, and discarded branches (only split branches hidden)
- **Chat Header Text**: Updated from "Resolved: Promote to Main" to "Resolved: Promoted to Main (this content now on trunk)"
- **User Guide**: Updated lifecycle and export sections to reflect promoted branch visibility
- **Status Legend**: Changed Promoted status from "hidden" to "visible in tree with ↑ indicator"

### Technical
- Modified `visibleBranches` filter to check `status !== 'split'` (removed `!== 'promoted'` condition)
- Extended branch rendering logic to handle `isPromoted` state with special badge and styling
- Fork entries preserved after promotion (retained from v1.6.1)

### User Benefits
- **Complete Audit Trail**: All branches visible regardless of resolution status (active, committed, discarded, promoted)
- **Decision History**: Clear visual indication of which branch "won" promotion at each fork point
- **Navigation**: Click promoted branch to review its complete history before it became trunk
- **Consistency**: Matches pattern of keeping discarded branches visible for transparency
- **Educational**: New users can see promotion outcomes and understand the workflow

## [1.6] - 2026-03-24

### Added
- **Selective Message Commit UI**: Interactive modal for choosing which branch messages to inject into trunk
- Checkbox-based message selection interface with role badges and content previews
- "Select All/Deselect All" toggle button for quick selection management
- Real-time selection counter showing how many messages are selected
- Visual selection indicators (highlighted background, border color change)
- Message preview truncation (200 characters) with full content in scrollable container

### Changed
- **Commit workflow**: Replaced automatic LLM summarization with user-controlled message selection
- Commit action now injects exact selected messages into trunk instead of generating summary
- Branch commit header shows count: "📌 Committed X messages from branch 'Name':"
- Action log now records detailed commit metadata: selected count, total count, message indices

### Technical
- Added commit modal HTML structure with message list container
- Added CSS for `.commit-modal-box`, `.commit-message-item`, checkbox styling, role badges
- Created `showCommitModal(branchLabel, messages)` function returning Promise<selectedMessages>
- Modified `commitBranch()` to use modal instead of confirm dialog
- Removed `generateSummary()` call from commit workflow (already removed in v1.5 for forking)
- Proper event listener cleanup for modal buttons including Select All handler

### User Benefits
- Cherry-pick specific insights or responses from branches
- Avoid committing hallucinations or off-track explorations
- Maintain narrative control over main conversation thread
- Full transparency - see exactly what's being merged
- Faster commits (no LLM summary generation)

## [1.5] - 2026-03-24

### Added
- **Exact Context Preservation**: Branches now inherit complete parent conversation history with zero information loss
- **Context Size Indicator**: Real-time message count display with color-coded efficiency metrics
  - Green (< 20 messages): Optimal context size
  - Yellow (20-39 messages): Good context size
  - Orange (40-59 messages): Large context - forking recommended
  - Red (≥ 60 messages): Very large context - forking strongly recommended
- Context comparison tooltip for branches showing parent vs. current context size
- Context counter updates dynamically as messages are added

### Changed
- **Snapshot architecture**: Replaced LLM-generated summaries with exact message copies
- Snapshot schema updated: `summary` string replaced with `messages` array containing full message objects
- Context injection now uses exact message history instead of summarized system notes
- Faster branch creation (eliminates extra LLM call for summary generation)
- Better semantic fidelity and conversation continuity across branches

### Technical
- Modified `launchBranches()`: Removed `generateSummary()` call, implemented deep copy of messages
- Updated `sendMessage()`: Reconstructs context from `[...snapshotMessages, ...branchMessages]`
- Added `updateContextCounter()` function for real-time context size display
- Added CSS for context counter badges (`.ctx-green`, `.ctx-yellow`, `.ctx-orange`, `.ctx-red`)
- Added context tooltip with hover interaction
- Updated snapshot creation to store `messages: [{role, content, createdAt}]`
- Marked `SUMMARY_MAX_TOKENS` config as deprecated

### Performance
- Eliminated one LLM API call per fork operation (no summary generation needed)
- Reduced fork creation latency by ~2-5 seconds depending on model speed

## [1.4] - 2026-03-24

### Added
- **Discarded Branch Visibility**: Discarded branches now visible in conversation tree for audit trail
- Read-only mode for discarded branches (cannot send messages or fork from discarded branches)
- Visual distinction for discarded branches (dimmed styling with strikethrough)
- Enhanced audit transparency - all conversation paths remain visible

### Changed
- Tree rendering now includes discarded branches with special styling
- Branch selection enforces read-only state for discarded branches
- Updated user interface messaging for discarded branch state

### Technical
- Added `.tree-node.discarded` CSS class for visual distinction
- Modified `renderTree()` to display discarded branches
- Added read-only enforcement in `renderChat()` for discarded branch state
- Updated branch filtering logic to show all lifecycle states except 'promoted' and 'split'

## [1.3] - 2026-03-22

### Added
- **Full-text search functionality** across all conversations, branches, and messages
- Search input with real-time results in sidebar
- Debounced search (300ms) for performance
- Search result highlighting with contextual snippets
- Click-to-navigate search results
- Clear search button with visual feedback
- Search across conversation titles, branch/node labels, and message content
- Result relevance sorting (conversation matches first, then by date)
- Limited to 50 most relevant results for performance
- Message snippets with ~60 characters of context before/after matches

### Changed
- Enhanced sidebar UI to accommodate search interface
- Improved user guide with search documentation
- Updated version metadata and changelog

### Technical
- Added `performSearch()` function with IndexedDB querying
- Added `renderSearchResults()` for dynamic result display
- Added `highlightSearchTerm()` for match highlighting
- Added `clearSearch()` for search state reset
- Search state management (searchQuery, searchResults)
- Event binding with debounce for search input
- CSS styling for search container, results, and highlights

## [1.2] - 2026-03-22

### Added
- Non-linear conversation branching system
- Fork management for exploring multiple conversation paths
- Soft-delete audit trail for tracking conversation history
- Markdown export functionality for entire conversation trees
- Visual tree view with color-coded branches
- LM Studio integration for local LLM inference
- IndexedDB storage for persistent local data
- Branch operations: commit, discard, promote, split
- Configurable LLM parameters (temperature, top_p, max_tokens)
- Auto-resize text input
- Confirm dialogs for destructive operations
- Built-in user guide tab

### Features
- Single-file HTML application (no dependencies)
- Real-time conversation tree visualization
- Multiple branch management
- Parallel branch execution
- Conversation metadata tracking
- Browser-based storage (privacy-focused)

### Technical
- Modular code architecture with clear sections
- CSS custom properties for theming
- Responsive design for mobile and desktop
- Error handling and user feedback
- Performance optimizations for large conversation trees

## [Unreleased]

### Planned
- Export to other formats (JSON, TXT)
- Import existing conversations
- Custom branch colors
- Keyboard shortcuts
- Dark mode theme
- Multi-model comparison
- Conversation templates

---

## Version History Format

### Added
New features added to the project.

### Changed
Changes to existing functionality.

### Deprecated
Features that will be removed in upcoming releases.

### Removed
Features that have been removed.

### Fixed
Bug fixes.

### Security
Security patches and improvements.
