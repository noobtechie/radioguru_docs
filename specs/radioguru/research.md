# Research: Dual Processor RC Controller System

## Technology Decisions

### Microcontroller Platform
**Decision**: STM32F4/F7 series with Zephyr RTOS
**Rationale**: 
- STM32F4/F7 provides sufficient processing power for real-time radio control
- Zephyr RTOS offers deterministic scheduling with priority-based preemption
- Strong ecosystem support for radio protocols and hardware interfaces
- Well-documented HAL for ADC, PWM, and communication peripherals
**Alternatives considered**: 
- FreeRTOS (less standardized, more manual configuration)
- Bare metal (no RTOS overhead but more complex scheduling)
- ESP32 (WiFi capability but less deterministic timing)

### Inter-Processor Communication
**Decision**: UART with custom binary protocol optimized for latency
**Rationale**:
- UART provides lowest latency among standard protocols
- Hardware flow control reduces communication errors
- Simple implementation on both processors
- Can achieve <1ms round-trip communication times
**Alternatives considered**:
- I2C (higher latency, more complex error handling)
- SPI (requires additional control lines, more complex protocol)
- USB (higher latency, more protocol overhead)

### Application Processor Platform
**Decision**: Radxa Zero 3W with Linux kernel 6.1+
**Rationale**:
- ARM Cortex-A55 quad-core provides sufficient performance for video processing
- Built-in WiFi/Bluetooth connectivity
- GPIO and I2C interfaces for communication with microcontroller
- V4L2 support for video capture devices
- Small form factor suitable for handheld controller
**Alternatives considered**:
- Raspberry Pi 4 (larger size, higher power consumption)
- Orange Pi Zero 2W (less community support)
- Custom Linux board (higher development cost)

### Application Development Framework
**Decision**: Rust with egui for UI and tokio for async runtime
**Rationale**:
- Rust provides memory safety critical for embedded applications
- egui offers immediate mode GUI suitable for settings interfaces
- tokio enables efficient async I/O for video streaming and communication
- Strong ecosystem for video processing and hardware interfaces
**Alternatives considered**:
- C++ with Qt (larger runtime overhead, more complex memory management)
- Python with tkinter (slower performance, not suitable for real-time video)
- Native C with GTK (more manual memory management, slower development)

### Video Processing
**Decision**: V4L2 for analog capture, USB Video Class for digital
**Rationale**:
- V4L2 is standard Linux interface for video capture
- Hardware acceleration available for common video formats
- USB Video Class provides plug-and-play compatibility
- GStreamer pipeline can handle format conversion and display
**Alternatives considered**:
- FFmpeg (more complex integration, higher CPU usage)
- Custom video drivers (significant development overhead)
- DirectShow on Windows (not Linux compatible)

### Communication Protocol Design
**Decision**: Binary message format with CRC error checking
**Rationale**:
- Binary format minimizes parsing overhead and message size
- CRC ensures data integrity over UART link
- Fixed-size headers enable efficient parsing
- Priority-based message queuing supports real-time requirements
**Alternatives considered**:
- JSON protocol (human readable but higher overhead)
- Protocol Buffers (efficient but more complex implementation)
- Custom text protocol (easier debugging but slower parsing)

### Real-Time Scheduling
**Decision**: Three priority levels - High (radio), Medium (config), Low (other)
**Rationale**:
- High priority ensures radio control meets <2ms latency requirement
- Medium priority allows configuration updates without blocking radio
- Low priority handles non-critical operations like logging
- Zephyr's priority-based preemption supports this model
**Alternatives considered**:
- Two priority levels (insufficient granularity for config updates)
- Rate monotonic scheduling (more complex, unnecessary for this application)
- Round-robin scheduling (doesn't guarantee real-time performance)

## Integration Patterns

### Hardware Abstraction
- Device tree configuration for hardware discovery
- HAL abstraction layer for cross-platform compatibility
- Plugin architecture for future radio module support
- Standardized connector interface (JR bay compliance)

### Error Handling
- Graceful degradation when communication fails
- Watchdog timers for both processors
- Fail-safe mode continues radio operation with last known configuration
- Error reporting through status LEDs and application interface

### Testing Strategy
- Hardware-in-the-loop testing with actual radio modules
- Mock hardware interfaces for unit testing
- Real-time performance validation with oscilloscope
- Integration testing with video capture devices

## Performance Considerations

### Latency Optimization
- Direct GPIO access for critical timing paths
- Interrupt-driven ADC sampling for gimbal inputs
- Zero-copy message passing where possible
- CPU affinity for time-critical threads

### Memory Management
- Static allocation for real-time paths
- Ring buffers for communication queues
- Memory pools for dynamic allocations
- Stack size optimization for embedded constraints

### Power Management
- Dynamic frequency scaling based on load
- Sleep modes when radio not active
- Efficient video codec selection
- Battery monitoring and low-power alerts
