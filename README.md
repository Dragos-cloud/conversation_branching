# ORAND Praxis

A non-linear LLM conversation tool with branching, fork management, soft-delete audit trail, and Markdown export capabilities.

## Overview

ORAND Praxis is a single-file HTML application that enables branching conversations with Large Language Models (LLMs). Unlike traditional linear chat interfaces, this tool allows you to explore multiple conversation paths, fork responses, and manage different branches of dialogue.

## Features

- **🌳 Branching Conversations**: Create multiple conversation branches from any point
- **🔀 Fork Management**: Split conversations into multiple parallel branches to explore different responses
- **� Full-Text Search**: Search across all conversations, branches, and messages with instant results
- **�📝 Audit Trail**: Soft-delete system maintains conversation history
- **💾 Local Storage**: All data stored locally in IndexedDB - no server required
- **📤 Markdown Export**: Export entire conversation trees to Markdown format
- **🔄 LM Studio Integration**: Connect to local LM Studio instance for LLM inference
- **📊 Visual Tree View**: Intuitive visual representation of conversation branches
- **🎨 Color-Coded Branches**: Easy-to-distinguish branch colors for better organization
- **⚡ Single File**: Entire application in one HTML file - no dependencies or build process

## Getting Started

### Prerequisites

- A modern web browser (Chrome, Firefox, Edge, or Safari)
- [LM Studio](https://lmstudio.ai/) (optional, for local LLM inference)

### Installation

1. Download `orand_conversation_brancher_v1.3.html` (or latest version)
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
- **Promote**: Make a branch the new trunk
- **Split**: Create a new independent conversation tree from a branch

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
  - `db`: IndexedDB operations
  - `actionlog`: Audit trail management
  - `lmstudio`: LM Studio API integration
  - `tree`: Conversation tree management
  - `branch`: Branch operations
  - `export`: Markdown export functionality
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

- **v1.2** (2026-03-22): Current version with full branching and export capabilities

## Support

For issues, questions, or suggestions, please open an issue on GitHub.

---

**Made with ❤️ for better LLM conversations**
