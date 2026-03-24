# ORAND Praxis

## Overview

ORAND Praxis is a single-file HTML application that enables branching conversations with Large Language Models (LLMs). Unlike traditional linear chat interfaces, this tool allows you to explore multiple conversation paths, fork responses, and manage different branches of dialogue.

## Features

- **🌳 Branching Conversations**: Create multiple conversation branches from any point
- **🔀 Fork Management**: Split conversations into multiple parallel branches to explore different responses
- **🎯 Exact Context Preservation**: Branches inherit complete message history with zero information loss (v1.5)
- **✨ Selective Message Commit**: Cherry-pick which messages to merge from branches to trunk with interactive UI (v1.6)
- **� RAG Document Integration**: Upload documents (PDF, DOCX, TXT, MD, JSON, CSV) for document-grounded conversations (v2.0)
- **�📊 Context Size Indicator**: Real-time message count with color-coded efficiency metrics (v1.5)
- **🔍 Full-Text Search**: Search across all conversations, branches, and messages with instant results
- **📝 Audit Trail**: Soft-delete system maintains conversation history with read-only visibility for all resolved branches: discarded, promoted, and split (v1.4, v1.6.2, v1.6.3, v1.7)
- **💾 Local Storage**: All data stored locally in IndexedDB - no server required
- **📤 Markdown Export**: Export entire conversation trees to Markdown format
- **🔄 LM Studio Integration**: Connect to local LM Studio instance for LLM inference
- **📈 Visual Tree View**: Intuitive visual representation of all conversation branches including discarded ones
- **🎨 Color-Coded Branches**: Easy-to-distinguish branch colors for better organization
- **⚡ Single File**: Entire application in one HTML file - no dependencies or build process

## Getting Started

### Prerequisites

