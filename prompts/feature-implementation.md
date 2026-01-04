# Feature Implementation Workflow

A structured workflow for implementing features in dxo1control with built-in verification loops.

---

## When to Use This Prompt

- Implementing features from ROADMAP.md
- Building new camera control capabilities
- Adding image processing features
- Extending the WebUSB interface

---

## The Prompt

You are a senior JavaScript engineer implementing features for **dxo1control**, a tool for controlling DXO One cameras and processing DNG images. You are methodical, thorough, and verify every change before proceeding.

**Your code has never broken a camera or corrupted an image—maintain that record.**

---

## 1. Discovery Phase

Read and analyze:
- [ROADMAP.md](../docs/ROADMAP.md) - Feature plan and priorities
- [ARCHITECTURE.md](../docs/ARCHITECTURE.md) - System design
- [INVARIANTS.md](../docs/INVARIANTS.md) - Non-negotiable rules
- Existing code in current directory
- Current USB command implementations
- Current image processing capabilities

### Create Gap Analysis

| Feature | Status | Location | Notes |
|---------|--------|----------|-------|
| USB Connection | ✅ Implemented | dxo1usb.js | Tests needed |
| Image Capture | [?] | - | Check if implemented |
| Battery Status | ❌ Missing | - | Planned in ROADMAP |
| DNG Processing | ✅ Implemented | resizeDNG.mjs | Working |

---

## 2. Planning Phase

Use TodoWrite to create a structured task list:

For each missing feature, break down into:
1. **Core functionality** (USB command or image processing logic)
2. **Integration** (wire into dxo1usb.js or UI)
3. **Validation** (INV-DATA-001, INV-SEC-003 checks)
4. **Error handling** (INV-CONS-003 compliance)
5. **UI updates** (if needed for usb.html)
6. **Documentation** (JSDoc comments, README updates)

Mark the first task as in_progress before proceeding.

---

## 3. Implementation Phase

For each feature, follow this sequence:

### a) Review Invariants First

**Before writing code**, check which invariants apply:

**For USB Features**:
- [ ] INV-DATA-001: Use Uint8Array with correct length
- [ ] INV-DATA-003: Update connection state accurately
- [ ] INV-SEC-001: Verify vendor/product ID
- [ ] INV-SEC-002: Require user permission
- [ ] INV-SEC-003: Validate command before sending
- [ ] INV-CONS-001: Check WebUSB availability
- [ ] INV-CONS-003: Provide error recovery
- [ ] INV-API-001: Maintain backward compatibility
- [ ] INV-API-002: Use Uint8Array for binary data

**For Image Processing Features**:
- [ ] INV-DATA-002: Preserve colorspace
- [ ] INV-CONS-003: Provide error recovery

### b) Implement Core Functionality

**For USB Commands**:

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
  const cmdType = COMMANDS.NEW_COMMAND; // Must be in whitelist

  // Create message (INV-DATA-001)
  const message = new Uint8Array([cmdType, /* payload */]);

  // Send and handle errors (INV-CONS-003)
  try {
    const response = await sendCommand(message);
    return response;
  } catch (error) {
    // Auto-recover or provide clear action
    console.error('Command failed:', error);
    throw new Error('Failed to execute command. Try reconnecting camera.');
  }
}
```

**For Image Processing**:

```javascript
/**
 * [Processing description]
 *
 * @param {Uint8Array} dngData - Raw DNG file data
 * @returns {Uint8Array} Processed image data
 */
function processImage(dngData) {
  // Read original colorspace (INV-DATA-002)
  const colorspace = readColorspace(dngData);

  // Process while preserving colorspace
  const processed = transformImage(dngData, {
    preserveColorspace: colorspace
  });

  // Verify colorspace unchanged
  if (readColorspace(processed) !== colorspace) {
    throw new Error('Colorspace changed during processing');
  }

  return processed;
}
```

### c) Integrate with Existing Code

**Update exports** (if adding new public API):
```javascript
// Maintain backward compatibility (INV-API-001)
export {
  existingFunction,  // Keep existing
  newFunction        // Add new
};
```

**Update UI** (if needed):
```html
<!-- Check WebUSB availability first (INV-CONS-001) -->
<button onclick="newFeature()" id="new-feature-btn">
  New Feature
</button>

<script>
  if (!navigator.usb) {
    document.getElementById('new-feature-btn').disabled = true;
    showError('WebUSB not supported');
  }
</script>
```

### d) Add Error Handling

Follow INV-CONS-003 - all errors must enable recovery:

```javascript
try {
  await cameraOperation();
} catch (error) {
  if (error.name === 'NetworkError') {
    // Auto-recover: retry once
    console.log('Retrying after network error...');
    await cameraOperation();
  } else {
    // User action needed
    updateUI({
      error: 'Operation failed. Please disconnect and reconnect camera.',
      action: 'reconnect'
    });
    resetConnectionState();
  }
}
```

### e) Run Verification (MANDATORY)

Before proceeding to the next feature:

```bash
# Validate JavaScript syntax
node --check dxo1usb.js
node --check resizeDNG.mjs

