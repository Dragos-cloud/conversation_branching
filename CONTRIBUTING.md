# Contributing to ORAND Conversation Brancher

Thank you for your interest in contributing to ORAND Conversation Brancher! This document provides guidelines for contributing to the project.

## Code of Conduct

By participating in this project, you agree to maintain a respectful and inclusive environment for all contributors.

## How Can I Contribute?

### Reporting Bugs

Before creating bug reports, please check existing issues to avoid duplicates. When creating a bug report, include:

- **Clear title and description**
- **Steps to reproduce** the issue
- **Expected behavior** vs **actual behavior**
- **Screenshots** if applicable
- **Browser version** and operating system
- **LM Studio version** if relevant

### Suggesting Enhancements

Enhancement suggestions are welcome! Please provide:

- **Clear use case** for the enhancement
- **Detailed description** of the proposed functionality
- **Example scenarios** where it would be useful
- **Potential implementation approach** if you have ideas

### Pull Requests

1. **Fork the repository** and create your branch from `main`
2. **Make your changes** in the HTML file
3. **Test thoroughly** in multiple browsers
4. **Update documentation** if needed (especially the USERGUIDE section)
5. **Follow the coding style** (see below)
6. **Submit a pull request** with a clear description

## Development Guidelines

### Code Structure

The application is organized into clear sections (marked with `SECTION:` comments):

- **META**: Version and metadata
- **CONFIG**: Configuration constants
- **STYLE**: CSS styling
- **STATE**: Application state
- **UI**: DOM rendering functions
- **LOGIC**: Core functionality modules
- **EVENTS**: Event handlers
- **INIT**: Initialization
- **USERGUIDE**: Built-in documentation

When adding new features, place code in the appropriate section.

### Coding Style

#### JavaScript
- Use **modern ES6+ syntax**
- Prefer `const` and `let` over `var`
- Use **meaningful variable names**
- Add **comments** for complex logic
- Use **async/await** for asynchronous code
- Keep functions **focused and small**

#### CSS
- Use **CSS custom properties** (variables) for colors and dimensions
- Follow the existing **naming conventions**
- Maintain **responsive design** principles
- Test on multiple screen sizes

#### HTML
- Keep the structure **semantic**
- Use **appropriate ARIA labels** for accessibility
- Maintain **indentation consistency**

### Testing

Before submitting a pull request, test your changes:

1. **Multiple browsers**: Chrome, Firefox, Safari, Edge
2. **Different screen sizes**: Desktop, tablet, mobile
3. **Core functionality**:
   - Creating conversations
   - Sending messages
   - Creating branches
   - Forking conversations
   - Exporting to Markdown
   - LM Studio integration
4. **Edge cases**: Empty states, long conversations, multiple branches

### Version Updates

When making changes, update the version in the META section:

```html
<!-- SECTION:META
  App: ORAND Conversation Brancher
  Version: X.Y
  ...
  Last-modified: YYYY-MM-DD
-->
```

Follow semantic versioning:
- **Major (X.0)**: Breaking changes
- **Minor (X.Y)**: New features, backward compatible
- **Patch (X.Y.Z)**: Bug fixes (if needed)

### Database Schema Changes

If modifying IndexedDB schema:
1. **Increment the database version** in the `DB_VERSION` constant
2. **Add migration logic** in the `onupgradeneeded` handler
3. **Test with existing data** to ensure no data loss
4. **Document the changes** in commit message

## Feature Development Checklist

- [ ] Code follows existing style and structure
- [ ] Tested in Chrome, Firefox, and Safari
- [ ] Tested on mobile/tablet screen sizes
- [ ] No console errors or warnings
- [ ] Documentation updated (if applicable)
- [ ] Version number updated
- [ ] Migration path for existing users (if DB changes)
- [ ] Performance tested with large conversation trees

## Architecture Notes

### State Management
- State is stored in the `state` object
- Use `setState()` for updates that trigger UI refresh
- Persist critical data to IndexedDB

### IndexedDB Usage
- All database operations are asynchronous
- Use transactions appropriately
- Handle errors gracefully
- Consider offline functionality

### LM Studio Integration
- API calls should handle network failures
- Support different model configurations
- Provide clear error messages to users

## Need Help?

- **Questions**: Open an issue with the "question" label
- **Discussion**: Use GitHub Discussions (if enabled)
- **Documentation**: Check the built-in User Guide tab

## Recognition

All contributors will be recognized in the project. Your contributions help make this tool better for everyone!

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

Thank you for contributing to ORAND Conversation Brancher! 🎉
