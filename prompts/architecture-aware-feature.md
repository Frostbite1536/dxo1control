# Architecture-Aware Feature

Use this prompt when adding new functionality to dxo1control. It ensures the feature aligns with architecture and respects invariants.

---

## Prompt

You are extending **dxo1control** with a new feature. Your implementation must respect the established architecture and system contracts.

**Feature to implement**: [describe the feature]

**Context**: [relevant background or user requirements]

**Project**: dxo1control - DXO One camera control via USB and DNG image processing

---

## Before You Write Any Code

Complete these steps in order:

### 1. Architecture Review

**Read these documents**:
- [ARCHITECTURE.md](../docs/ARCHITECTURE.md) - System structure
- [INVARIANTS.md](../docs/INVARIANTS.md) - Non-negotiable rules
- [ROADMAP.md](../docs/ROADMAP.md) - Feature priorities
- [README.md](../README.md) - Project overview

**Then answer**:

**Where does this feature belong?**
- [ ] USB Control Layer (dxo1usb.js) - Camera commands and communication
- [ ] Utility Layer (u8a.js) - Uint8Array helpers
- [ ] Web Interface (usb.html) - UI for camera control
- [ ] Post-Processing (resizeDNG.mjs) - Image file processing
- [ ] New module (justify why existing modules don't fit)

**Which components will it interact with?**
- [ ] WebUSB API (browser feature detection required)
- [ ] DXO One camera (USB communication required)
- [ ] File system (Node.js for image processing)
- [ ] UI (HTML/DOM updates)

**What's the data flow?**
```
[Describe flow, e.g.:]
User clicks button → Request USB permission → Send command (Uint8Array) →
Receive response → Parse status → Update UI
```

**Does this extend existing or create new?**
- Extending existing: Which function/module?
- Creating new: Where will it live? What will it export?

---

### 2. Invariant Check

**Review all 11 invariants** in [INVARIANTS.md](../docs/INVARIANTS.md):

**Data Integrity**:
- [ ] INV-DATA-001: USB messages as Uint8Array with proper length
- [ ] INV-DATA-002: DNG colorspace preservation (if processing images)
- [ ] INV-DATA-003: Connection state consistency

**Security**:
- [ ] INV-SEC-001: USB vendor/product ID verification
- [ ] INV-SEC-002: User permission required for USB access
- [ ] INV-SEC-003: Command validation before transmission

**Consistency**:
- [ ] INV-CONS-001: WebUSB availability check (if using USB)
- [ ] INV-CONS-002: microUSB warning (if new USB feature)
- [ ] INV-CONS-003: Error recovery or clear user action

**API Contracts**:
- [ ] INV-API-001: Backward compatible exports
- [ ] INV-API-002: Uint8Array for binary data

**Which invariants apply to this feature?** [List them]

**Any invariant conflicts?** [Flag immediately and propose resolution]

**New invariants needed?** [Propose if this introduces new guarantees]

---

### 3. Testing Strategy

**Identify required tests**:

**Happy Path**:
- What's the expected success flow?
- What should the output be?

**Edge Cases**:
- Empty/null inputs?
- Maximum/minimum values?
- Disconnected camera?
- Unsupported browser?

**Failure Modes**:
- What can go wrong?
- How should errors be handled?
- Can user recover from errors?

**Integration Points**:
- Which existing functionality could be affected?
- What integration testing is needed?

**Plan to test**:
- [ ] Manual testing with actual DXO One camera
- [ ] Manual testing in Chrome/Edge (WebUSB supported)
- [ ] Manual testing error cases (disconnect, wrong input)
- [ ] Browser compatibility (show error in Firefox/Safari)

---

### 4. Integration Plan

**Public API**:
- What functions/exports will be added?
- What parameters do they take?
- What do they return?
- Will they be backward compatible? (INV-API-001)

**Configuration/Setup**:
- Any new constants or configuration needed?
- Any UI changes required?
- Any new dependencies?

**Affects existing functionality?**
- Which existing features might be impacted?
- Are there backward compatibility concerns?
- Will existing tests still pass?

**Documentation updates**:
- README.md (if user-facing feature)
- ARCHITECTURE.md (if design changes)
- JSDoc comments (for all new functions)

---

## Implementation Guidelines

### Minimal Surface Area

- Implement **only** what's requested
- Don't refactor unrelated code
- Don't add "nice to have" features
- Keep changes localized

### Follow Existing Patterns

**USB Commands** (if adding camera control):
```javascript
/**
 * [Command description]
 *
 * @returns {Promise<Uint8Array>} Response from camera
 * @throws {Error} If camera not connected
 * @throws {Error} If command fails
 */
async function newCameraCommand() {
  // Check connection (INV-DATA-003)
  if (!isConnected) {
    throw new Error('Camera not connected');
  }

  // Validate command (INV-SEC-003)
  const commandType = COMMANDS.NEW_COMMAND;

  // Create message (INV-DATA-001)
  const message = new Uint8Array([commandType /* , params */]);

  // Send and handle errors (INV-CONS-003)
  try {
    const response = await sendCommand(message);
    return response;
  } catch (error) {
    console.error('Command failed:', error);
    throw new Error('Failed to execute command. Try reconnecting camera.');
  }
}
```

**Image Processing** (if adding DNG feature):
```javascript
/**
 * [Processing description]
 *
 * @param {string} filePath - Path to DNG file
 * @returns {Uint8Array} Processed image data
 */
function processImage(filePath) {
  const imageData = readDNGFile(filePath);

  // Preserve colorspace (INV-DATA-002)
  const originalColorspace = readColorspace(imageData);

  const processed = transformImage(imageData, {
    preserveColorspace: originalColorspace
  });

  // Verify colorspace preserved
  if (readColorspace(processed) !== originalColorspace) {
    throw new Error('Colorspace changed during processing');
  }

  return processed;
}
```

**UI Updates** (if adding interface elements):
```html
<!-- Check WebUSB availability (INV-CONS-001) -->
<script>
  if (!navigator.usb) {
    document.getElementById('new-feature-btn').disabled = true;
    showError('WebUSB not supported. Use Chrome, Edge, or Opera.');
  }
</script>

<button onclick="newFeature()" id="new-feature-btn">
  New Feature
</button>
```

### Maintain Modularity

- Keep new files focused and < 500 lines
- If extending existing files, consider extracting new module
- Clear boundaries between components
- Minimize dependencies

### Explicit Over Implicit

- Make dependencies clear
- Avoid hidden state or side effects
- Document assumptions (JSDoc)
- Make error cases explicit

---

## Implementation Checklist

Before submitting:

**Functionality**:
- [ ] Feature works as specified
- [ ] Tested manually with actual hardware (if USB)
- [ ] Tested with real DNG files (if image processing)
- [ ] Error cases handled gracefully

**Code Quality**:
- [ ] Follows existing code style
- [ ] JSDoc comments added
- [ ] No magic numbers (use named constants)
- [ ] Clear variable/function names

**Invariants**:
- [ ] All applicable invariants respected
- [ ] No new invariant violations
- [ ] New invariants documented (if needed)

**Integration**:
- [ ] Exports added properly (INV-API-001)
- [ ] Backward compatible
- [ ] No breaking changes to existing API
- [ ] Integration points tested

**Documentation**:
- [ ] README.md updated (if user-facing)
- [ ] ARCHITECTURE.md updated (if design changes)
- [ ] microUSB limitation noted (if USB feature)
- [ ] JSDoc complete and accurate

**Security & Safety**:
- [ ] No new security vulnerabilities
- [ ] User permission required (if USB)
- [ ] Commands validated (if USB)
- [ ] Won't damage camera or corrupt data

---

## Output Format

### 1. Architecture Summary

**Where this fits**: [Component and rationale]

**Components affected**:
- **dxo1usb.js**: [Changes needed, if any]
- **usb.html**: [Changes needed, if any]
- **resizeDNG.mjs**: [Changes needed, if any]
- **New file**: [If creating new module, explain why]

**Data flow**:
```
[Describe the complete flow through the system]
```

**Dependencies**:
- Uses: [Existing functions/modules]
- Exports: [New public API]
- Integrates with: [External APIs like WebUSB]

---

### 2. Invariant Confirmation

**Checked invariants**:

- **INV-DATA-001**: ✅ Uses Uint8Array for USB messages
- **INV-DATA-003**: ✅ Checks connection state before commands
- **INV-SEC-003**: ✅ Command validated against whitelist
- **INV-CONS-001**: ✅ Checks navigator.usb availability
- **INV-CONS-003**: ✅ Provides error recovery
- **INV-API-001**: ✅ Additive change, no breaking changes
- **INV-API-002**: ✅ Returns Uint8Array for binary data

**Not applicable**:
- INV-DATA-002: (Not processing images)
- INV-SEC-001, INV-SEC-002: (Using existing connection)
- INV-CONS-002: (Not a new USB connection feature)

**New invariants needed**: [None / List if any]

---

### 3. Test Plan

**Manual Tests**:
1. [Test case 1: Happy path]
   - Steps: ...
   - Expected: ...

2. [Test case 2: Error case]
   - Steps: ...
   - Expected: ...

3. [Test case 3: Edge case]
   - Steps: ...
   - Expected: ...

**Integration Tests**:
- [Which existing features to verify still work]

**Browser/Environment Tests**:
- [ ] Chrome/Edge (should work)
- [ ] Firefox/Safari (should show clear error if WebUSB needed)
- [ ] With actual DXO One camera
- [ ] Disconnect during operation

---

### 4. Implementation

[Provide code organized by file]

**File: dxo1usb.js**
```javascript
// Code here
```

**File: usb.html**
```html
<!-- Code here -->
```

**File: (new file if needed)**
```javascript
// Code here
```

---

### 5. Documentation Updates

**README.md**:
```markdown
[Changes or additions needed]
```

**ARCHITECTURE.md**:
```markdown
[Changes if architecture affected]
```

**JSDoc**:
```javascript
/**
 * [Complete JSDoc for all new functions]
 */
```

---

## Red Flags to Avoid

❌ **Don't** start coding before understanding architecture

❌ **Don't** skip the invariant check

❌ **Don't** assume WebUSB is available

❌ **Don't** send unvalidated commands to camera

❌ **Don't** modify colorspace in image processing

❌ **Don't** break backward compatibility

❌ **Don't** add features beyond the requirements

❌ **Don't** refactor unrelated code

❌ **Don't** test only in Chrome (check error handling in other browsers)

❌ **Don't** forget microUSB warning for USB features

---

## Common Feature Types

### Adding USB Command

1. Define command constant (add to whitelist)
2. Create function following USB pattern
3. Validate command (INV-SEC-003)
4. Handle errors with recovery (INV-CONS-003)
5. Update UI if needed
6. Test with camera

### Adding Image Processing

1. Read colorspace (INV-DATA-002)
2. Implement processing logic
3. Preserve and verify colorspace
4. Handle large files gracefully
5. Test with real DNG files

### Adding UI Feature

1. Check WebUSB availability (INV-CONS-001)
2. Show error if unavailable
3. Require user interaction for USB
4. Update connection state on changes
5. Test in multiple browsers

---

## Remember

**You're not just adding a feature—you're maintaining the integrity of a system.**

- Camera hardware can be damaged by wrong commands
- User's image data can be lost by wrong processing
- Follow all invariants strictly
- Test with real hardware
- Provide clear error messages
- Enable recovery from all errors

**When in doubt, ask before implementing.**

---

**Related Documentation**:
- [ARCHITECTURE.md](../docs/ARCHITECTURE.md) - System design
- [INVARIANTS.md](../docs/INVARIANTS.md) - Rules to follow
- [ROADMAP.md](../docs/ROADMAP.md) - Feature priorities
- [engineering.md](./engineering.md) - Coding standards
- [feature-implementation.md](./feature-implementation.md) - Implementation workflow