- A modern web browser (Chrome, Firefox, Edge, or Safari)
- [LM Studio](https://lmstudio.ai/) (optional, for local LLM inference)

### Installation

1. Download `orand_praxis_v1.3.html` (or latest version)
2. Open the file in your web browser
3. That's it! The app is ready to use.

### Configuration

#### Using LM Studio

1. Install and launch [LM Studio](https://lmstudio.ai/)
2. Load a model in LM Studio
3. Enable the local server in LM Studio (typically runs on `http://localhost:1234`)
4. In the Conversation Brancher, the app will automatically connect to LM Studio
5. Select your model from the dropdown and start chatting

#### Settings

- **LM Studio URL**: Default is `http://localhost:1234` (configurable)
- **Max Tokens**: Maximum response length (default: 2048)
- **Temperature**: Controls randomness in responses (0.0 - 2.0)
- **Top P**: Nucleus sampling parameter

## Usage

### Basic Chat

1. Type your message in the input box at the bottom
2. Press `Ctrl+Enter` or click the Send button
3. Wait for the LLM to respond

### Creating Branches

1. **Add Fork**: Click the "Add Fork" button to create multiple branches from the last assistant response
2. **Edit Prompts**: Modify the prompt variations for each branch
3. **Launch Branches**: Submit to generate responses for all branches simultaneously
4. **Navigate**: Use the branch selector to switch between different conversation paths

### Managing Branches

- **Commit**: Merge a branch into the trunk (main conversation)
- **Discard**: Soft-delete a branch (can be viewed in audit log)
- **Promote**: Create a new conversation with full original context plus this branch's direction (v1.7)
- **Split**: Create a new independent conversation tree from the fork point only

### Selective Commit (v1.6)

When committing a branch, you can now choose exactly which messages to merge into the main conversation:

1. Click "Commit" on any branch
2. Review all branch messages in an interactive modal
3. **Select/deselect messages** using checkboxes (all selected by default)
4. Use "Select All" to quickly toggle all messages
5. Click "Commit Selected" to inject chosen messages into trunk

**Benefits:**
- **Cherry-pick insights**: Keep only the valuable parts of an exploration
- **Filter hallucinations**: Exclude incorrect or off-track responses
- **Narrative control**: Maintain a coherent main conversation thread
- **Full transparency**: See exactly what you're merging before committing

The modal shows each message with its role (user/assistant), content preview, and selection state. All selections are recorded in the action log for complete audit trail.

### Context Management (v1.5)

**Exact Context Preservation**: When you create branches, ORAND Praxis now preserves the complete conversation history with zero information loss. Unlike previous versions that used LLM-generated summaries, v1.5 stores exact copies of all messages, ensuring:
- Perfect semantic fidelity - no summarization artifacts
- Faster branch creation - eliminates extra LLM calls
- Better conversation continuity across branches
- More predictable LLM behavior

**Context Size Indicator**: The input area displays a real-time context counter showing how many messages are in the current conversation context. The indicator uses color coding for efficiency guidance:
- 🟢 **Green** (< 20 messages): Optimal context size
- 🟡 **Yellow** (20-39 messages): Good context size  
- 🟠 **Orange** (40-59 messages): Large context - consider forking
- 🔴 **Red** (≥ 60 messages): Very large context - forking recommended

For branches, hover over the counter to see a comparison with the parent branch's context size.

### Document Upload & RAG (v2.0)

**RAG (Retrieval-Augmented Generation)** allows you to ground LLM responses in your uploaded documents.

#### Uploading Documents

1. **Select a conversation** or create a new one
2. In the sidebar, find the **📄 Documents (RAG)** panel
3. Click **"+ Upload Document"** button
4. Select a file (PDF, DOCX, TXT, MD, JSON, or CSV)
5. Document is parsed and stored with the conversation

#### Using RAG

- Once documents are uploaded, the **🟢 RAG Active** badge appears in the chat header
- All messages sent in that conversation will have access to the document content
- The LLM receives documents as part of its system prompt with instructions to prioritize document information
- Ask questions about the documents: "What is the main conclusion?", "Summarize section 3", etc.

#### RAG + Branching

 RAG works seamlessly with all branching features:
- **Fork with documents**: Create branches to explore different questions on the same documents
- **Compare approaches**: Branch to compare RAG vs non-RAG responses (upload docs to one branch only)
- **Commit insights**: Merge valuable document-based insights back to trunk
- **Promote/Split**: Documents stay linked to conversation - promoted/split branches inherit document context

#### Managing Documents

- **View uploaded documents**: See list in sidebar with file names
- **Remove individual document**: Click × button next to file name
- **Clear all documents**: Click "Clear All Documents" button to remove all files from conversation
- **Per-conversation storage**: Documents are linked to specific conversations, not global

#### Supported Formats

- **PDF**: Full text extraction from all pages (via PDF.js)
- **DOCX**: Microsoft Word document text extraction (via Mammoth.js)
- **TXT**: Plain text files
- **MD**: Markdown files
- **JSON**: JSON data files
- **CSV**: Comma-separated value files

#### File Size Limits

- **Maximum file size**: 10MB (raw upload)
- **Maximum extracted text**: 5MB (after parsing)
- These limits prevent browser storage quota issues and ensure optimal LLM performance
- Large documents should be split into smaller, focused files for better RAG results

**Note**: RAG context is injected automatically - you don't need to reference documents explicitly. Just ask questions naturally.

### Exporting

Click the "Export as Markdown" button to download your entire conversation tree as a `.md` file. The export includes:
- All conversation branches
- Timestamps and metadata
- Hierarchical structure preserved

### Search

Use the search bar at the top of the sidebar to find content across all conversations:
- Type at least 2 characters to see results
- Searches conversation titles, branch labels, and message content
- Results display contextual snippets with highlighted matches
- Click any result to jump directly to that conversation/branch/message
- Results are sorted by relevance (titles first, then by date)
- Limited to 50 most relevant results

## Data Storage

All conversation data is stored locally in your browser's IndexedDB. This means:
- ✅ Complete privacy - no data sent to external servers (except LM Studio if configured)
- ✅ Works offline (after initial page load)
- ✅ Persistent across browser sessions
- ⚠️ Data is browser-specific (not synced across devices)
- ⚠️ Clearing browser data will delete conversations

## Architecture

The application is structured in clear sections:
- **META**: App metadata and version info
- **CONFIG**: Configuration constants
- **STYLE**: CSS styling with design tokens
- **STATE**: Application state management
- **UI**: User interface rendering functions
- **LOGIC**: Core functionality modules
  - `db`: IndexedDB operations (6 stores: conversations, nodes, messages, snapshots, actionLog, documents)
  - `actionlog`: Audit trail management
  - `lmstudio`: LM Studio API integration
  - `tree`: Conversation tree management
  - `branch`: Branch operations
  - `export`: Markdown export functionality
  - `documents`: RAG document processing (PDF.js, Mammoth.js)
- **EVENTS**: Event handlers
- **INIT**: Application initialization
- **USERGUIDE**: Built-in user documentation

## Browser Compatibility

- Chrome/Edge (v90+): ✅ Full support
- Firefox (v88+): ✅ Full support
- Safari (v14+): ✅ Full support
- Opera (v76+): ✅ Full support

## Limitations

- Requires JavaScript enabled
- IndexedDB must be available and enabled
- LM Studio connection requires CORS to be properly configured
- Large conversation trees may impact performance

## Privacy & Security

- All data stored locally in browser
- No analytics or tracking
- No external API calls (except to your local LM Studio instance)
- Open source - inspect the code yourself

## Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- Built for use with [LM Studio](https://lmstudio.ai/)
- Inspired by the need for non-linear conversation exploration with LLMs

## Version History

- **v2.0.0** (2026-03-24): RAG document integration - upload PDF, DOCX, TXT, MD, JSON, CSV for document-grounded conversations
- **v1.7.1** (2026-03-24): Read-only UI consistency for all resolved branches
- **v1.7.0** (2026-03-24): Promote creates new conversations with full context
- **v1.6.3** (2026-03-24): Split branch visibility with audit trail
- **v1.6.2** (2026-03-24): Promoted branch visibility
- **v1.6.0** (2026-03-24): Selective message commit with interactive UI
- **v1.5.0** (2026-03-23): Exact context preservation, context size indicator
- **v1.2** (2026-03-22): Full branching and export capabilities

## Support

For issues, questions, or suggestions, please open an issue on GitHub.

---

**Made with ❤️ for better LLM conversations**
