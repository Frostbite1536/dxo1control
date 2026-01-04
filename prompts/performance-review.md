# Performance Review

Use this prompt to identify performance issues in dxo1control code or architecture.

---

## Prompt

You are conducting a performance review of **dxo1control**, a tool for controlling DXO One cameras via USB and processing DNG image files. Your goal is to identify bottlenecks, inefficiencies, and optimization opportunities.

**Scope**: [describe what to review - specific file/function/feature or entire system]

**Context**:
- USB communication: Real-time camera control over USB 2.0
- Image processing: DNG files (typically 15-25 MB each)
- Browser environment: Chrome/Edge with WebUSB API
- Node.js environment: CLI tools for batch processing

---

## Review Areas

### 1. USB Communication Performance

**Check for**:
- Multiple sequential USB transfers that could be batched
- Large command/response payloads that could be reduced
- Missing timeout logic (causing hung operations)
- Synchronous operations blocking UI thread
- Excessive connection state checking
- Inefficient Uint8Array operations

**Questions**:
- How many USB round-trips per camera operation?
- Are command responses cached when appropriate?
- Is UI responsive during USB operations?
- Are transfers properly queued or can they collide?

**Optimization opportunities**:
- Batch multiple commands into single transfer
- Cache stable camera settings (battery, model info)
- Use Web Workers for USB communication (keep UI thread free)
- Implement command queue to prevent collisions

**Example Issue**:
```javascript
// Bad: Sequential operations
async function getCameraInfo() {
  const battery = await getBattery();    // USB transfer 1
  const model = await getModel();        // USB transfer 2
  const firmware = await getFirmware();  // USB transfer 3
  return { battery, model, firmware };
}

// Better: Batch if protocol allows
async function getCameraInfo() {
  const response = await getBatchedInfo([
    COMMANDS.BATTERY,
    COMMANDS.MODEL,
    COMMANDS.FIRMWARE
  ]);
  return parseResponse(response);
}
```

---

### 2. Image Processing Performance

**Check for**:
- Loading entire DNG file into memory when streaming possible
- Inefficient image transformations (multiple passes)
- No progress reporting for long operations
- Synchronous operations blocking main thread
- Creating unnecessary intermediate copies
- Unoptimized buffer allocations

**Questions**:
- Can large files (20+ MB) be processed without OOM?
- Is processing time reasonable for batch operations?
- Are users informed of progress on large files?
- Can multiple files be processed in parallel?

**Optimization opportunities**:
- Stream file processing (chunk at a time)
- Use Worker threads for parallel batch processing
- Implement progress callbacks
- Optimize buffer reuse (avoid allocations)
- Consider native modules for CPU-intensive ops

**Example Issue**:
```javascript
// Bad: Load entire file into memory
function resizeDNG(filePath) {
  const entireFile = fs.readFileSync(filePath);  // 25 MB in memory
  const processed = processImage(entireFile);
  return processed;
}

// Better: Stream processing
function resizeDNG(filePath) {
  return new Promise((resolve, reject) => {
    const stream = fs.createReadStream(filePath);
    const chunks = [];
    stream.on('data', chunk => {
      const processed = processChunk(chunk);
      chunks.push(processed);
    });
    stream.on('end', () => resolve(Buffer.concat(chunks)));
  });
}
```

---

### 3. Browser Performance

**Check for**:
- Large JavaScript bundle size (slow initial load)
- No code splitting or lazy loading
- Blocking scripts during page load
- Unnecessary re-renders in UI
- Missing request animation frame for UI updates
- Excessive DOM manipulation

**Questions**:
- What's the bundle size?
- How long until interactive?
- Are UI updates smooth?
- Is WebUSB code loaded on every page or only when needed?

**Optimization opportunities**:
- Lazy load USB module only when user clicks "Connect"
- Use requestAnimationFrame for smooth UI updates
- Debounce/throttle frequent operations
- Minimize DOM updates (batch changes)

**Example Issue**:
```javascript
// Bad: Immediate import, always loaded
import * as usb from './dxo1usb.js';

// Better: Dynamic import when needed
async function connectCamera() {
  const usb = await import('./dxo1usb.js');
  return usb.connect();
}
```

---

### 4. Algorithmic Efficiency

**Check for**:
- O(n²) or worse algorithms on large datasets
- Unnecessary repeated computations
- Inefficient data structure choices
- Linear search where Map/Set would be better
- String concatenation in loops

**Questions**:
- What's the time complexity of hot paths?
- Are computations memoized when appropriate?
- Are data structures optimal for access patterns?

**dxo1control-Specific Checks**:
- Uint8Array operations (concatenation, search, copy)
- Command lookup (array vs object/map)
- Response parsing efficiency

**Example Issue**:
```javascript
// Bad: Linear search in array
const COMMANDS = [
  { id: 0x01, name: 'CAPTURE' },
  { id: 0x02, name: 'STATUS' },
  // ... 50 more
];
function getCommand(id) {
  return COMMANDS.find(cmd => cmd.id === id);  // O(n)
}

// Better: Map for O(1) lookup
const COMMANDS = new Map([
  [0x01, 'CAPTURE'],
  [0x02, 'STATUS'],
  // ...
]);
function getCommand(id) {
  return COMMANDS.get(id);  // O(1)
}
```

---

### 5. Memory Usage

**Check for**:
- Loading multiple large DNG files simultaneously
- Memory leaks (unreleased event listeners, closures)
- Excessive object creation in hot paths
- Large allocations that could be streamed
- Missing cleanup on disconnect

**Questions**:
- Does memory grow unbounded during batch processing?
- Are event listeners properly removed?
- Are large buffers released after use?
- Is there garbage collection pressure?

