# Security Policy

## Supported Versions

Currently supported version of ORAND Conversation Brancher:

| Version | Supported          |
| ------- | ------------------ |
| 1.2     | :white_check_mark: |
| < 1.2   | :x:                |

## Security Considerations

### Data Privacy

ORAND Conversation Brancher is designed with privacy in mind:

- **Local Storage Only**: All conversation data is stored locally in your browser's IndexedDB
- **No External Servers**: No data is sent to external servers (except your configured LM Studio instance)
- **No Analytics**: No tracking or analytics code is included
- **No Cookies**: The application doesn't use cookies
- **Client-Side Only**: Everything runs in your browser - no backend server

### LM Studio Connection

When using LM Studio integration:

- Connections are made to `localhost` by default
- All communication is local (unless you configure a remote LM Studio instance)
- Be cautious when connecting to remote LM Studio instances
- Use HTTPS if connecting to remote instances
- Verify the endpoint before sending sensitive information

### Browser Security

The application relies on browser security features:

- **Same-Origin Policy**: Protects against unauthorized access
- **IndexedDB Security**: Data is isolated per origin
- **Content Security**: No eval() or inline scripts (best practices)

### Data Loss Prevention

- Regular browser cache clearing may delete your conversations
- Export important conversations to Markdown as backups
- Browser's private/incognito mode won't persist data after closure

## Reporting a Vulnerability

If you discover a security vulnerability in ORAND Conversation Brancher, please follow these steps:

### How to Report

1. **Do NOT** open a public issue for security vulnerabilities
2. Email the maintainers directly (check repository for contact)
3. Alternatively, use GitHub's private security advisory feature

### What to Include

Please provide:

- Description of the vulnerability
- Steps to reproduce
- Potential impact
- Suggested fix (if you have one)
- Your contact information (optional)

### Response Timeline

- **Initial Response**: Within 48 hours
- **Status Update**: Within 7 days
- **Fix Timeline**: Depends on severity
  - Critical: 1-7 days
  - High: 7-30 days
  - Medium: 30-90 days
  - Low: Best effort

### Disclosure Policy

- We will acknowledge your report
- We will investigate and develop a fix
- We will coordinate disclosure timing with you
- You will be credited (unless you prefer anonymity)

## Best Practices for Users

### Protecting Your Data

1. **Export Regularly**: Save important conversations as Markdown files
2. **Secure Your Device**: Use device encryption and strong passwords
3. **Update Browser**: Keep your browser updated for security patches
4. **Verify Source**: Only download the application from trusted sources
5. **Review Code**: The application is open source - inspect it if needed

### LM Studio Configuration

1. **Use Local Connections**: Prefer `localhost` over remote connections
2. **Firewall Rules**: Ensure LM Studio isn't exposed to the internet unnecessarily
3. **CORS Settings**: Configure CORS properly to prevent unauthorized access
4. **Model Selection**: Be aware of the models you're using and their capabilities

### Sensitive Information

- **Don't share**: Avoid including passwords, API keys, or personal information in conversations
- **Local Only**: Remember that data is stored in browser storage
- **Shared Devices**: Be cautious when using on shared or public computers
- **Clear Data**: Use browser's clear data feature when done on shared devices

## Known Limitations

- Data is not encrypted at rest in IndexedDB
- No built-in authentication (client-side only application)
- Relies on browser security mechanisms
- No audit logging of access (only conversation actions)

## Security Updates

Security updates will be:
- Released as soon as possible after verification
- Announced in the CHANGELOG
- Tagged with version increments
- Documented in release notes

## Questions?

For non-sensitive security questions, open an issue with the "security" label.

---

**Last Updated**: 2026-03-22
