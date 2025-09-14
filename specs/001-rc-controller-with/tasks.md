# Tasks: RC Controller MVP Prototype

**Input**: Design documents from `/specs/001-rc-controller-with/`
**Prerequisites**: plan.md (required), research.md, data-model.md, contracts/

**MVP Hardware Context**: STM32 Nucleo F446RE, Radxa Zero 3W, USB-UART cable, IP2312 charging board, LM2596 regulator, 18650 battery, HappyModel ES24TX, repurposed Flysky FS-i6s gimbals/switches, JHEMCU ELRS RX24T, MG905 servo, 4S LiPo battery

## Project Structure
```
microcontroller-firmware/         # STM32 Nucleo F446RE Zephyr project
├── src/
├── tests/
├── boards/nucleo_f446re/
└── zephyr/

application-processor/             # Radxa Zero 3W Rust application  
├── src/
├── tests/
└── hardware/

hardware-integration/              # MVP-specific hardware configs
├── power-management/
├── gimbal-calibration/
└── radio-modules/

tests/
├── contract/
├── integration/
└── hardware/
```

## Phase 3.1: MVP Hardware Setup
- [ ] T001 Create MVP project structure for dual processor system
- [ ] T002 Initialize STM32 Zephyr project for Nucleo F446RE board
- [ ] T003 Initialize Rust project for Radxa Zero 3W application processor  
- [ ] T004 [P] Configure power management for IP2312 + LM2596 + 18650 setup
- [ ] T005 [P] Configure STM32CubeProgrammer for Nucleo F446RE flashing
- [ ] T006 [P] Set up cross-compilation toolchain for ARM Cortex-M4

## Phase 3.2: Hardware Tests First (TDD) ⚠️ MUST COMPLETE BEFORE 3.3
**CRITICAL: These tests MUST be written and MUST FAIL before ANY implementation**

### Communication Protocol Tests
- [ ] T007 [P] Contract test UART heartbeat message in tests/contract/test_heartbeat_protocol.c
- [ ] T008 [P] Contract test control input message format in tests/contract/test_control_input_protocol.c  
- [ ] T009 [P] Contract test configuration update protocol in tests/contract/test_config_update_protocol.c
- [ ] T010 [P] Contract test error reporting protocol in tests/contract/test_error_protocol.c

### Hardware Interface Tests  
- [ ] T011 [P] Contract test gimbal ADC reading in tests/contract/test_gimbal_adc.c
- [ ] T012 [P] Contract test switch GPIO states in tests/contract/test_switch_gpio.c
- [ ] T013 [P] Contract test ES24TX UART interface in tests/contract/test_es24tx_uart.c
- [ ] T014 [P] Contract test servo PWM output in tests/contract/test_servo_pwm.c

### Application Processor API Tests
- [ ] T015 [P] Contract test GET /radio/models in tests/contract/test_radio_models_get.rs
- [ ] T016 [P] Contract test PUT /radio/models/{id} in tests/contract/test_radio_models_put.rs
- [ ] T017 [P] Contract test POST /radio/models/{id}/activate in tests/contract/test_radio_activate.rs
- [ ] T018 [P] Contract test GET /system/settings in tests/contract/test_system_settings.rs

### Integration Tests
- [ ] T019 [P] Integration test gimbal-to-servo control loop in tests/integration/test_control_loop.rs
- [ ] T020 [P] Integration test UART communication between processors in tests/integration/test_uart_comm.rs
- [ ] T021 [P] Integration test configuration sync between processors in tests/integration/test_config_sync.rs
- [ ] T022 [P] Integration test ELRS radio binding and control in tests/integration/test_elrs_radio.rs

## Phase 3.3: Core Implementation (ONLY after tests are failing)