**dxo1control-Specific Checks**:
- Uint8Array allocations in USB communication
- DNG file buffers released after processing
- WebUSB device handles closed properly
- Event listeners removed on disconnect

**Example Issue**:
```javascript
// Bad: Memory leak
function setupCamera() {
  device.addEventListener('disconnect', () => {
    console.log('Disconnected');
  });
  // Listener never removed - leaks on reconnect
}

// Better: Cleanup listeners
let disconnectHandler;
function setupCamera() {
  disconnectHandler = () => console.log('Disconnected');
  device.addEventListener('disconnect', disconnectHandler);
}
function cleanup() {
  device.removeEventListener('disconnect', disconnectHandler);
}
```

---

### 6. Caching Opportunities

**Check for**:
- Repeated camera queries for stable data
- USB device information fetched multiple times
- Same DNG metadata read repeatedly
- No caching of processed results

**Questions**:
- What camera data is stable (model, serial)?
- What camera data changes (battery, settings)?
- Should processed images be cached?
- What's the cache invalidation strategy?

**Optimization opportunities**:
- Cache camera model/serial number (never changes)
- Cache battery/settings with TTL (changes slowly)
- Cache DNG metadata after first read
- Consider LRU cache for recently processed files

**Example Issue**:
```javascript
// Bad: Fetch every time
async function displayCameraInfo() {
  const model = await getModel();  // USB call
  // ... later ...
  const model2 = await getModel();  // Redundant USB call
}

// Better: Cache stable data
const cameraInfoCache = new Map();
async function getModel() {
  if (cameraInfoCache.has('model')) {
    return cameraInfoCache.get('model');
  }
  const model = await fetchModelFromCamera();
  cameraInfoCache.set('model', model);
  return model;
}
```

---

### 7. Concurrency & Parallelization

**Check for**:
- Sequential image processing that could be parallel
- No worker threads for CPU-intensive operations
- Single-threaded batch processing
- Blocking operations on main thread

**Questions**:
- Can batch DNG processing use multiple cores?
- Should USB communication be in a Web Worker?
- Are CPU-intensive operations blocking UI?

**Optimization opportunities**:
- Use Worker threads for parallel DNG processing
- Process multiple files concurrently (with limit)
- Move USB operations to Web Worker (if possible)
- Use async/await properly to avoid blocking

**Example Issue**:
```javascript
// Bad: Sequential processing
async function processBatch(files) {
  for (const file of files) {
    await processFile(file);  // One at a time
  }
}

// Better: Parallel with concurrency limit
async function processBatch(files) {
  const CONCURRENCY = 4;
  const chunks = chunkArray(files, CONCURRENCY);
  for (const chunk of chunks) {
    await Promise.all(chunk.map(processFile));
  }
}
```

---

## Output Format

For each issue found:

**Location**: [file:line or component]

**Issue**: [description of performance problem]

**Category**: [USB Communication | Image Processing | Browser | Algorithm | Memory | Caching | Concurrency]

**Impact**:
- **High**: Causes poor UX, timeouts, or crashes
- **Medium**: Noticeable slowdown but usable
- **Low**: Minor inefficiency

**Current Performance**: [e.g., "Takes 10s to process one image", "3 USB round-trips per operation"]

**Optimization**: [how to fix it]

**Expected Improvement**: [rough estimate, e.g., "50% faster", "Reduce memory by 80%"]

**Effort**:
- **Low**: Simple code change
- **Medium**: Refactoring required
- **High**: Architectural change or new dependencies

**Priority**: [Impact/Effort ratio - High/Medium/Low]

---

## Example Report

### Issue 1: Sequential USB Command Calls

**Location**: dxo1usb.js:125-135

**Issue**: Getting camera info makes 3 sequential USB calls (battery, model, firmware)

**Category**: USB Communication

**Impact**: Medium (adds 200-300ms latency)

**Current Performance**: ~300ms total (100ms per USB call)

**Optimization**: Batch commands into single USB transfer if protocol supports it, or at minimum parallelize independent calls

**Expected Improvement**: 66% faster (100ms instead of 300ms)

**Effort**: Low (use Promise.all for parallel calls)

**Priority**: High

---

### Issue 2: Loading Entire DNG File Into Memory

**Location**: resizeDNG.mjs:45

**Issue**: Using `readFileSync()` loads entire 25MB file into memory at once

**Category**: Image Processing, Memory

**Impact**: High (causes OOM for large batches or low-memory systems)

**Current Performance**: Uses 25MB+ per file, crashes on batch of 10+ files

**Optimization**: Stream file processing chunk-by-chunk

**Expected Improvement**: Reduce peak memory by 80%+

**Effort**: Medium (requires refactoring to streaming approach)

**Priority**: High

---

## Priorities

Order by **Impact / Effort**:

1. **High Priority** (High Impact, Low Effort): Do these first
2. **Medium Priority** (Medium Impact, Low-Medium Effort): Do after high
3. **Low Priority** (Low Impact or High Effort): Consider for later

---

## Remember

- Don't optimize prematurely - focus on actual bottlenecks
- Measure before and after if possible
- Sometimes "good enough" is better than "optimal but complex"
- User experience matters more than theoretical performance
- For dxo1control: Camera safety and data integrity > performance

**USB commands must remain correct** (INV-DATA-001) even if batching
**Colorspace must be preserved** (INV-DATA-002) even if caching results

---

**Related Documentation**:
- [ARCHITECTURE.md](../docs/ARCHITECTURE.md) - Performance considerations
- [INVARIANTS.md](../docs/INVARIANTS.md) - Constraints during optimization