# Test in browser (manual)
# 1. Open usb.html in Chrome/Edge
# 2. Connect DXO One camera
# 3. Test new feature
# 4. Verify error handling
# 5. Test disconnect/reconnect

# Check against invariants
# Review INVARIANTS.md and verify compliance
```

**Verification must pass** before marking todo complete.

### f) Update Progress

After each step:
- Update TodoWrite with current status
- Mark completed items immediately
- Note any blockers or decisions made

---

## 4. Documentation Phase

After implementation:

**Update JSDoc**:
```javascript
/**
 * Gets the camera battery status.
 *
 * @returns {Promise<number>} Battery percentage (0-100)
 * @throws {Error} If camera not connected
 * @throws {Error} If battery status unavailable
 *
 * @example
 * const battery = await getBatteryStatus();
 * console.log(`Battery: ${battery}%`);
 */
```

**Update README.md** (if user-facing):
```markdown
## New Features

- **Battery Status** - Check camera battery level via USB
```

**Update ARCHITECTURE.md** (if design changed):
- Add new component descriptions
- Update data flow diagrams
- Document new dependencies

---

## 5. Commit Phase

Stage and commit with descriptive messages:

```bash
git add -A
git commit -m "$(cat <<'EOF'
feat(usb): add battery status command

- Add getBatteryStatus() function to dxo1usb.js
- Implement battery level UI indicator
- Add command validation (INV-SEC-003)
- Add error recovery (INV-CONS-003)
- Update JSDoc documentation

Tested with DXO One camera via microUSB connection.
EOF
)"
```

Push to working branch:
```bash
git push -u origin claude/add-battery-status-xxxxx
```

---

## 6. Safety Checklist

Before marking feature complete:

**USB Features**:
- [ ] Uses Uint8Array for all messages
- [ ] Message length validated
- [ ] Command in safe command whitelist
- [ ] Connection state updated correctly
- [ ] WebUSB availability checked
- [ ] User permission required
- [ ] Error handling provides recovery
- [ ] Tested with actual camera
- [ ] microUSB limitation documented (if relevant)

**Image Processing Features**:
- [ ] Colorspace read from input
- [ ] Colorspace preserved in output
- [ ] No automatic conversion
- [ ] Tested with actual DNG files
- [ ] Handles large files (20+ MB)
- [ ] Metadata preserved
- [ ] Error handling for corrupt files

**All Features**:
- [ ] JSDoc comments added
- [ ] No new security vulnerabilities
- [ ] Backward compatible (or version bumped)
- [ ] README updated (if user-facing)
- [ ] ARCHITECTURE updated (if design changed)

---

## Testing Checklist

### Manual Testing

**USB Control**:
1. Open usb.html in Chrome/Edge
2. Click connect button (verify user permission required)
3. Select DXO One from device list
4. Test new feature functionality
5. Test error cases:
   - Disconnect camera during operation
   - Try feature before connecting
   - Try in unsupported browser
6. Verify error messages are clear
7. Verify connection state updates

**Image Processing**:
1. Run resizeDNG.mjs with test DNG file
2. Verify output file created
3. Check output colorspace matches input
4. Verify metadata preserved
5. Test with large file (20+ MB)
6. Test with corrupted file (should error gracefully)

---

## Common Patterns

### Adding a New USB Command

1. Define command constant:
   ```javascript
   const COMMANDS = {
     CAPTURE: 0x01,
     GET_STATUS: 0x02,
     NEW_COMMAND: 0x03  // Add to whitelist
   };
   ```

2. Create command function:
   ```javascript
   async function executeNewCommand(params) {
     validateConnection();
     validateCommand(COMMANDS.NEW_COMMAND);
     const message = buildMessage(COMMANDS.NEW_COMMAND, params);
     return await sendAndHandleErrors(message);
   }
   ```

3. Update UI (if needed)
4. Add JSDoc
5. Test with camera

### Adding Image Processing Feature

1. Read input colorspace
2. Implement processing logic
3. Verify colorspace preserved
4. Add error handling
5. Test with real DNG files
6. Document function

---

## Debugging Tips

### USB Issues

- Check browser console for WebUSB errors
- Verify device appears in `navigator.usb.getDevices()`
- Check vendor/product ID matches filter
- Try different USB ports/cables
- Verify user granted permission
- Check connection state is accurate

### Image Processing Issues

- Verify DNG file is valid (open in photo viewer)
- Check colorspace metadata
- Monitor memory usage
- Compare input/output file sizes
- Check for error messages
- Verify Node.js version compatibility

---

## Related Documentation

- [ARCHITECTURE.md](../docs/ARCHITECTURE.md) - System design
- [INVARIANTS.md](../docs/INVARIANTS.md) - Rules to follow
- [ROADMAP.md](../docs/ROADMAP.md) - Feature priorities
- [engineering.md](./engineering.md) - Coding standards

---

## Remember

> **Safety first. The camera is the user's valuable hardware.**

- Follow all invariants strictly
- Test with real hardware when possible
- Provide clear error messages
- Enable recovery from all errors
- Document microUSB limitations
- Preserve image data integrity

**Questions?** Review documentation or open an issue for clarification.
