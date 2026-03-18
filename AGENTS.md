# AI-Assisted Development Guidelines

This document outlines standards and best practices for maintaining code quality when using AI tools and agents to contribute to the Noctalia Plugins repository.

## Table of Contents

1. [Code Quality Standards](#code-quality-standards)
2. [AI-Generated Code Review Checklist](#ai-generated-code-review-checklist)
3. [QML-Specific Guidelines](#qml-specific-guidelines)
4. [Testing Requirements](#testing-requirements)
5. [Documentation Standards](#documentation-standards)
6. [Performance Considerations](#performance-considerations)
7. [Security Considerations](#security-considerations)
8. [Pull Request Requirements](#pull-request-requirements)

## Code Quality Standards

### General Requirements

- **No Copy-Paste Anti-patterns**: Avoid duplicated code. Extract common functionality into reusable components
- **Naming Conventions**: Use camelCase for variables and functions, PascalCase for classes and components
- **Comments**: Include meaningful comments for complex logic; avoid obvious or redundant comments
- **File Organization**: Keep files organized by feature/plugin with clear separation of concerns

### Complexity Limits

- Component files should focus on a single responsibility
- Property bindings should be simple; complex logic belongs in functions
- Use the `Main.qml` file as a singleton for functionality that needs to be shared

## AI-Generated Code Review Checklist

### ✓ Before Submitting a PR

- [ ] **No Hallucinated Functions**: Verify all functions actually exist in imported libraries
- [ ] **Type Safety**: Ensure all variables have appropriate types (for typed QML)
- [ ] **Property Bindings**: Check for circular bindings or unnecessary re-evaluations
- [ ] **Error Handling**: Verify error cases are handled appropriately
- [ ] **Resource Cleanup**: Ensure proper cleanup of timers, connections, and other resources
- [ ] **Dependencies**: Verify all required imports are included
- [ ] **Deprecated APIs**: Do not use deprecated Qt/QML APIs
- [ ] **No Placeholder Code**: Remove debug statements, TODOs, and placeholder implementations
- [ ] **Licensing**: Ensure code complies with the project license (check existing plugin licenses)

### ✓ Code Logic Verification

- [ ] **Boundary Conditions**: Test edge cases (empty lists, null values, zero/negative numbers)
- [ ] **Signal/Slot Connections**: Verify connections are properly established and disconnected
- [ ] **State Management**: Check that component state is properly managed and updated
- [ ] **Synchronous vs Asynchronous**: Confirm appropriate use of async operations
- [ ] **File Operations**: Verify all file reads/writes have error handling

### ✓ Performance & Resources

- [ ] **Memory Leaks**: Check for objects that won't be garbage collected
- [ ] **Rendering Performance**: Avoid excessive re-renders or expensive calculations in bindings
- [ ] **API Calls**: Batch requests where possible; implement proper rate limiting
- [ ] **Disk I/O**: Minimize blocking disk operations; use asynchronous alternatives

## QML-Specific Guidelines

### Component Structure

```qml
import QtQuick
import QtQuick.Controls
import QtQuick.Layouts

Rectangle {
  id: root

  // Properties first
  required property string title
  property int itemCount: 0

  // Signals
  signal itemClicked(int index)

  // Child components
  ColumnLayout {
    // Layout content
  }

  // Functions
  function handleClick(index) {
    // Implementation
  }
}
```

### Property Binding Best Practices

- ✓ **Good**: `color: isActive ? "blue" : "gray"`
- ✗ **Bad**: `color: { calculateColorBasedOnMultipleConditions() }`
- Keep bindings simple and readable
- Avoid function calls in bindings unless necessary
- Use `Component.onCompleted` for initialization logic

### Translations

Use translations instead of hardcoded labels.

- ✓ **Good**: `pluginApi?.tr("example.key")`
- ✗ **Bad**: `Hardcoded Label`

```qml
NLabel {
  label: pluginApi?.tr("example.label")
}
```

### Components

Preferably use Noctalias own components that can be found [here](https://github.com/noctalia-dev/noctalia-shell/tree/main/Widgets). That way the plugin follows the correct color scheme!

- ✓ **Good**: `NLabel {}`
- ✗ **Bad**: `Text {}`

### Signal Handling

```qml
Connections {
    target: someObject
    function onSomeSignal(arg) {
        // Handle signal
    }
}

// Clean up in Component.onDestruction
Component.onDestruction: {
    someConnection.destroy()
}
```

### Anchoring vs Layouts

- Use `Anchors` for simple, direct positioning
- Use `Layout` components for complex, responsive layouts
- Do not mix anchoring and layout management in the same container

## Testing Requirements

### Manual Testing

- [ ] Test on target device/emulator
- [ ] Verify all interactive elements work correctly
- [ ] Check responsive behavior at different screen sizes
- [ ] Test with both light and dark themes (if applicable)
- [ ] Verify accessibility features work

### Automated Testing (if applicable)

- [ ] Unit tests for business logic
- [ ] Integration tests for component interactions
- [ ] Performance tests for CPU-heavy operations

## Documentation Standards

### Plugin Metadata

Every plugin should include the correct manifest file:

```json
{
  "id": "plugin-name",
  "name": "Human Readable Title",
  "version": "1.0.0",
  "minNoctaliaVersion": "3.6.0",
  "author": "Author Name",
  "license": "MIT",
  "repository": "https://github.com/noctalia-dev/noctalia-plugins",
  "description": "Clear, concise description of what the plugin does",
  "tags": ["tag1", "tag2"],
  "entryPoints": {
    "main": "Main.qml"
  },
  "dependencies": [],
  "metadata": {
    "defaultSettings": {}
  }
}
```

#### id

The id should be the same value as the folder.

#### name

This is a human readable name to be displayed for users to see.

#### version

Start with version 1.0.0. If this plugin already exists, with every update bump the version up a number based on the changes made.

#### minNoctaliaVersion

The minimum version of Noctalia that this plugin can run on. Some features have been implemented in later version, always check that this is correct!

#### author

The name / username of the author.

#### license

The license this specific plugin is licensed under.

#### repository

The repository that this plugin exists in. When submitting a PR for the noctalia-plugins repository always have it to the github url for the noctalia-plugins repository.

#### description

A helpful and concise description about what the plugin does.

#### tags

The available tags you can find in the README.md file of this repository. ONLY use one of those. Use helpful tags that describe this plugin. Use also specific compositor tags if the plugin can only be used for some / one specific compositor.

#### entryPoints

The available entryPoints to this plugin. Look at the specific documentation to check what entryPoints exist!

#### dependencies

This is a list of plugin dependencies. If the plugin depends on another specific plugin.

#### metadata

The metadata of this plugin. Look at the documentation to find out what can be in here!

### Code Comments

- Document **why**, not **what** (code shows the what)
- Explain non-obvious design decisions
- Include parameter descriptions for complex functions
- Add examples for unusual usage patterns

### README Files

Plugin README should include:

- Title
- Brief description
- Features list
- Installation instructions
- Usage examples
- Configuration options
- Known limitations

## Performance Considerations

### Avoid

- Polling-based state checks (use signals/bindings instead)
- Creating objects in high-frequency code paths
- Binding to expensive calculations
- Loading large files synchronously
- Holding references to QML objects without cleanup

### Optimize

- Use lazy loading for heavy components (use the `Loader` component)
- Debounce/throttle frequent updates (use the `Timer` component)
- Cache computed values when appropriate

## Security Considerations

### Input Validation

- [ ] Validate all external inputs (files, URLs, user input)
- [ ] Sanitize strings before use in system calls
- [ ] Implement bounds checking for numeric inputs
- [ ] Verify file paths don't escape intended directories

### Permissions

- [ ] Request only necessary permissions
- [ ] Document what permissions are needed and why
- [ ] Handle permission denial gracefully
- [ ] Don't assume permissions will be granted

### Data Handling

- [ ] Don't hardcode sensitive information
- [ ] Encrypt sensitive data at rest if applicable
- [ ] Clear sensitive data from memory when done
- [ ] Log appropriately without exposing sensitive data

## Pull Request Requirements

### PR Title and Description

- Title should be concise and descriptive: `feat(plugin_name): add new feature` or `fix(plugin_name): resolve issue with xyz`
- Description must include:
  - What was changed and why
  - How to test the changes
  - Any breaking changes
  - References to related issues (e.g., "Closes #123")

### PR Size

- Keep PRs focused and reasonably sized (no 10000 line PR)
- Large changes should be split into logical commits
- Each commit should be self-contained and functional

### Commit Message Format

```
type(scope): subject

body

footer
```

- **type**: feat, fix, docs, style, refactor, perf, test, chore
- **scope**: plugin name or area affected
- **subject**: lowercase, no period, imperative mood
- **body**: explain the change in detail if needed
- **footer**: reference issues, breaking changes

### AI Disclosure

If substantial portions of code were AI-generated, include in the PR description:

```markdown
## AI-Assisted Development Note

This PR includes code generated with AI assistance using [Tool Name].
All generated code has been reviewed and tested according to our guidelines.

- Generated components: [list]
- Manual modifications: [brief summary]
```

## Review Process for AI-Generated PRs

Reviewers should:

1. **Verify Functionality**: Test the implementation thoroughly
2. **Check for Hallucinations**: Verify all API calls and library references
3. **Performance Review**: Look for inefficiencies typical of AI code
4. **Code Style**: Ensure consistency with existing code
5. **Documentation**: Verify documentation is complete and accurate
6. **Test Coverage**: Ensure adequate testing (both manual and automated)

## Resources

- [Qt Quick Controls Documentation](https://doc.qt.io/qt-6/qtquick-controls-index.html)
- [QML Best Practices](https://doc.qt.io/qt-6/qml-best-practices.html)
- [Noctalia Plugins Project Guidelines](./README.md)
- [Noctalia Development Guidelines](https://docs.noctalia.dev/development/guideline/)
- [Noctalia Plugins Documentation](https://docs.noctalia.dev/development/plugins/overview/)

## Questions?

If you have questions about these guidelines, please open an issue or discuss with the maintainers.

---

**Last Updated**: 2026-03-18
**Maintained By**: Noctalia Development Team