### Data Models (Parallel Implementation)
- [ ] T023 [P] RadioConfiguration model in microcontroller-firmware/src/models/radio_config.c
- [ ] T024 [P] ControlInput model in microcontroller-firmware/src/models/control_input.c
- [ ] T025 [P] SystemSettings model in application-processor/src/models/system_settings.rs
- [ ] T026 [P] HardwareProfile model in application-processor/src/models/hardware_profile.rs
- [ ] T027 [P] VideoStream model in application-processor/src/models/video_stream.rs

### Communication Protocol Implementation  
- [ ] T028 [P] UART protocol message encoding in microcontroller-firmware/src/comm/protocol.c
- [ ] T029 [P] UART protocol message decoding in application-processor/src/comm/protocol.rs
- [ ] T030 Message priority queue implementation in microcontroller-firmware/src/comm/message_queue.c
- [ ] T031 Communication error handling and recovery in application-processor/src/comm/error_handler.rs

### Hardware Abstraction Layer
- [ ] T032 [P] Gimbal ADC driver for Flysky FS-i6s inputs in microcontroller-firmware/src/hal/gimbal_adc.c
- [ ] T033 [P] Switch GPIO driver for FS-i6s switches in microcontroller-firmware/src/hal/switch_gpio.c  
- [ ] T034 [P] ES24TX UART driver interface in microcontroller-firmware/src/hal/es24tx_uart.c
- [ ] T035 [P] Servo PWM driver for MG905 in microcontroller-firmware/src/hal/servo_pwm.c
- [ ] T036 [P] Power management for IP2312/LM2596 in microcontroller-firmware/src/hal/power_mgmt.c

### Core Services
- [ ] T037 Real-time mixer service in microcontroller-firmware/src/services/mixer_service.c
- [ ] T038 Failsafe handler service in microcontroller-firmware/src/services/failsafe_service.c
- [ ] T039 Configuration manager in application-processor/src/services/config_manager.rs
- [ ] T040 Hardware detection service in application-processor/src/services/hardware_detector.rs

### API Endpoints (Sequential - shared web server)
- [ ] T041 GET /radio/models endpoint implementation
- [ ] T042 PUT /radio/models/{id} endpoint implementation  
- [ ] T043 POST /radio/models/{id}/activate endpoint implementation
- [ ] T044 GET /system/settings endpoint implementation
- [ ] T045 Input validation middleware for all endpoints
- [ ] T046 Error handling and logging for API layer

## Phase 3.4: MVP Integration

### Hardware Integration
- [ ] T047 Integrate gimbal calibration for FS-i6s hardware in hardware-integration/gimbal-calibration/
- [ ] T048 Configure ES24TX module for ELRS protocol compatibility  
- [ ] T049 Set up JHEMCU ELRS RX24T binding and test with 4S LiPo
- [ ] T050 Implement power sequencing for IP2312 charging board
- [ ] T051 Configure LM2596 voltage regulation for stable 5V/3.3V rails

### System Integration  
- [ ] T052 Connect RadioConfiguration service to UART communication
- [ ] T053 Implement configuration persistence to STM32 flash memory
- [ ] T054 Add structured logging from microcontroller to application processor
- [ ] T055 Implement watchdog timers for both processors
- [ ] T056 Add system health monitoring and diagnostics

### Safety and Failsafe
- [ ] T057 Implement communication timeout failsafe mode
- [ ] T058 Add battery voltage monitoring and low-battery warnings
- [ ] T059 Implement emergency stop functionality
- [ ] T060 Add servo position limits and sanity checks

## Phase 3.5: MVP Polish and Validation

### Performance Optimization
- [ ] T061 [P] Optimize ADC sampling rate for <2ms latency requirement
- [ ] T062 [P] Tune UART baud rate for optimal communication throughput  
- [ ] T063 [P] Implement zero-copy message passing where possible
- [ ] T064 [P] Add CPU usage monitoring and performance profiling

