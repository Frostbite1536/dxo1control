# Bug Hunt

Use this prompt after implementing features or making significant changes to proactively find bugs in dxo1control.

---

## Prompt

You are conducting a thorough bug hunt on recent code changes to **dxo1control**. Your goal is to find bugs **before** they reach users.

**Review the following changes**: [paste git diff or describe changes]

**Context**: dxo1control is a tool for controlling DXO One cameras via USB (WebUSB) and processing DNG image files.

---

## Review Methodology

### 1. USB Communication Issues

**Check for**:
- Invalid Uint8Array construction (violates INV-DATA-001)
- Message length mismatches
- Missing validation before USB transmission
- Incorrect vendor/product ID filtering (violates INV-SEC-001)
- Missing user permission checks (violates INV-SEC-002)
- Command types not validated against safe command set (violates INV-SEC-003)
- Connection state not updated on disconnect (violates INV-DATA-003)

**Questions**:
- Are all USB messages properly typed as Uint8Array?
- Is message length validated before sending?
- Are all commands in the known-safe command whitelist?
- Does connection state accurately reflect hardware status?

### 2. Image Processing Issues

**Check for**:
- Colorspace conversion where preservation is required (violates INV-DATA-002)
- Missing colorspace parameter in processing functions
- Input file validation missing
- Memory issues with large DNG files (20+ MB)
- Metadata loss during processing
- Output file corruption

**Questions**:
- Is original colorspace preserved through all operations?
- Are DNG files validated before processing?
- Can the code handle large files without running out of memory?
- Is metadata correctly preserved?

### 3. Browser Compatibility Issues

**Check for**:
- navigator.usb usage without availability check (violates INV-CONS-001)
- WebUSB API calls in unsupported browsers
- Missing error messages for unsupported browsers
- No graceful degradation

**Questions**:
- Is WebUSB API availability checked before use?
- Are clear error messages shown for unsupported browsers?
- Does the code gracefully degrade when WebUSB is unavailable?

### 4. Error Handling & Recovery

**Check for**:
- Silent failures (violates INV-CONS-003)
- Errors that leave connection in stuck state
- Missing auto-recovery mechanisms
- Error messages without user action guidance
- Missing error handling in async operations
- State not cleaned up on errors

**Questions**:
- Do all error paths either auto-recover or provide clear user actions?
- Is state properly reset on errors?
- Are errors surfaced to users appropriately?
- Can the system recover from disconnections?

### 5. Logic Errors

**Check for**:
- Conditional logic errors (off-by-one, incorrect operators)
- Loop termination issues
- Unhandled edge cases (empty arrays, null values)
- Incorrect default values
- Type coercion issues

**Questions**:
- Are array bounds checked?
- Do loops terminate correctly?
- Are null/undefined cases handled?

### 6. Data Flow Issues

**Check for**:
- Type mismatches (Array vs Uint8Array) (violates INV-API-002)
- Using plain Array where Uint8Array required
- Using Buffer (Node.js) in browser code
- Mutations where immutability expected
- Stale data being used

**Questions**:
- Is Uint8Array used consistently for binary data?
- Are all type assumptions validated?
- Is browser/Node.js compatibility maintained?

### 7. Security Vulnerabilities

**Check for**:
- Command injection via unvalidated user input
- Accessing wrong USB device (vendor/product ID not checked)
- Automatic connection attempts (no user permission)
- Sensitive data in logs or error messages
- XSS in UI (if HTML content is dynamically generated)

**Questions**:
- Is all user input validated?
- Are USB device filters properly configured?
- Is user permission required for all USB access?
- Are error messages safe to display?

### 8. Resource Management

**Check for**:
- USB connections not properly closed
- Memory leaks (event listeners not removed)
- File handles not closed in error paths
- Large allocations not freed
- Cleanup missing in error paths

**Questions**:
- Are USB connections cleaned up on disconnect?
- Are event listeners properly removed?
- Is cleanup code in error paths?

### 9. API Contract Violations

**Check for**:
- Breaking changes to exported functions (violates INV-API-001)
- Changed function signatures
- Removed exports
- Renamed exports without deprecation
- Return type changes

**Questions**:
- Are all public exports backward compatible?
- If breaking changes needed, is version bumped?
- Are deprecated functions warned before removal?

### 10. microUSB-Specific Issues

**Check for**:
- Missing warnings about microUSB-only support (violates INV-CONS-002)
- Assuming Lightning connector works
- Documentation not mentioning connection type limitations

**Questions**:
- Is microUSB-only limitation clearly documented?
- Are warnings displayed before connection?
- Is Lightning support status clear?

---

## dxo1control-Specific Checks

### USB Control (dxo1usb.js, usb.html)

- [ ] WebUSB availability checked before use
- [ ] User interaction required for device selection
- [ ] Vendor/Product ID filters in place
- [ ] All messages are Uint8Array instances
- [ ] Message length fields match actual length
- [ ] Commands validated against whitelist
- [ ] Connection state updated on disconnect events
- [ ] Error handling provides recovery path
- [ ] Browser compatibility clearly documented

### Image Processing (resizeDNG.mjs)

- [ ] Colorspace read from input file
- [ ] Colorspace preserved in output
- [ ] No automatic colorspace conversion
- [ ] Input file validated
- [ ] Memory usage reasonable for large files
- [ ] Metadata preserved
- [ ] Error handling for corrupted files
- [ ] Output quality acceptable

### Utility Layer (u8a.js)

- [ ] All functions work with Uint8Array
- [ ] No Array/Buffer mixing
- [ ] Type validation in place
- [ ] Edge cases handled (empty arrays, null)

---

## Output Format

For each potential bug found:

**Location**: [file:line or function name]

**Type**: [USB Communication | Image Processing | Browser Compat | Error Handling | Logic Error | Data Flow | Security | Resource Management | API Contract | microUSB]

**Issue**: [description of the problem]

**Invariant Violated**: [INV-XXX-### if applicable]

**Impact**: [High/Medium/Low]

**Severity**:
- **Critical**: Could damage camera, lose data, or crash
- **High**: Feature broken, poor UX, or data corruption
- **Medium**: Edge case bug, minor UX issue
- **Low**: Cosmetic, rare edge case

**Suggested Fix**: [how to fix it]

---

## If No Bugs Found

Explicitly state: "No bugs detected in this review"

And list what was checked:
- [ ] USB communication validation
- [ ] Image processing correctness
- [ ] Browser compatibility
- [ ] Error handling and recovery
- [ ] Security checks
- [ ] Resource cleanup
- [ ] API backward compatibility
- [ ] All 11 invariants verified

---

## Remember

- **Better to flag a false positive than miss a real bug**
- **USB bugs can damage camera hardware** - be extra careful
- **Image processing bugs can lose user data** - verify colorspace preservation
- **Check invariants systematically** - use INVARIANTS.md as checklist
- **Test with actual hardware when possible** - emulators may hide bugs

---

**Related Documentation**:
- [INVARIANTS.md](../docs/INVARIANTS.md) - System invariants
- [ARCHITECTURE.md](../docs/ARCHITECTURE.md) - System design
