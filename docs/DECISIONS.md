# Decisions Log

This document records significant technical and architectural decisions made during the development of dxo1control. Each entry captures the context, options considered, and rationale for the chosen approach.

---

## Decision Format

Each decision should follow this template:

```markdown
## DEC-###: [Decision Title]

**Date**: YYYY-MM-DD
**Status**: [Proposed | Accepted | Deprecated | Superseded]
**Deciders**: [Who made the decision]
**Context**: [What led to this decision]

### Options Considered

1. **Option A**: Description
   - Pros: ...
   - Cons: ...

2. **Option B**: Description
   - Pros: ...
   - Cons: ...

### Decision

[Chosen option and explanation]

### Consequences

- Positive: ...
- Negative: ...
- Neutral: ...
```

---

## DEC-001: Use WebUSB for Camera Control

**Date**: Pre-project (inherited from community research)
**Status**: Accepted
**Deciders**: Original project authors
**Context**: Need to control DXO One camera from a web browser without requiring native drivers or applications.

### Options Considered

1. **WebUSB Browser API**
   - Pros: Cross-platform, no driver installation, easy deployment, works in browser
   - Cons: Limited browser support, requires user permission, browser-only

2. **Native USB Libraries (libusb/node-usb)**
   - Pros: More control, works in any environment, no browser limitations
   - Cons: Requires Node.js, platform-specific builds, harder deployment

3. **Serial Port Communication**
   - Pros: Simpler protocol, better browser support
   - Cons: Camera doesn't expose serial interface, not applicable

### Decision

Use WebUSB API for camera control. This aligns with the project goal of providing easy-to-use browser-based camera control and leverages existing community research.

### Consequences

- ✅ Positive: Easy demo deployment (see: https://dxo1demo.jsyang.ca/usb.html)
- ✅ Positive: No installation required for end users
- ✅ Positive: Cross-platform compatibility (any WebUSB browser)
- ❌ Negative: Limited to Chrome, Edge, Opera (not Firefox/Safari)
- ❌ Negative: Requires user permission for each session
- ⚪ Neutral: Separate Node.js tools needed for post-processing

---

## DEC-002: Focus on microUSB Connection Only

**Date**: Pre-project (inherited from community research)
**Status**: Accepted
**Deciders**: Original project authors, based on github.com/rickdeck/DxO-One
**Context**: DXO One camera supports both microUSB and Lightning connectors. Lightning protocol is not publicly documented.

### Options Considered

1. **microUSB Only (Documented Protocol)**
   - Pros: Protocol understood from community research, working implementation exists
   - Cons: Doesn't support iOS/Lightning users

2. **Reverse-Engineer Lightning Protocol**
   - Pros: Would support iOS users, full camera connectivity
   - Cons: Significant reverse-engineering effort, potential legal issues, no guarantee of success

3. **Support Both Protocols**
   - Pros: Maximum compatibility
   - Cons: Can't do this without Lightning protocol documentation

### Decision

Focus on microUSB connection using the protocol documented by rickdeck/DxO-One. Clearly document this limitation and warn users about Lightning support status.

### Consequences

- ✅ Positive: Working implementation available immediately
- ✅ Positive: Builds on proven community research
- ✅ Positive: Legal and ethical clarity
- ❌ Negative: iOS users cannot use Lightning connection
- ❌ Negative: Requires prominent warnings in documentation
- ⚪ Neutral: Future opportunity to add Lightning support if protocol is documented

---

## DEC-003: Separate Tools for USB Control and Post-Processing

**Date**: Pre-project (original architecture)
**Status**: Accepted
**Deciders**: Original project authors
**Context**: Need both real-time camera control and offline image processing capabilities.

### Options Considered

1. **Separate Tools (Current Approach)**
   - Pros: Clear separation of concerns, optimized for each environment
   - Cons: No integrated workflow, different runtime requirements

2. **Integrated Application (Electron)**
   - Pros: Single application, integrated workflow, native features
   - Cons: More complex, requires installation, larger download

3. **Web-Only Solution**
   - Pros: Everything in browser, easy deployment
   - Cons: Limited file system access, performance constraints for image processing

### Decision

Maintain separate tools: `dxo1usb.js` for browser-based USB control, `resizeDNG.mjs` for Node.js-based post-processing. Each tool is optimized for its environment.

### Consequences

- ✅ Positive: Each tool can use appropriate APIs (WebUSB vs. Node.js file system)
- ✅ Positive: Lightweight browser demo doesn't need image processing dependencies
- ✅ Positive: Users can use tools independently
- ❌ Negative: No single integrated workflow
- ❌ Negative: Different installation/setup for each tool
- ⚪ Neutral: Future option to create Electron wrapper for integration (Phase 4)

---

## DEC-004: Adopt Safe Vibe Coding Infrastructure

**Date**: 2026-01-04
**Status**: Accepted
**Deciders**: Current maintainers
**Context**: Project needs better documentation, testing, and development guidelines to support community contributions and LLM-assisted development.

### Options Considered

1. **Safe Vibe Coding Bootstrap**
   - Pros: Comprehensive documentation structure, clear invariants, LLM-friendly
   - Cons: Initial documentation effort, overhead for small changes

2. **Minimal Documentation**
   - Pros: Less upfront work, faster to start coding
   - Cons: Harder for contributors, no clear guidelines, LLM context issues

3. **Standard OSS Practices Only**
   - Pros: Familiar to contributors, well-established patterns
   - Cons: Lacks LLM-specific optimizations, less explicit about invariants

### Decision

Adopt Safe Vibe Coding infrastructure including ARCHITECTURE.md, INVARIANTS.md, ROADMAP.md, and custom engineering prompts. This provides clear structure for both human and LLM contributors.

### Consequences

- ✅ Positive: Clear documentation of system design and constraints
- ✅ Positive: Better LLM-assisted development with explicit context
- ✅ Positive: Easier onboarding for new contributors
- ✅ Positive: Invariants prevent common bugs and breaking changes
- ❌ Negative: Initial time investment to create documentation
- ❌ Negative: Documentation maintenance overhead
- ⚪ Neutral: Sets foundation for Phase 1 completion

---

## Future Decisions (To Be Made)

### Pending

- **Testing Framework**: Jest vs. Mocha vs. Vitest vs. native Node.js test runner
- **Type System**: TypeScript adoption vs. JSDoc comments vs. no types
- **Linting**: ESLint configuration and rules
- **Build System**: Whether to introduce build step or keep simple scripts
- **API Documentation**: JSDoc, TypeDoc, or manual documentation
- **Versioning**: When to tag v1.0.0

### Under Discussion

- None currently

---

## Deprecated Decisions

None yet.

---

## Notes

- Decisions should be recorded as they're made, not retroactively
- Include enough context for future developers to understand the reasoning
- Update status when decisions are superseded or deprecated
- Link to related issues, PRs, or discussions when available
- Use decision numbers (DEC-###) for easy reference in code comments

---

**Last Updated**: 2026-01-04
**Document Version**: 1.0
**Total Decisions**: 4 (3 inherited, 1 new)