### Testing and Validation
- [ ] T065 [P] Unit tests for mixer calculations in tests/unit/test_mixer_math.c
- [ ] T066 [P] Unit tests for protocol message validation in tests/unit/test_protocol_validation.rs
- [ ] T067 [P] Performance tests for <2ms control loop latency
- [ ] T068 [P] Hardware-in-the-loop tests with actual servo movement
- [ ] T069 Run complete MVP validation from quickstart.md scenarios

### Documentation and CLI Tools  
- [ ] T070 [P] CLI tool for radio configuration in application-processor/src/cli/radio_config_cli.rs
- [ ] T071 [P] CLI tool for system diagnostics in application-processor/src/cli/diagnostics_cli.rs
- [ ] T072 [P] Update hardware setup documentation for MVP components
- [ ] T073 [P] Create troubleshooting guide for common MVP issues

## Dependencies

**Setup Phase**: T001-T006 must complete before any other tasks
**Tests Phase**: T007-T022 must complete and FAIL before implementation starts  
**Models Phase**: T023-T027 can run in parallel, block services (T037-T040)
**Protocol Phase**: T028-T031 sequential due to shared protocol definition
**HAL Phase**: T032-T036 can run in parallel, different hardware components
**Services Phase**: T037-T040 depend on models, block API endpoints
**API Phase**: T041-T046 sequential due to shared web server
**Integration Phase**: T047-T060 depend on core implementation completion
**Polish Phase**: T061-T073 can mostly run in parallel after integration

## Parallel Execution Examples

### Phase 3.2 Tests (All Parallel)
```bash
# Launch contract tests together:
Task: "Contract test UART heartbeat message in tests/contract/test_heartbeat_protocol.c"
Task: "Contract test gimbal ADC reading in tests/contract/test_gimbal_adc.c" 
Task: "Contract test GET /radio/models in tests/contract/test_radio_models_get.rs"
Task: "Integration test gimbal-to-servo control loop in tests/integration/test_control_loop.rs"
```

### Phase 3.3 Models (All Parallel)  
```bash
# Launch model creation together:
Task: "RadioConfiguration model in microcontroller-firmware/src/models/radio_config.c"
Task: "ControlInput model in microcontroller-firmware/src/models/control_input.c"
Task: "SystemSettings model in application-processor/src/models/system_settings.rs"
```

### Phase 3.3 HAL Drivers (All Parallel)
```bash
# Launch hardware drivers together:
Task: "Gimbal ADC driver for Flysky FS-i6s inputs in microcontroller-firmware/src/hal/gimbal_adc.c"
Task: "Switch GPIO driver for FS-i6s switches in microcontroller-firmware/src/hal/switch_gpio.c"
Task: "ES24TX UART driver interface in microcontroller-firmware/src/hal/es24tx_uart.c"
```

## Hardware-Specific Notes

### STM32 Nucleo F446RE Configuration
- Use TIM2 for servo PWM generation (MG905)
- Configure USART2 for ES24TX communication  
- Use ADC1 channels for gimbal position sensing
- GPIO pins for switch state detection

### Radxa Zero 3W Configuration  
- Use /dev/ttyS1 for UART communication to STM32
- Configure GPIO for hardware status LEDs
- Set up power management for 18650 battery monitoring
- Ensure proper voltage levels for UART communication

### Power Management Setup
- IP2312: USB-C charging with battery protection
- LM2596: Buck converter for stable 5V rail
- 18650: Primary power source with voltage monitoring
- 4S LiPo: High-power source for servo operation

## Validation Checklist
*GATE: All items must pass before MVP completion*

- [ ] All protocol contract tests pass with actual hardware
- [ ] Gimbal movements translate to correct servo positions
- [ ] UART communication maintains <1ms latency under load
- [ ] ES24TX module successfully binds and controls ELRS receiver
- [ ] Battery monitoring accurately reports charge levels
- [ ] Failsafe mode activates correctly during communication loss
- [ ] Configuration changes persist across power cycles
- [ ] System operates reliably from 18650 battery power
- [ ] All hardware components integrate without conflicts
- [ ] Performance meets <2ms control loop latency requirement
