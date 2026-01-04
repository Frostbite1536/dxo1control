# Architecture

## Overview

**dxo1control** is a collection of tools for controlling and managing DXO One cameras. The project consists of two main components:

1. **USB Control Interface** - Programmatic control of DXO One cameras over USB connection
2. **Post-processing Tools** - Image processing utilities for DNG files from the camera

**Type**: Library/Tools Collection
**Primary Users**: Developers and photographers using DXO One cameras
**Environment**: Node.js for backend tools, Browser for WebUSB interface

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    dxo1control                          │
├─────────────────────────┬───────────────────────────────┤
│   USB Control Layer     │   Post-Processing Layer       │
│                         │                               │
│  ┌──────────────┐      │   ┌────────────────┐         │
│  │  dxo1usb.js  │      │   │ resizeDNG.mjs  │         │
│  │              │      │   │                │         │
│  │ - WebUSB API │      │   │ - DNG Reader   │         │
│  │ - Camera Cmd │      │   │ - Image Resize │         │
│  │ - u8a utils  │      │   │ - JPG Export   │         │
│  └──────────────┘      │   └────────────────┘         │
│         │               │          │                    │
│         ▼               │          ▼                    │
│  ┌──────────────┐      │   ┌────────────────┐         │
│  │  usb.html    │      │   │  File System   │         │
│  │ (Demo/UI)    │      │   │                │         │
│  └──────────────┘      │   └────────────────┘         │
└─────────────────────────┴───────────────────────────────┘
         │                              │
         ▼                              ▼
  ┌──────────────┐            ┌──────────────────┐
  │  DXO One     │            │  DNG/JPG Files   │
  │  Camera      │            │                  │
  │  (microUSB)  │            │                  │
  └──────────────┘            └──────────────────┘
