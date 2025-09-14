# Implementation Plan: Dual Processor RC Controller System

**Branch**: `001-rc-controller-with` | **Date**: 2025-09-14 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/001-rc-controller-with/spec.md`

## Execution Flow (/plan command scope)
```
1. Load feature spec from Input path
   → If not found: ERROR "No feature spec at {path}"
2. Fill Technical Context (scan for NEEDS CLARIFICATION)
   → Detect Project Type from context (web=frontend+backend, mobile=app+api)
   → Set Structure Decision based on project type
3. Evaluate Constitution Check section below
   → If violations exist: Document in Complexity Tracking
   → If no justification possible: ERROR "Simplify approach first"
   → Update Progress Tracking: Initial Constitution Check
4. Execute Phase 0 → research.md
   → If NEEDS CLARIFICATION remain: ERROR "Resolve unknowns"
5. Execute Phase 1 → contracts, data-model.md, quickstart.md, agent-specific template file (e.g., `CLAUDE.md` for Claude Code, `.github/copilot-instructions.md` for GitHub Copilot, or `GEMINI.md` for Gemini CLI).
6. Re-evaluate Constitution Check section
   → If new violations: Refactor design, return to Phase 1
   → Update Progress Tracking: Post-Design Constitution Check
7. Plan Phase 2 → Describe task generation approach (DO NOT create tasks.md)
8. STOP - Ready for /tasks command
```

**IMPORTANT**: The /plan command STOPS at step 7. Phases 2-4 are executed by other commands:
- Phase 2: /tasks command creates tasks.md
- Phase 3-4: Implementation execution (manual or via tools)

## Summary
Dual processor RC controller system featuring an STM32 microcontroller running Zephyr RTOS for low-latency radio control and a Linux application processor running a Rust-based fly application with video display, settings management, and future autonomous flight SDK capabilities. The system provides standard JR bay compatibility, analog/digital video support, and expandable architecture.

## Technical Context
**Language/Version**: Rust 1.75+ (AP), C 17+ (Microcontroller), Zephyr RTOS  
**Primary Dependencies**: Zephyr RTOS, HAL drivers, Linux kernel, V4L2, ALSA, tokio/async-std, egui/tauri  
**Storage**: Flash storage for radio config, Linux filesystem for app data, EEPROM for failsafe config  
**Testing**: cargo test (Rust), Zephyr test framework, hardware-in-the-loop testing  
**Target Platform**: STM32 microcontroller + Linux SBC (Radxa Zero 3W)
**Project Type**: Embedded dual-processor system with real-time and application components  
**Performance Goals**: <2ms radio latency, 60fps video playback, <100ms settings response  
**Constraints**: Real-time radio control priority, deterministic timing, low power consumption  
**Scale/Scope**: Single device, dual processor architecture, extensible SDK interface

**Architecture Details from User**:
- Microcontroller: STM32 family with Zephyr RTOS
- Communication: UART/I2C/SPI optimized for lowest latency  
- Priority: Analog inputs (high) → Config changes (medium) → Other (low)
- API design: Wrapper-compatible for standalone LVGL display operation
- JR bay standard compliance required
- AP: Linux + Rust application with video processing capabilities
- Video inputs: Analog VRX + digital (USB-C/HDMI)
- Future SDK: AI autonomous flying, waypoint navigation support

## Constitution Check
*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

**Simplicity**:
- Projects: 2 (microcontroller firmware, application processor app) - within max 3 limit
- Using framework directly? Yes - Zephyr RTOS directly, Rust std libs directly
- Single data model? Yes - shared data structures between processors
- Avoiding patterns? Yes - direct hardware access, minimal abstraction layers

**Architecture**:
- EVERY feature as library? Yes - radio control lib, video lib, settings lib, communication lib
- Libraries listed: 
  - radio-control (mixer, failsafe, signal processing)
  - inter-proc-comm (UART/I2C protocol)
  - video-processing (analog/digital input handling)
  - settings-manager (configuration persistence)
  - autonomous-sdk (future extensibility interface)
- CLI per library: Yes - debug/config tools for each library component
- Library docs: llms.txt format planned for each component

**Testing (NON-NEGOTIABLE)**:
- RED-GREEN-Refactor cycle enforced? Yes - hardware tests must fail before implementation
- Git commits show tests before implementation? Yes - test-driven embedded development
- Order: Contract→Integration→E2E→Unit strictly followed? Yes
- Real dependencies used? Yes - actual hardware, real communication protocols
- Integration tests for: new libraries, contract changes, shared schemas? Yes
- FORBIDDEN: Implementation before test, skipping RED phase - strictly enforced

**Observability**:
- Structured logging included? Yes - both processors log to unified stream via communication link
- Frontend logs → backend? Yes - AP aggregates microcontroller logs
- Error context sufficient? Yes - detailed error reporting for debugging

**Versioning**:
- Version number assigned? 0.1.0 (initial implementation)
- BUILD increments on every change? Yes
- Breaking changes handled? Yes - protocol versioning, backward compatibility

## Project Structure

### Documentation (this feature)
```
specs/[###-feature]/
├── plan.md              # This file (/plan command output)
├── research.md          # Phase 0 output (/plan command)
├── data-model.md        # Phase 1 output (/plan command)
├── quickstart.md        # Phase 1 output (/plan command)
├── contracts/           # Phase 1 output (/plan command)
└── tasks.md             # Phase 2 output (/tasks command - NOT created by /plan)
```

### Source Code (repository root)
```
# Option 1: Single project (DEFAULT)
src/
├── models/
├── services/
├── cli/
└── lib/

tests/
├── contract/
├── integration/
└── unit/

# Option 2: Web application (when "frontend" + "backend" detected)
backend/
├── src/
│   ├── models/
│   ├── services/
│   └── api/
└── tests/

frontend/
├── src/
│   ├── components/
│   ├── pages/
│   └── services/
└── tests/

# Option 3: Mobile + API (when "iOS/Android" detected)
api/
└── [same as backend above]

ios/ or android/
└── [platform-specific structure]
```

**Structure Decision**: Embedded dual-processor system (custom structure for microcontroller + Linux application)

## Phase 0: Outline & Research
1. **Extract unknowns from Technical Context** above:
   - For each NEEDS CLARIFICATION → research task
   - For each dependency → best practices task
   - For each integration → patterns task

2. **Generate and dispatch research agents**:
   ```
   For each unknown in Technical Context:
     Task: "Research {unknown} for {feature context}"
   For each technology choice:
     Task: "Find best practices for {tech} in {domain}"
   ```

3. **Consolidate findings** in `research.md` using format:
   - Decision: [what was chosen]
   - Rationale: [why chosen]
   - Alternatives considered: [what else evaluated]

**Output**: research.md with all NEEDS CLARIFICATION resolved

## Phase 1: Design & Contracts
*Prerequisites: research.md complete*

1. **Extract entities from feature spec** → `data-model.md`:
   - Entity name, fields, relationships
   - Validation rules from requirements
   - State transitions if applicable

2. **Generate API contracts** from functional requirements:
   - For each user action → endpoint
   - Use standard REST/GraphQL patterns
   - Output OpenAPI/GraphQL schema to `/contracts/`

3. **Generate contract tests** from contracts:
   - One test file per endpoint
   - Assert request/response schemas
   - Tests must fail (no implementation yet)

4. **Extract test scenarios** from user stories:
   - Each story → integration test scenario
   - Quickstart test = story validation steps

5. **Update agent file incrementally** (O(1) operation):
   - Run `/scripts/update-agent-context.sh [claude|gemini|copilot]` for your AI assistant
   - If exists: Add only NEW tech from current plan
   - Preserve manual additions between markers
   - Update recent changes (keep last 3)
   - Keep under 150 lines for token efficiency
   - Output to repository root

**Output**: data-model.md, /contracts/*, failing tests, quickstart.md, agent-specific file

## Phase 2: Task Planning Approach
*This section describes what the /tasks command will do - DO NOT execute during /plan*

**Task Generation Strategy**:
- Load `/templates/tasks-template.md` as base
- Generate tasks from Phase 1 design docs (contracts, data model, quickstart)
- Each protocol message type → contract test task [P]
- Each API endpoint → contract test task [P]
- Each data entity → model creation task [P] 
- Each user story → integration test task
- Hardware abstraction layer tasks for both processors
- Communication protocol implementation tasks
- Video processing pipeline tasks
- Settings UI implementation tasks

**Ordering Strategy**:
- TDD order: Tests before implementation 
- Dependency order: Protocol → HAL → Services → UI
- Hardware setup before software implementation
- Critical path prioritization: Radio control → Video → Settings
- Mark [P] for parallel execution (independent components)

**Dual Processor Considerations**:
- Microcontroller tasks can run parallel to AP tasks
- Communication protocol must be implemented before integration
- Hardware-specific tasks require actual hardware
- Cross-processor testing requires both systems operational

**Estimated Output**: 35-45 numbered, ordered tasks covering:
- Hardware abstraction layer (8-10 tasks)
- Communication protocol (6-8 tasks)  
- Radio control system (10-12 tasks)
- Video processing (6-8 tasks)
- Settings management (5-7 tasks)
- Integration testing (8-10 tasks)

**IMPORTANT**: This phase is executed by the /tasks command, NOT by /plan

## Phase 3+: Future Implementation
*These phases are beyond the scope of the /plan command*

**Phase 3**: Task execution (/tasks command creates tasks.md)  
**Phase 4**: Implementation (execute tasks.md following constitutional principles)  
**Phase 5**: Validation (run tests, execute quickstart.md, performance validation)

## Complexity Tracking
*Fill ONLY if Constitution Check has violations that must be justified*

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| [e.g., 4th project] | [current need] | [why 3 projects insufficient] |
| [e.g., Repository pattern] | [specific problem] | [why direct DB access insufficient] |


## Progress Tracking
*This checklist is updated during execution flow*

**Phase Status**:
- [x] Phase 0: Research complete (/plan command)
- [x] Phase 1: Design complete (/plan command)
- [x] Phase 2: Task planning complete (/plan command - describe approach only)
- [ ] Phase 3: Tasks generated (/tasks command)
- [ ] Phase 4: Implementation complete
- [ ] Phase 5: Validation passed

**Gate Status**:
- [x] Initial Constitution Check: PASS
- [x] Post-Design Constitution Check: PASS
- [x] All NEEDS CLARIFICATION resolved
- [x] Complexity deviations documented

---
*Based on Constitution v2.1.1 - See `/memory/constitution.md`*