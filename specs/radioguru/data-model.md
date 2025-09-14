# Data Model: Dual Processor RC Controller System

## Core Entities

### RadioConfiguration
**Purpose**: Stores all radio-related settings and mixer configurations
**Fields**:
- `model_id`: u16 - Unique identifier for aircraft model
- `model_name`: String - Human-readable model name  
- `mixer_settings`: MixerConfig - Channel mixing configuration
- `channel_map`: ChannelMap - Input to output channel mapping
- `failsafe_config`: FailsafeConfig - Behavior when signal lost
- `flight_modes`: Vec<FlightMode> - Available flight mode configurations
- `trim_values`: TrimConfig - Stick trim adjustments
- `expo_curves`: ExpoConfig - Exponential response curves
- `created_at`: Timestamp
- `updated_at`: Timestamp

**Validation Rules**:
- model_id must be unique across all configurations
- channel_map must not have duplicate output assignments
- failsafe_config must specify valid channel positions
- flight_modes must have at least one default mode

**State Transitions**:
- Draft → Active (when configuration validated and applied)
- Active → Modified (when settings changed)
- Modified → Active (when changes applied)
- Active → Archived (when model deleted)

### ControlInput
**Purpose**: Real-time control input data from gimbals and switches
**Fields**:
- `gimbal_positions`: [i16; 4] - Raw ADC values for primary sticks
- `aux_channels`: [i16; 8] - Additional analog/digital inputs
- `switch_states`: u16 - Bit field for discrete switch positions
- `timestamp_us`: u64 - Microsecond timestamp for latency tracking
- `sequence_number`: u32 - Monotonic counter for lost packet detection

**Validation Rules**:
- gimbal_positions must be within calibrated min/max ranges
- timestamp_us must be monotonically increasing
- sequence_number must increment for each sample

### VideoStream
**Purpose**: Video feed configuration and metadata
**Fields**:
- `source_type`: VideoSourceType - Analog/Digital/USB/HDMI
- `source_id`: String - Hardware identifier
- `resolution`: Resolution - Width/height configuration
- `frame_rate`: u8 - Target frames per second
- `format`: VideoFormat - Pixel format and encoding
- `is_active`: bool - Whether stream is currently displayed
- `latency_ms`: u16 - Measured display latency
- `quality_metrics`: QualityMetrics - Signal strength, dropped frames

**Validation Rules**:
- Only one stream can be active at a time
- resolution must be supported by source device
- frame_rate must not exceed source capability

### SystemSettings
**Purpose**: Application-level configuration and preferences
**Fields**:
- `wifi_config`: WiFiConfig - Network credentials and settings
- `bluetooth_config`: BluetoothConfig - Paired devices and preferences
- `display_settings`: DisplayConfig - Brightness, orientation, theme
- `audio_settings`: AudioConfig - Volume levels, input/output devices
- `calibration_data`: CalibrationData - Gimbal and switch calibration
- `user_preferences`: UserPrefs - Language, units, shortcuts

**Validation Rules**:
- wifi_config passwords must be encrypted at rest
- calibration_data must include min/max/center values for all inputs
- display_settings brightness must be 0-100 range

### HardwareProfile
**Purpose**: Connected hardware and expansion module configuration
**Fields**:
- `radio_module`: RadioModuleInfo - JR bay module information
- `video_receivers`: Vec<VideoReceiverInfo> - Connected VRX devices
- `expansion_modules`: Vec<ExpansionModuleInfo> - Additional hardware
- `gpio_assignments`: GPIOConfig - Pin function assignments
- `communication_config`: CommConfig - UART/I2C/SPI settings
- `firmware_versions`: FirmwareInfo - Version tracking for all components

**Validation Rules**:
- radio_module must be compatible with JR bay standard
- gpio_assignments must not have conflicting pin usage
- firmware_versions must track compatibility matrix

## Relationships

### RadioConfiguration ↔ ControlInput
- One active RadioConfiguration processes multiple ControlInput samples
- ControlInput values are transformed by RadioConfiguration mixer settings
- Changes to RadioConfiguration affect ControlInput processing immediately

### VideoStream ↔ HardwareProfile  
- VideoStream sources must exist in HardwareProfile.video_receivers
- HardwareProfile changes can invalidate active VideoStream
- VideoStream quality depends on HardwareProfile configuration

### SystemSettings ↔ All Entities
- SystemSettings.calibration_data affects ControlInput processing
- SystemSettings.display_settings affects VideoStream rendering
- SystemSettings preferences influence all user interactions

## Data Flow Patterns

### High-Priority Path (Radio Control)
```
ControlInput → RadioConfiguration.mixer_settings → OutputSignals
- Sub-2ms latency requirement
- No blocking operations allowed
- Direct memory access to hardware registers
```

### Medium-Priority Path (Configuration)
```
SystemSettings → RadioConfiguration → PersistentStorage
- <100ms update latency acceptable
- Validation before applying changes
- Rollback capability for invalid configurations
```

### Low-Priority Path (Monitoring)
```
All Entities → LoggingSystem → DiagnosticData
- Best-effort delivery
- Buffered for batch processing
- Non-blocking to higher priority operations
```

## Persistence Strategy

### Microcontroller (Flash/EEPROM)
- RadioConfiguration active settings
- Failsafe configuration (critical)
- Calibration data backup
- Communication protocol state

### Application Processor (Filesystem)
- Complete RadioConfiguration database
- VideoStream preferences and history
- SystemSettings and user data
- Logs and diagnostic information
- Firmware update packages

### Synchronization Protocol
- Periodic sync of critical settings to microcontroller
- Change notification from AP to microcontroller
- Conflict resolution for concurrent modifications
- Backup/restore capability for system recovery
