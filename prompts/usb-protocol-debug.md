# USB Protocol Debug

Use this prompt when debugging USB communication issues with the DXO One camera.

---

## Prompt

You are debugging USB protocol communication between **dxo1control** and the DXO One camera. Your goal is to identify and fix communication errors, understand protocol issues, and improve reliability.

**Issue**: [Describe the problem - command fails, unexpected response, timeout, etc.]

**Context**: dxo1control uses WebUSB API to communicate with DXO One cameras via microUSB connection.

---

## Background

### DXO One USB Protocol

**Known Information** (from community research):
- **microUSB connection**: Protocol documented by github.com/rickdeck/DxO-One
- **Lightning connection**: Protocol unknown, not supported
- **WebUSB API**: Browser standard for USB device communication
- **Connection type**: USB 2.0
- **Data format**: Binary messages (Uint8Array)

**Community Resources**:
- https://github.com/rickdeck/DxO-One - microUSB protocol
- https://github.com/yeongrokgim/dxo-one-firmware-study - debug output

---

## Debugging Checklist

### 1. Connection Issues

**Verify Basic Connection**:
- [ ] Is WebUSB API available? (`navigator.usb` exists)
- [ ] Is browser supported? (Chrome, Edge, Opera)
- [ ] Is camera powered on?
- [ ] Is camera in correct mode for USB control?
- [ ] Is microUSB cable connected (not Lightning)?
- [ ] Does device appear in `navigator.usb.getDevices()`?

**Check Device Selection**:
```javascript
// Log available devices
const devices = await navigator.usb.getDevices();
console.log('Available devices:', devices);

// Log device info
devices.forEach(device => {
  console.log('Vendor ID:', device.vendorId.toString(16));
  console.log('Product ID:', device.productId.toString(16));
  console.log('Manufacturer:', device.manufacturerName);
  console.log('Product:', device.productName);
});
```

**Expected DXO One IDs**:
- Vendor ID: [Document actual vendor ID]
- Product ID: [Document actual product ID]

**Filter Configuration**:
```javascript
// Ensure filter matches actual camera
const filters = [
  {
    vendorId: 0xVVVV,  // Replace with actual DXO One vendor ID
    productId: 0xPPPP   // Replace with actual DXO One product ID
  }
];

const device = await navigator.usb.requestDevice({ filters });
```

---

### 2. Message Format Issues

**Verify Message Structure**:

```javascript
// Log outgoing message
function sendCommand(commandType, payload = []) {
  const message = new Uint8Array([commandType, ...payload]);

  console.log('Sending command:');
  console.log('  Type:', '0x' + commandType.toString(16));
  console.log('  Length:', message.length);
  console.log('  Bytes:', Array.from(message).map(b => '0x' + b.toString(16).padStart(2, '0')));
  console.log('  Raw:', message);

  return sendUSB(message);
}
```

**Check for INV-DATA-001 Violations**:
- [ ] Is message a Uint8Array (not plain Array)?
- [ ] Does length field match actual message length?
- [ ] Are all bytes valid (0-255)?
- [ ] Is message properly formatted per protocol?

**Common Message Format Errors**:
```javascript
// ❌ Wrong: Plain array
const message = [0x01, 0x02];

// ✅ Right: Uint8Array
const message = new Uint8Array([0x01, 0x02]);

// ❌ Wrong: Length mismatch
const message = new Uint8Array(10);
// ... only fill 5 bytes

// ✅ Right: Length matches content
const message = new Uint8Array([0x01, 0x02, 0x03, 0x04, 0x05]);
```

---

### 3. Transfer Issues

**Log USB Transfers**:

