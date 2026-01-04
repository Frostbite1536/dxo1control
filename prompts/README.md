# Project Prompts

Reusable prompts for working with the dxo1control codebase. These prompts provide context and guidelines for LLM-assisted development.

---

## Available Prompts

### Core Prompts

- **[engineering.md](./engineering.md)** - Default prompt for feature development and code changes
  - Use for: Adding features, fixing bugs, refactoring code
  - Includes: Architecture principles, critical invariants, coding standards

### Future Prompts (Planned)

- **bug-hunt.md** - Systematic bug discovery and analysis
- **performance-review.md** - Performance optimization and profiling
- **api-review.md** - API design and compatibility review
- **usb-protocol.md** - USB protocol implementation and debugging
- **image-processing.md** - DNG/image processing optimization

---

## How to Use These Prompts

### For LLM Chat Sessions

1. **Copy the prompt**: Open the relevant `.md` file and copy its contents
2. **Fill in context**: Replace placeholders like `[TASK]` with your specific requirements
3. **Paste into your LLM**: Start your conversation with the customized prompt
4. **Reference documentation**: The LLM will use docs/ files as context

### For Claude Code / AI-Assisted Development

The prompts are designed to work seamlessly with AI coding assistants:

```bash
# Example: Start a feature development session
# The assistant will automatically read engineering.md and relevant docs
```

### Prompt Template

When creating new prompts, use this template:

```markdown
# [Prompt Name]

## Role & Context
[Who the LLM should act as]

## Project Context
[Brief description of dxo1control]

## Current Task
[What needs to be done - filled in by user]

## Key Constraints
[Reference to INVARIANTS.md and ARCHITECTURE.md]

## Process
[Step-by-step approach]

## Success Criteria
[How to know when done]
```

---

## Prompt Guidelines

### ✅ Good Practices

- **Be Specific**: Include exact file paths, function names, or error messages
- **Reference Docs**: Point to ARCHITECTURE.md, INVARIANTS.md for context
- **Define Success**: Clearly state what "done" looks like
- **Include Examples**: Show expected inputs/outputs when relevant
- **List Constraints**: Call out specific invariants that apply

### ❌ Avoid

- **Vague Requests**: "Make it better" without specifics
- **Skipping Context**: Not mentioning relevant files or constraints
- **Ignoring Invariants**: Making changes that violate INVARIANTS.md
- **Over-Engineering**: Requesting features beyond current requirements
- **Breaking Changes**: Altering public APIs without version consideration

---

## Example Usage

### Example 1: Feature Development

```markdown
Using: engineering.md

Task: Add support for getting camera battery status

Context:
- Camera exposes battery status via USB command 0x15
- Response is single byte: 0-100 representing percentage
- Should update UI in usb.html to display battery level
- Must follow INV-SEC-003 (command validation)
- Must follow INV-DATA-001 (message integrity)

Success Criteria:
- New getBatteryStatus() function in dxo1usb.js
- UI shows battery percentage
- Validates battery command before sending
- Handles error if command fails
- Unit test for battery status parsing
```

### Example 2: Bug Fix

```markdown
Using: engineering.md

Task: Fix USB disconnect not updating connection state

Context:
- Issue: UI still shows "Connected" after device unplugged
- Violates: INV-DATA-003 (connection state consistency)
- Files: dxo1usb.js, usb.html
- Expected: Disconnect event should clear state and update UI

Success Criteria:
- Connection state accurately reflects hardware
- UI updates immediately on disconnect
- No phantom connection state
- Test with actual USB disconnect
```

### Example 3: Refactoring

```markdown
Using: engineering.md

Task: Extract command validation into reusable function

Context:
- Multiple functions validate commands (INV-SEC-003)
- Code duplication in sendCommand(), captureImage(), etc.
- Create validateCommand() utility
- Must not break existing API (INV-API-001)

Success Criteria:
- Single validateCommand(cmdType) function
- All command functions use it
- Existing tests still pass
- No breaking changes to exports
```

---

## Contributing New Prompts

### When to Create a New Prompt

Create a new prompt when:
- You find yourself repeating the same context for similar tasks
- A specific workflow emerges (e.g., USB protocol debugging)
- A new development phase begins (e.g., performance optimization)
- Community contributors need guidance for common tasks

### Prompt Creation Process

1. **Identify the need**: What recurring task needs guidance?
2. **Draft the prompt**: Use the template above
3. **Test it**: Use the prompt yourself for a real task
4. **Refine**: Update based on what worked/didn't work
5. **Document**: Add entry to this README
6. **Submit PR**: Share with the community

### Prompt Naming Convention

- Use kebab-case: `feature-implementation.md`
- Be descriptive: `usb-protocol-debug.md` not `debug.md`
- Indicate scope: `api-` prefix for API-specific prompts

---

## Relationship to Documentation

### Prompts vs. Documentation

| Documentation (docs/) | Prompts (prompts/) |
|----------------------|-------------------|
| **What** and **Why** | **How** to work with it |
| Reference material | Task-oriented guidance |
| Describes the system | Guides development process |
| Updated when code changes | Updated when workflows change |

### Integration

Prompts should reference documentation:
- `ARCHITECTURE.md` for system design context
- `INVARIANTS.md` for constraints and rules
- `ROADMAP.md` for feature priorities
- `DECISIONS.md` for historical context

---

## Maintenance

### Keeping Prompts Current

- **Review prompts** when architecture changes
- **Update references** when docs/ files are modified
- **Deprecate prompts** that no longer apply
- **Version prompts** alongside code (in git)

### Prompt Versioning

Prompts evolve with the project:
- Keep prompts in sync with current codebase state
- Mark deprecated prompts clearly
- Archive old prompts if they're no longer relevant
- Document breaking changes in prompt usage

---

## Resources

### Safe Vibe Coding

These prompts follow Safe Vibe Coding principles:
- https://github.com/Frostbite1536/Safe-Vibe-Coding

### LLM Best Practices

- Be explicit about constraints
- Provide concrete examples
- Define success criteria clearly
- Reference existing patterns in codebase
- Use structured format for consistency

---

**Last Updated**: 2026-01-04
**Prompt Count**: 1 (engineering.md)
**Status**: Active development
