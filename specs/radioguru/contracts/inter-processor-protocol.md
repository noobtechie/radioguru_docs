# Inter-Processor Communication Protocol

## Overview
Binary protocol over UART optimized for minimal latency and reliable delivery.

## Message Format
```
Header (8 bytes):
- magic: u16 (0xA5C3) - Protocol identifier
- msg_type: u8 - Message type identifier  
- priority: u8 - Priority level (0=high, 1=medium, 2=low)
- payload_len: u16 - Length of payload in bytes
- seq_num: u8 - Sequence number for ordering
- crc8: u8 - Header checksum

Payload (0-255 bytes):
- Variable length based on message type
- Specific format defined per message type

Footer (2 bytes):
- crc16: u16 - Payload checksum
```

## Message Types

### Control Input (Type: 0x01, Priority: High)
```rust
struct ControlInputMessage {
    timestamp_us: u64,
    gimbal_positions: [i16; 4],  // Aileron, Elevator, Throttle, Rudder
    aux_channels: [i16; 8],      // Additional analog inputs
    switch_states: u16,          // Bit field for switches
    sequence_number: u32,
}
```

### Configuration Update (Type: 0x02, Priority: Medium)  
```rust
struct ConfigUpdateMessage {
    config_type: u8,     // RadioConfig=1, MixerConfig=2, etc.
    config_id: u16,      // Specific configuration identifier
    data_offset: u16,    // Offset within configuration
    data_length: u8,     // Length of data payload
    data: [u8; 247],     // Configuration data
}
```

### Status Request (Type: 0x03, Priority: Low)
```rust
struct StatusRequestMessage {
    request_type: u8,    // SystemStatus=1, Diagnostics=2, etc.
    request_id: u16,     // Request identifier for correlation
}
```

### Status Response (Type: 0x04, Priority: Low)
```rust
struct StatusResponseMessage {
    request_id: u16,     // Correlates with StatusRequest
    status_type: u8,     // Type of status data
    data_length: u8,     // Length of status payload
    data: [u8; 251],     // Status data payload
}
```

### Error Report (Type: 0x05, Priority: High)
```rust
struct ErrorReportMessage {
    error_code: u16,     // Standardized error identifier
    error_source: u8,    // Hardware=1, Software=2, Communication=3
    timestamp_us: u64,   // When error occurred
    context_data: [u8; 243], // Additional error context
}
```

### Heartbeat (Type: 0x06, Priority: Low)
```rust
struct HeartbeatMessage {
    sender_id: u8,       // Microcontroller=1, AP=2
    timestamp_us: u64,   // Sender timestamp
    system_state: u8,    // Operational state
    reserved: [u8; 246], // Future expansion
}
```

## Protocol Rules

### Timing Constraints
- Control Input messages: <1ms processing time
- Configuration Update: <10ms acknowledgment  
- Status messages: Best effort, no timing guarantee
- Heartbeat interval: 1 second maximum

### Error Handling
- CRC mismatch: Request retransmission
- Sequence gap: Log but continue processing
- Timeout: Sender retries up to 3 times
- Critical failure: Switch to failsafe mode

### Flow Control
- Hardware flow control (RTS/CTS) enabled
- Message acknowledgment for medium/low priority
- High priority messages are fire-and-forget
- Buffer overflow triggers priority-based message dropping

### Priority Handling
- High priority bypasses all queues
- Medium priority uses dedicated queue with timeout
- Low priority best-effort delivery
- Queue sizes: High=unlimited, Medium=16, Low=8