```javascript
async function sendUSB(message) {
  try {
    console.log('📤 USB Transfer OUT');
    console.log('  Endpoint:', ENDPOINT_OUT);
    console.log('  Data:', Array.from(message));

    const result = await device.transferOut(ENDPOINT_OUT, message);

    console.log('✅ Transfer OUT complete');
    console.log('  Bytes sent:', result.bytesWritten);
    console.log('  Status:', result.status);

    return result;
  } catch (error) {
    console.error('❌ Transfer OUT failed');
    console.error('  Error:', error.name);
    console.error('  Message:', error.message);
    throw error;
  }
}

async function receiveUSB() {
  try {
    console.log('📥 USB Transfer IN');
    console.log('  Endpoint:', ENDPOINT_IN);

    const result = await device.transferIn(ENDPOINT_IN, BUFFER_SIZE);

    console.log('✅ Transfer IN complete');
    console.log('  Bytes received:', result.data.byteLength);
    console.log('  Status:', result.status);
    console.log('  Data:', Array.from(new Uint8Array(result.data.buffer)));

    return new Uint8Array(result.data.buffer);
  } catch (error) {
    console.error('❌ Transfer IN failed');
    console.error('  Error:', error.name);
    console.error('  Message:', error.message);
    throw error;
  }
}
```

**USB Endpoint Configuration**:
- [ ] Are endpoints correctly identified?
- [ ] Is endpoint direction correct (IN vs OUT)?
- [ ] Is transfer size within limits?
- [ ] Is device interface claimed?

**Common Transfer Errors**:
- **NetworkError**: Device disconnected or communication failed
- **NotFoundError**: Device not found or removed
- **InvalidStateError**: Interface not claimed or wrong state
- **DataError**: Invalid data or protocol error

---

### 4. Timing Issues

**Add Delays if Needed**:

```javascript
// Some cameras need time between commands
async function sendCommandWithDelay(command) {
  await sendCommand(command);
  await new Promise(resolve => setTimeout(resolve, 100)); // 100ms delay
  return await receiveResponse();
}
```

**Timeout Detection**:

```javascript
async function sendCommandWithTimeout(command, timeoutMs = 5000) {
  return Promise.race([
    sendCommand(command),
    new Promise((_, reject) =>
      setTimeout(() => reject(new Error('Command timeout')), timeoutMs)
    )
  ]);
}
```

**Monitor for Race Conditions**:
- [ ] Are multiple commands sent concurrently?
- [ ] Is there a command queue?
- [ ] Could responses be mixed up?

---

### 5. Response Parsing

**Log and Analyze Responses**:

```javascript
function parseResponse(response) {
  console.log('📋 Parsing response');
  console.log('  Length:', response.length);
  console.log('  Bytes:', Array.from(response).map(b => '0x' + b.toString(16).padStart(2, '0')));

  if (response.length === 0) {
    console.error('❌ Empty response');
    return null;
  }

  const statusByte = response[0];
  console.log('  Status byte:', '0x' + statusByte.toString(16));

  // Parse based on known protocol
  switch (statusByte) {
    case 0x00:
      console.log('  ✅ Success');
      return { status: 'ok', data: response.slice(1) };
    case 0x01:
      console.log('  ❌ Error');
      return { status: 'error', code: response[1] };
    default:
      console.log('  ⚠️  Unknown status');
      return { status: 'unknown', raw: response };
  }
}
```

**Validation**:
- [ ] Is response non-empty?
- [ ] Is response format expected?
- [ ] Are all bytes within valid range?
- [ ] Does checksum match (if protocol uses checksums)?

---

### 6. Command-Specific Debugging

**Test Individual Commands**:

```javascript
// Test each command type separately
async function testCommands() {
  const tests = [
    { name: 'Get Status', cmd: 0x01 },
    { name: 'Get Battery', cmd: 0x15 },
    { name: 'Capture', cmd: 0x10 }
  ];

  for (const test of tests) {
    console.log(`\n🧪 Testing: ${test.name}`);
    try {
      const result = await sendCommand(test.cmd);
      console.log(`✅ ${test.name} succeeded:`, result);
    } catch (error) {
      console.error(`❌ ${test.name} failed:`, error);
    }
  }
}
```

**Compare with Known Working Implementations**:
- Check rickdeck/DxO-One for command examples
- Compare message formats
- Verify expected responses

---

## Common Issues and Solutions

### Issue: Device Not Found

**Symptoms**: `requestDevice()` doesn't show camera

**Debug**:
1. Check vendor/product ID filters match camera
2. Verify camera is powered on and in USB mode
3. Try different USB port
4. Check browser console for errors
5. Verify browser supports WebUSB

