# Changelog

All notable changes to ORAND Praxis will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
