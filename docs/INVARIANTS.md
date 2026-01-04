# Invariants

This document defines the non-negotiable rules for dxo1control. These invariants must hold true across all code changes, features, and refactoring.

---

## Data Integrity Invariants

### INV-DATA-001: USB Message Integrity

**Rule**: All USB messages sent to the camera MUST be valid Uint8Array instances with proper length fields.

**Rationale**: Invalid message formats can cause camera to enter undefined states, potentially requiring power cycle or causing data corruption.

**Examples**:

Valid:
```javascript
const message = new Uint8Array([0x01, 0x02, 0x03]);
// length field matches actual array length
```

Invalid:
```javascript
const message = [0x01, 0x02, 0x03];  // Plain array, not Uint8Array
const message = new Uint8Array([0x01, 0x02, 0x03]);
message.length = 5;  // Length mismatch
```

**Enforcement**:
- Type checking before USB transmission
- Length validation before send
- Unit tests for message construction
- Runtime assertions in development mode

### INV-DATA-002: DNG File Colorspace Preservation

**Rule**: Post-processing operations MUST preserve the original colorspace of DNG files. No automatic colorspace conversion is allowed.

**Rationale**: Photographers rely on preserving the original color profile from the camera sensor. Automatic conversion can degrade image quality and alter artistic intent.

**Examples**:

Valid:
```javascript
// Read original colorspace
const originalColorspace = dng.getColorspace();
// Process without modification
resizeImage(dng, { preserveColorspace: originalColorspace });
```

Invalid:
```javascript
// Automatic conversion to sRGB
resizeImage(dng, { convertToSRGB: true });
// Ignore original colorspace
processImage(dng);  // defaults to different colorspace
```

**Enforcement**:
- Explicit colorspace parameter in all image processing functions
- Unit tests verify colorspace preservation
- Code review checklist item
- Integration tests compare input/output colorspace metadata

### INV-DATA-003: Connection State Consistency

**Rule**: USB connection state MUST accurately reflect the actual hardware connection status. No phantom connections or stale state.

**Rationale**: Acting on stale connection state can lead to command failures, user confusion, and potential data loss if user assumes connection is active.

**Examples**:

Valid:
```javascript
// Check actual USB connection before operations
if (await device.isConnected()) {
  await sendCommand(cmd);
}

// Update state on disconnect events
device.on('disconnect', () => {
  connectionState = 'disconnected';
  clearCommandQueue();
});
```

Invalid:
```javascript
// Assume connection from stale flag
if (wasConnectedBefore) {
  sendCommand(cmd);  // May fail if device disconnected
}

// Don't update state on disconnect
// connectionState remains 'connected' after device removed
```

**Enforcement**:
- Real-time connection status checking before commands
- Event listeners for disconnect events
- Automatic state cleanup on disconnect
- Unit tests for state transitions

---

## Security Invariants

### INV-SEC-001: USB Device Vendor/Product ID Verification

**Rule**: WebUSB connection MUST only be established with devices matching DXO One vendor and product IDs. No wildcard or unverified device access.

**Rationale**: Prevents accidental or malicious control of non-camera USB devices. Protects users from unintended device access.

**Examples**:

Valid:
```javascript
const filters = [{
  vendorId: 0x1234,    // DXO One vendor ID
  productId: 0x5678    // DXO One product ID
}];
const device = await navigator.usb.requestDevice({ filters });
```

Invalid:
```javascript
// No filters - any USB device
const device = await navigator.usb.requestDevice({});

// Wildcard vendor ID
const filters = [{ vendorId: undefined }];
```

**Enforcement**:
- Hardcoded vendor/product ID constants
- Filter validation before requestDevice()
- Code review requirement
- Documentation of expected device IDs

### INV-SEC-002: User Permission Required for USB Access

**Rule**: USB access MUST require explicit user interaction and permission grant. No automatic or background connection attempts.

**Rationale**: WebUSB security model requires user consent. Attempting to bypass violates browser security and user trust.

**Examples**:

Valid:
```javascript
// Triggered by user click/button press
button.onclick = async () => {
  const device = await navigator.usb.requestDevice({ filters });
};
```

Invalid:
```javascript
// Automatic connection on page load
window.onload = async () => {
  const device = await navigator.usb.requestDevice({ filters });
};

// Background polling for devices
setInterval(async () => {
  try { await navigator.usb.requestDevice({ filters }); } catch {}
}, 1000);
```

**Enforcement**:
- WebUSB API enforces user gesture requirement
- Code review to prevent auto-connection patterns
- UI/UX design requires explicit connect button
- Documentation emphasizes user permission flow

### INV-SEC-003: Command Validation Before Transmission

**Rule**: All camera commands MUST be validated against a known-safe command set before transmission to the device.

**Rationale**: Sending arbitrary or malformed commands could damage the camera hardware, corrupt firmware, or cause undefined behavior.

**Examples**:

Valid:
```javascript
const SAFE_COMMANDS = {
  CAPTURE: 0x01,
  GET_STATUS: 0x02,
  SET_ISO: 0x03
};

function sendCommand(cmdType, payload) {
  if (!Object.values(SAFE_COMMANDS).includes(cmdType)) {
    throw new Error('Unknown command type');
  }
  // Proceed with transmission
}
```

