# Project Prompts

Reusable prompts for working with the dxo1control codebase. These prompts provide context and guidelines for LLM-assisted development.

---

## Available Prompts

### Core Prompts

- **[engineering.md](./engineering.md)** - Default prompt for feature development and code changes
  - Use for: Adding features, fixing bugs, refactoring code
  - Includes: Architecture principles, critical invariants, coding standards

### Development Workflow

- **[feature-implementation.md](./feature-implementation.md)** - Structured workflow for implementing features
  - Use for: Implementing features from ROADMAP.md, building new capabilities
  - Includes: Discovery, planning, implementation phases with verification loops

- **[architecture-aware-feature.md](./architecture-aware-feature.md)** - Adding features while respecting architecture
  - Use for: New functionality that must align with existing design
  - Includes: Architecture review, invariant checking, integration planning

### Quality Assurance

- **[bug-hunt.md](./bug-hunt.md)** - Systematic bug discovery and analysis
  - Use for: Proactive bug finding after changes, code review
  - Includes: USB communication, image processing, browser compatibility checks

- **[invariant-check.md](./invariant-check.md)** - Verify code against system invariants
  - Use for: Checking changes respect all 11 system invariants
  - Includes: Systematic checklist, violation detection, compliance reporting

### Optimization & Refactoring

- **[performance-review.md](./performance-review.md)** - Performance optimization and profiling
  - Use for: Identifying bottlenecks, improving efficiency
  - Includes: USB communication, image processing, browser performance

- **[refactor-for-clarity.md](./refactor-for-clarity.md)** - Improve code clarity without changing behavior
  - Use for: Making code more maintainable, improving readability
  - Includes: Naming, control flow, complexity reduction

### Specialized Tasks

- **[usb-protocol-debug.md](./usb-protocol-debug.md)** - Debug USB communication with DXO One camera
  - Use for: Troubleshooting USB issues, understanding protocol
  - Includes: Connection debugging, message format, transfer issues

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
Using: feature-implementation.md

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
- Tested with actual DXO One camera
```

### Example 2: Bug Hunting

```markdown
Using: bug-hunt.md

Task: Review recent USB communication changes for bugs

Context:
- Just implemented battery status and settings features
- Need to find bugs before users encounter them
- Focus on USB communication and error handling

Review Areas:
- USB message format (INV-DATA-001)
- Command validation (INV-SEC-003)
- Connection state management (INV-DATA-003)
- Error recovery (INV-CONS-003)

Success:
- Systematic review of all changes
- List of potential bugs found (or "none found")
- Priority and severity for each issue
```

### Example 3: Invariant Check

```markdown
Using: invariant-check.md

Task: Verify battery status feature respects invariants

Context:
- Adding new getBatteryStatus() function to dxo1usb.js
- Adding UI battery indicator
- Need to confirm all invariants are maintained

Check:
- All 11 invariants from INVARIANTS.md
- Identify any violations or ambiguities
- Propose new invariants if needed

Success:
- ✅ Invariants maintained (list which ones)
- ⚠️ Items needing attention (if any)
- ❌ Violations found (must fix before merge)
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

## Quick Reference

| Prompt | When to Use | Key Focus |
|--------|-------------|-----------|
| **engineering.md** | General development | Coding standards, workflow |
| **feature-implementation.md** | Adding new features | Structured implementation |
| **architecture-aware-feature.md** | Architecture-sensitive changes | Design alignment |
| **bug-hunt.md** | After changes | Proactive bug finding |
| **invariant-check.md** | Before merging | Invariant compliance |
| **performance-review.md** | Optimization needed | Bottleneck identification |
| **refactor-for-clarity.md** | Code hard to read | Maintainability |
| **usb-protocol-debug.md** | USB issues | Communication debugging |

---

**Last Updated**: 2026-01-04
**Prompt Count**: 8
**Status**: Active development