```

## Component Breakdown

### 1. USB Control Layer (`dxo1usb.js`)

**Responsibilities:**
- Establish and manage WebUSB connection to DXO One camera
- Send commands to camera (capture, settings, etc.)
- Handle camera responses and events
- Provide programmatic API for camera control

**Dependencies:**
- WebUSB API (browser standard)
- `u8a.js` for Uint8Array utilities

**Key Interfaces:**
- USB communication protocol specific to DXO One
- Command/response message format

### 2. Utility Layer (`u8a.js`)

**Responsibilities:**
- Uint8Array manipulation utilities
- Data conversion helpers
- Binary data handling for USB communication

**Dependencies:** None (pure JavaScript utilities)

### 3. Web Interface (`usb.html`)

**Responsibilities:**
- Provide user interface for camera control
- Demonstrate WebUSB API usage
- Live camera control demo

**Dependencies:**
- `dxo1usb.js`
- WebUSB-compatible browser

### 4. Post-Processing Tool (`resizeDNG.mjs`)

**Responsibilities:**
- Read DNG (Digital Negative) files from DXO One
- Resize images while preserving quality
- Convert to JPG without modifying colorspace
- Batch processing capabilities

**Dependencies:**
- Node.js runtime
- Image processing libraries [TBD: Specific libraries to be documented]

## Data Models

### USB Command Message
```javascript
{
  type: Uint8Array,        // Command type identifier
  payload: Uint8Array,     // Command-specific data
  length: Number           // Message length
}
```

### Camera Response
```javascript
{
  status: Number,          // Response status code
  data: Uint8Array,        // Response payload
  timestamp: Number        // Response timestamp
}
```

### DNG File Metadata
```javascript
{
  width: Number,           // Image width in pixels
  height: Number,          // Image height in pixels
  colorspace: String,      // Original colorspace (preserved)
  rawData: ArrayBuffer     // Raw image data
}
```

## External Dependencies

### WebUSB API
- **Purpose**: Browser API for USB device communication
- **Failure Mode**: Graceful degradation - show error message if not supported
- **Constraint**: Only works in WebUSB-compatible browsers (Chrome, Edge, Opera)
- **Constraint**: Only tested with microUSB connection (Lightning not tested)

### Node.js Runtime
- **Purpose**: Execute post-processing scripts
- **Failure Mode**: Require Node.js installation for CLI tools
- **Version Requirements**: [TBD: Minimum Node.js version]

### Image Processing Libraries
- **Purpose**: DNG parsing and JPG conversion
- **Libraries**: [TBD: Document specific libraries used in resizeDNG.mjs]

## Key Design Decisions

### Decision: WebUSB for Camera Control
**Rationale**: Enables browser-based camera control without native drivers or applications. Provides cross-platform compatibility and easy demo deployment.

**Trade-offs:**
- ✅ No driver installation required
- ✅ Cross-platform (any WebUSB browser)
- ✅ Easy to demonstrate and share
- ❌ Browser-only (no native app support)
- ❌ Limited to WebUSB-compatible browsers

### Decision: Separate USB and Post-Processing Tools
**Rationale**: Different use cases with different runtime requirements. USB control needs browser, post-processing needs Node.js.

**Trade-offs:**
- ✅ Clear separation of concerns
- ✅ Each tool optimized for its environment
- ✅ Can use tools independently
- ❌ No integrated workflow (yet)
- ❌ Different runtime requirements

### Decision: microUSB Connection Only
**Rationale**: Based on community research (rickdeck/DxO-One), microUSB protocol is understood. Lightning connection not yet reverse-engineered.

**Trade-offs:**
- ✅ Working implementation available now
- ✅ Builds on existing community knowledge
- ❌ Doesn't support Lightning connector
- ❌ Warning needed in documentation

## Security Considerations

### USB Access Control
- **Risk**: Malicious websites could request USB access
- **Mitigation**: WebUSB requires explicit user permission per session
- **Mitigation**: Connection only works with DXO One vendor/product ID

### File Processing
- **Risk**: Processing malicious DNG files could cause crashes or exploits
- **Mitigation**: [TBD: Input validation for DNG files]
- **Mitigation**: [TBD: Sandboxing for file processing]

### Command Injection
- **Risk**: Invalid commands could damage camera or cause undefined behavior
- **Mitigation**: Validate all commands before sending to camera
- **Mitigation**: Document safe command set and boundaries

## Performance Considerations

### USB Communication
- **Latency**: USB 2.0 communication latency varies by host
- **Throughput**: Limited by USB 2.0 bandwidth for image transfer
- **Optimization**: [TBD: Implement command queuing if needed]

### Image Processing
- **Memory**: Large DNG files (20+ MB) require sufficient RAM
- **CPU**: Image resizing is CPU-intensive
- **Optimization**: [TBD: Consider streaming/chunked processing]
- **Optimization**: [TBD: Parallel processing for batch operations]

### Browser Performance
- **Memory**: Keep connection state minimal
- **UI**: Ensure camera commands don't block UI thread
- **Optimization**: Use Web Workers if heavy processing needed

## Open Questions

- [ ] What are the minimum Node.js and npm version requirements?
- [ ] Which specific image processing libraries does resizeDNG.mjs use?
- [ ] What is the complete USB command protocol for DXO One?
- [ ] Are there plans to support Lightning connector?
- [ ] Should there be integration between USB control and post-processing?
- [ ] What are the camera's supported image formats and settings?

## Future Considerations

### Potential Enhancements
1. Native desktop application (Electron) for integrated workflow
2. Lightning connector support (requires protocol reverse-engineering)
3. Real-time preview during USB control
4. Batch operation UI for post-processing
5. Camera settings management and persistence
6. Firmware update capabilities (if safe to implement)

### Integration Opportunities
1. Integration with photo management software
2. Plugin architecture for custom post-processing filters
3. Remote camera control over network
4. Mobile app using WebUSB on Android

---

**Last Updated**: 2026-01-04
**Document Version**: 1.0
**Status**: Initial architecture documentation