Invalid:
```javascript
// Accept arbitrary command codes
function sendCommand(cmdType, payload) {
  device.transferOut(endpoint, new Uint8Array([cmdType, ...payload]));
}

// No validation of command parameters
sendCommand(userInput, userPayload);  // Direct user input
```

**Enforcement**:
- Whitelist of known-safe commands
- Input validation for command parameters
- Unit tests for command validation logic
- Fuzzing tests to verify rejection of invalid commands

---

## Consistency Invariants

### INV-CONS-001: Browser WebUSB Support Check

**Rule**: The application MUST check for WebUSB API availability before attempting any USB operations and provide clear error messaging if unavailable.

**Rationale**: Not all browsers support WebUSB. Attempting operations without the API causes confusing errors and poor user experience.

**Examples**:

Valid:
```javascript
if (!navigator.usb) {
  showError('WebUSB not supported. Please use Chrome, Edge, or Opera.');
  disableUSBFeatures();
  return;
}
// Proceed with USB operations
```

Invalid:
```javascript
// Assume WebUSB exists
await navigator.usb.requestDevice({ filters });  // May throw in Safari/Firefox
```

**Enforcement**:
- Feature detection at application startup
- Graceful degradation with clear messaging
- Unit tests with mocked navigator.usb
- Documentation lists supported browsers

### INV-CONS-002: microUSB Connection Warning

**Rule**: All documentation and UI MUST prominently warn that only microUSB connections are tested. Lightning connector usage is at user's risk.

**Rationale**: Lightning protocol is not reverse-engineered. Users attempting Lightning connection should be warned of untested behavior.

**Examples**:

Valid:
```javascript
// In UI:
<div class="warning">
  ⚠️ Only microUSB connection tested. Lightning connector not supported.
</div>

// In README:
> **Warning**: Only tested with microUSB connection! Lightning connector support is unknown.
```

Invalid:
```javascript
// Generic connection message
<div>Connect your DXO One camera via USB</div>

// No mention of connection type limitations
```

**Enforcement**:
- README includes warning
- UI displays warning before connection
- Code comments note limitation
- Documentation review checklist

### INV-CONS-003: Error State Recovery

**Rule**: All error conditions MUST either auto-recover or provide clear user actions for recovery. No silent failures or stuck states.

**Rationale**: Camera control involves hardware that can disconnect, fail, or timeout. Users need clear feedback and recovery paths.

**Examples**:

Valid:
```javascript
try {
  await sendCommand(cmd);
} catch (error) {
  if (error.name === 'NetworkError') {
    // Auto-recover: retry once
    await sendCommand(cmd);
  } else {
    // User action needed
    showError('Command failed. Please disconnect and reconnect camera.');
    resetConnectionState();
  }
}
```

Invalid:
```javascript
try {
  await sendCommand(cmd);
} catch (error) {
  // Silent failure
  console.log(error);
}

try {
  await sendCommand(cmd);
} catch (error) {
  // Stuck state - no recovery
  isProcessing = true;  // Never cleared
  throw error;
}
```

**Enforcement**:
- Error handling in all async operations
- State reset on errors
- User-facing error messages with action items
- Integration tests for error scenarios

---

## API Contract Invariants

### INV-API-001: Backward Compatible Exports

**Rule**: Public exports from `dxo1usb.js` and utility modules MUST maintain backward compatibility. Breaking changes require major version bump.

**Rationale**: External users and demo page depend on stable API. Breaking changes without version updates cause integration failures.

**Examples**:

Valid:
```javascript
// Add new optional parameter (backward compatible)
function sendCommand(type, payload, options = {}) { }

// Add new export (backward compatible)
export { sendCommand, newHelper };
```

Invalid:
```javascript
// Remove existing export (breaking)
export { sendCommand };  // Previously exported newHelper removed

// Change function signature (breaking)
function sendCommand(commandObject) { }  // Was (type, payload)

// Rename export (breaking)
export { sendCommand as transmitCommand };
```

**Enforcement**:
- Semantic versioning (semver)
- API compatibility tests
- Changelog review for breaking changes
- Deprecation warnings before removal

### INV-API-002: Uint8Array for Binary Data

**Rule**: All binary data interfaces MUST use Uint8Array. No mixing with Array, Buffer, or other types.

**Rationale**: Consistent typing prevents confusion, enables type checking, and ensures browser/Node.js compatibility.

**Examples**:

Valid:
```javascript
function processData(data: Uint8Array): Uint8Array {
  return new Uint8Array(data.length);
}
```

Invalid:
```javascript
function processData(data) {  // Accepts any type
  return Array.from(data);   // Returns Array instead of Uint8Array
}

function sendBytes(data: Buffer) {  // Node.js Buffer not browser-compatible
  // ...
}
```

**Enforcement**:
- TypeScript type annotations (if adopted)
- JSDoc type comments
- Runtime type validation in development
- Unit tests verify Uint8Array input/output

---

## Notes

- These invariants should be reviewed and updated as the project evolves
- Breaking an invariant requires explicit justification and team discussion
- CI/CD should include automated checks for enforcing invariants where possible
- New features should consider which invariants apply and document compliance

---

**Last Updated**: 2026-01-04
**Document Version**: 1.0
**Total Invariants**: 11
