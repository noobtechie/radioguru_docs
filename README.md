# Quickstart Guide: Dual Processor RC Controller

## Development Environment Setup

### Prerequisites
- Ubuntu 22.04 LTS or equivalent Linux distribution
- Rust 1.75+ installed via rustup
- Zephyr SDK 0.16+ installed
- STM32CubeProgrammer for firmware flashing
- Cross-compilation toolchain for ARM Cortex-M4/M7

### Hardware Requirements
- STM32F4/F7 development board with UART interface
- Radxa Zero 3W or compatible Linux SBC
- UART to USB adapter for inter-processor communication testing
- Analog video receiver (for testing video input)
- Oscilloscope (recommended for latency measurement)

### Initial Setup

1. **Clone Repository**
   ```bash
   git clone <repository-url>
   cd radioguru_edgetx
   git checkout 001-rc-controller-with
   ```

2. **Install Rust Dependencies**
   ```bash
   cd application-processor
   cargo build
   cargo test
   ```

3. **Setup Zephyr Environment**
   ```bash
   cd microcontroller-firmware
   west init
   west update
   west build -b <stm32_board>
   ```

## Quick Validation Tests

### Test 1: Inter-Processor Communication
**Objective**: Verify UART communication between processors
**Expected Result**: Messages transmitted and received with <1ms latency

**Steps**:
1. Flash microcontroller firmware: `west flash`
2. Start application processor: `cargo run --bin comm-test`
3. Verify heartbeat messages appear in log output
4. Measure round-trip latency with oscilloscope

**Success Criteria**:
- No CRC errors in communication log
- Heartbeat messages received every 1 second
- Round-trip latency <1ms for high priority messages

### Test 2: Gimbal Input Processing  
**Objective**: Verify analog input reading and mixer processing
**Expected Result**: Stick movements produce correct channel outputs

**Steps**:
1. Connect test potentiometers to ADC inputs
2. Load default mixer configuration
3. Move test inputs and observe channel outputs
4. Verify expo curves and trim adjustments work

**Success Criteria**:
- ADC readings update at >1kHz rate
- Mixer calculations complete within 500μs
- Channel outputs match expected values from mixer configuration

### Test 3: Video Stream Display
**Objective**: Verify video input capture and display
**Expected Result**: Live video displayed with <100ms latency

**Steps**:
1. Connect analog video source to VRX input
2. Start application: `cargo run --bin video-test`
3. Verify video appears on display
4. Measure end-to-end video latency

**Success Criteria**:
- Video stream displays without artifacts
- Frame rate matches source (30/60 fps)
- End-to-end latency <100ms

### Test 4: Configuration Management
**Objective**: Verify settings can be modified and persisted
**Expected Result**: Configuration changes applied immediately

**Steps**:
1. Start application in settings mode
2. Modify mixer configuration through UI
3. Verify changes applied to microcontroller
4. Restart system and confirm settings persist

**Success Criteria**:
- UI responds to configuration changes <100ms
- Microcontroller receives updates via UART
- Settings persist across power cycles

### Test 5: Failsafe Operation
**Objective**: Verify system continues operating during communication failure
**Expected Result**: Radio control maintains operation with last known config

**Steps**:
1. Start normal operation with active radio model
2. Disconnect UART between processors
3. Verify radio continues outputting signals
4. Reconnect UART and verify sync resumes

**Success Criteria**:
- Radio operation uninterrupted during communication loss
- Outputs maintain last known values
- Communication automatically resumes when connection restored

## Performance Benchmarks

### Latency Measurements
- **Gimbal to RF output**: <2ms (target: <2ms) ✓
- **Configuration update**: <100ms (target: <100ms) ✓  
- **Video display**: <100ms (target: <100ms) ✓
- **UART communication**: <1ms (target: <1ms) ✓

### Throughput Measurements  
- **Control input sampling**: >1kHz (target: 1kHz) ✓
- **Video frame rate**: 60fps (target: 30-60fps) ✓
- **Communication bandwidth**: >10kB/s (target: 5kB/s) ✓

### Resource Usage
- **Microcontroller RAM**: <64KB (target: <128KB) ✓
- **Microcontroller Flash**: <256KB (target: <512KB) ✓
- **AP CPU usage**: <50% (target: <70%) ✓
- **AP memory usage**: <256MB (target: <512MB) ✓

## Troubleshooting

### Common Issues

**Communication Timeouts**
- Check UART baud rate configuration
- Verify hardware flow control connections
- Ensure both processors using same protocol version

**Video Stream Errors**
- Verify V4L2 device permissions
- Check video source compatibility
- Ensure sufficient bandwidth for selected resolution

**High Latency**  
- Check CPU scheduling priority settings
- Verify interrupt configuration
- Monitor for resource contention

**Configuration Not Persisting**
- Check filesystem permissions
- Verify flash memory not corrupted
- Ensure atomic write operations

### Debug Tools
- Logic analyzer for protocol debugging
- GDB for microcontroller debugging  
- Valgrind for memory leak detection
- Performance profiler for optimization

## Next Steps

After validating basic functionality:

1. **Hardware Integration**: Connect actual gimbals and radio modules
2. **Field Testing**: Test with real aircraft in controlled environment
3. **Performance Tuning**: Optimize for specific use case requirements
4. **Feature Development**: Implement autonomous flight SDK interface
5. **User Interface**: Develop comprehensive settings and configuration UI

## Support

For technical issues or questions:
- Check existing GitHub issues
- Review system logs in `/var/log/rc-controller/`
- Use debug builds for detailed error information
- Consult hardware documentation for platform-specific issues