**Solution**:
```javascript
// Log filter being used
console.log('Requesting device with filters:', filters);

// Try without filters to see all devices (debug only)
const allDevices = await navigator.usb.requestDevice({ filters: [] });
console.log('All available devices:', allDevices);
```

---

### Issue: Transfer Fails Immediately

**Symptoms**: `transferOut()` throws error

**Debug**:
1. Check device interface is claimed
2. Verify endpoint number is correct
3. Check message format (Uint8Array)
4. Verify device not disconnected

**Solution**:
```javascript
// Claim interface before transfer
await device.open();
await device.selectConfiguration(1);
await device.claimInterface(0);  // Usually interface 0

// Then transfer
await device.transferOut(endpointOut, message);
```

---

### Issue: Response Timeout

**Symptoms**: Never receive response from camera

**Debug**:
1. Check if command is valid
2. Verify endpoint IN is correct
3. Check buffer size is sufficient
4. Monitor for camera going to sleep

**Solution**:
```javascript
// Ensure camera is awake
await sendWakeupCommand();

// Increase timeout
const response = await receiveUSBWithTimeout(10000);  // 10s timeout
```

---

### Issue: Corrupt Response

**Symptoms**: Response has unexpected bytes

**Debug**:
1. Log complete response in hex
2. Check for partial reads
3. Verify protocol expectations
4. Look for timing issues

**Solution**:
```javascript
// Read full response size
const EXPECTED_SIZE = 64;  // Adjust based on protocol
const result = await device.transferIn(endpointIn, EXPECTED_SIZE);

// Verify size matches
if (result.data.byteLength !== EXPECTED_SIZE) {
  console.warn('Partial response received');
}
```

---

## Protocol Documentation

### Document New Findings

When you discover protocol details, document them:

```javascript
/**
 * DXO One USB Protocol - Command Reference
 *
 * Command Format:
 *   Byte 0: Command type
 *   Byte 1-N: Parameters (command-specific)
 *
 * Response Format:
 *   Byte 0: Status (0x00 = success, 0x01 = error)
 *   Byte 1-N: Response data (command-specific)
 *
 * Known Commands:
 *   0x01: Get device status
 *   0x10: Capture image
 *   0x15: Get battery level
 *   // Add more as discovered
 */
```

**Update ARCHITECTURE.md** with protocol findings

---

## USB Capture Tool

**Use browser DevTools or extension to capture USB traffic**:

1. Open Chrome DevTools → Network tab → Filter to "Other"
2. Look for USB transfer logs
3. Analyze command/response sequences
4. Compare with expected protocol

**Or use system-level USB monitoring**:
- Windows: USBPcap, Wireshark
- macOS: USB Prober
- Linux: usbmon, Wireshark

---

## Safety Reminders

⚠️ **Camera hardware can be damaged by invalid commands**

- Only send validated commands (INV-SEC-003)
- Don't experiment with unknown command codes
- Test with non-destructive commands first (status, battery)
- Have a recovery plan if camera becomes unresponsive

⚠️ **microUSB only - Lightning untested**

- Warn users about connection type (INV-CONS-002)
- Document any connector-specific issues found

---

## Output Format

### Debug Report

**Issue**: [Description of the problem]

**Environment**:
- Browser: [Chrome/Edge/Opera version]
- OS: [Windows/Mac/Linux]
- Camera: DXO One (microUSB connection)
- dxo1control version: [version or commit]

**Debugging Steps Taken**:
1. [Step 1 and result]
2. [Step 2 and result]
3. [Step 3 and result]

**USB Logs**:
```
[Paste relevant USB communication logs]
```

**Findings**:
- [What was discovered]
- [Root cause if identified]

**Solution**:
- [How issue was resolved]
- [Code changes needed]

**Related Invariants**:
- [Which invariants were violated or apply]

---

## Related Documentation

- [ARCHITECTURE.md](../docs/ARCHITECTURE.md) - USB communication design
- [INVARIANTS.md](../docs/INVARIANTS.md) - INV-DATA-001, INV-SEC-003
- https://github.com/rickdeck/DxO-One - microUSB protocol reference
- https://wicg.github.io/webusb/ - WebUSB specification

---

**Remember**: Document all protocol findings to help future debugging and development.
