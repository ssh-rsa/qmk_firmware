# CLAUDE.md - QMK Firmware Guide for AI Assistants

**Version:** 1.0
**Last Updated:** 2025-11-17
**Repository:** QMK Firmware - Quantum Mechanical Keyboard Firmware

---

## Table of Contents

1. [Repository Overview](#repository-overview)
2. [Directory Structure](#directory-structure)
3. [Development Workflow](#development-workflow)
4. [Coding Conventions](#coding-conventions)
5. [Build System](#build-system)
6. [Testing Infrastructure](#testing-infrastructure)
7. [Common Patterns](#common-patterns)
8. [Guidelines for AI Assistants](#guidelines-for-ai-assistants)
9. [Key Files Reference](#key-files-reference)
10. [Useful Commands](#useful-commands)

---

## Repository Overview

### What is QMK?

QMK (Quantum Mechanical Keyboard) is a keyboard firmware based on the TMK keyboard firmware, supporting:
- **Platforms**: AVR (ATmega32U4, AT90USB), ARM (STM32 via ChibiOS), RP2040 (Raspberry Pi Pico)
- **Scale**: 1000+ keyboard definitions, 10,000+ keymaps
- **Languages**: Primarily C (embedded), with C++ for testing and some features
- **License**: GPL v2/v3 and Modified BSD

### Key Characteristics

- **Memory Constrained**: Many AVR boards have only 32KB flash, 2.5KB RAM
- **Modular Features**: Features can be enabled/disabled per keyboard via `rules.mk`
- **Community-Driven**: Maintained by OLKB (Jack Humbert) with extensive community contributions
- **Well-Documented**: 120+ markdown files in `docs/`

### Official Resources

- **Website**: https://qmk.fm
- **Documentation**: https://docs.qmk.fm
- **Discord**: https://discord.gg/qmk
- **Subreddit**: https://reddit.com/r/olkb

---

## Directory Structure

### Top-Level Organization

```
/home/user/qmk_firmware/
├── keyboards/          # 1000+ keyboard implementations
├── quantum/            # Core QMK firmware engine
├── tmk_core/           # Base TMK keyboard firmware
├── platforms/          # Platform-specific code (AVR, ARM, RP2040)
├── drivers/            # Hardware drivers (LEDs, displays, audio, etc.)
├── lib/                # External libraries (LUFA, ChibiOS, googletest)
├── layouts/            # Predefined keyboard layouts (ANSI, ISO, etc.)
├── modules/            # Modular QMK extensions
├── data/               # Schemas, templates, constants for codegen
├── builddefs/          # Build system definitions and rules
├── tests/              # Unit tests (GoogleTest framework)
├── docs/               # VitePress documentation
├── util/               # Build utilities, installation scripts
├── users/              # User-specific code (deprecated for new submissions)
└── .github/            # CI/CD workflows
```

### Critical Directories Explained

#### `keyboards/` - Keyboard Implementations

**Structure**: `keyboards/{manufacturer}/{keyboard_name}/{revision}/`

Each keyboard contains:
- **`config.h`**: Hardware configuration (pins, matrix, features)
- **`rules.mk`**: Build flags and feature toggles
- **`{name}.c/h`**: Keyboard initialization and custom code
- **`info.json`**: Metadata (maintainer, URL, QMK Configurator data)
- **`keymaps/`**: User-defined layouts, each with:
  - `keymap.c` - Key layout definition
  - `config.h` (optional) - Keymap-specific overrides
  - `rules.mk` (optional) - Keymap-specific features

**Example**: `/keyboards/planck/rev7/` - Planck keyboard revision 7

#### `quantum/` - Core Firmware Engine

The heart of QMK, containing:
- **`quantum.c/h`**: Main initialization and event loop
- **`action.c/h`**: Keypress action processing (1300+ lines)
- **`action_tapping.c`**: Tap/hold key logic (900+ lines)
- **`keyboard.c/h`**: Keyboard scanning loop
- **`matrix.c/h`**: Key matrix scanning
- **`keycodes.h`**: 1000+ keycode definitions
- **`quantum_keycodes.h`**: Extended keycodes (layers, mod-tap, etc.)

**Subdirectories**:
- `process_keycode/` - Feature-specific keycode processors
- `audio/` - Music/audio synthesis
- `rgb_matrix/`, `rgblight/` - LED control systems
- `split_common/` - Split keyboard communication
- `debounce/` - Key debouncing algorithms
- `unicode/`, `send_string/` - Text input features
- `painter/`, `encoder/` - Display and rotary encoder support

#### `platforms/` - Hardware Abstraction

Platform-specific implementations:
- **`avr/`**: AVR microcontroller support (LUFA-based USB)
- **`chibios/`**: ARM support via ChibiOS RTOS
- **`test/`**: Mock drivers for unit testing

Provides: GPIO, timers, suspend, EEPROM, bootloader interfaces

#### `drivers/` - Hardware Drivers

Organized by peripheral type:
- `backlight/`, `led/` - LED control
- `oled/`, `lcd/` - Display drivers
- `painter/` - Advanced graphics (LVGL integration)
- `audio/` - Audio synthesis
- `haptic/` - Vibration feedback
- `eeprom/`, `flash/` - Persistent storage
- `sensors/` - Environmental sensors
- `encoder/` - Rotary encoders

#### `tmk_core/` - TMK Base

Foundation code from the TMK keyboard project:
- `protocol/` - USB HID protocol (LUFA-based)
- Legacy but essential layer beneath QMK

#### `tests/` - Testing Infrastructure

**Structure**: `tests/{feature}/test_{feature}.c`

Each test contains:
- `config.h` - Test-specific configuration
- `test_*.c` - Test implementation (GoogleTest)
- `keymap.c` - Test keyboard layout
- `rules.mk` - Test build configuration

**Common infrastructure**: `tests/test_common/`
- `test_driver.cpp` - Mock USB HID driver
- `matrix.c` - Simulated key matrix

#### `builddefs/` - Build System

Make-based build system components:
- **`common_features.mk`** (700+ lines): Feature compilation logic
- **`build_keyboard.mk`**: Main keyboard build process
- **`common_rules.mk`**: Compiler flags, optimization, linking
- **`build_test.mk`**: Test compilation

#### `data/` - Code Generation

Resources for code generation:
- `constants/` - Keycode definitions
- `schemas/` - JSON Schema for `info.json`
- `templates/` - Code generation templates
- `mappings/` - Feature/keycode mappings

#### `docs/` - Documentation

VitePress-rendered documentation:
- **Getting Started**: `newbs_*.md` (~10 files)
- **Feature Guides**: `feature_*.md` (~50 files)
- **Platform Guides**: `platformdev_*.md`
- **Development**: `api_*.md`, debugging guides
- **Contributing**: Breaking changes, conventions

---

## Development Workflow

### Making Contributions

**Process** (from `docs/contributing.md`):

1. **Fork** the repository to your GitHub account
2. **Clone** to local machine: `git clone https://github.com/your-username/qmk_firmware.git`
3. **Create branch**: `git checkout -b feature-name`
4. **Make changes** following coding conventions
5. **Test locally**: `make keyboard:keymap` or `make test:feature`
6. **Commit** with clear messages (see commit style below)
7. **Push**: `git push origin feature-name`
8. **Open PR** against `qmk/qmk_firmware:master`

### Commit Message Style

**Format**:
```
Short description (≤70 characters)

Detailed explanation of what changed and why (if needed).
Reference issues with #issue_number.
```

**Example**:
```
Adjust the fronzlebop for the kerpleplork

The kerpleplork was intermittently failing with error code 23.
The root cause was the fronzlebop setting, which causes the
kerpleplork to activate every N iterations.
```

### PR Guidelines

- **Separate PRs** for separate features/fixes
- **No whitespace changes** - check with `git diff --check`
- **Must compile** - verify with `make keyboard:keymap`
- **Keep commits reasonable** or they will be squashed
- **No user keymaps** - user contributions are no longer accepted
- **Core changes separate** from keyboard additions
- **Fork must be up to date** with upstream before PR

### Feature Development

**For new features**:
1. **Discuss first** - Open issue or chat on Discord
2. **Disabled by default** - Use opt-in via `rules.mk`
3. **Document** in `docs/feature_{name}.md`
4. **Write tests** in `tests/`
5. **Support both AVR and ARM** where possible
6. **Keep reasonable code size** - memory is limited

### Testing Before Submission

- **Keymaps**: `make keyboard:keymap` (no errors)
- **Keyboards**: `make keyboard:all` (all keymaps compile)
- **Core**: `make all` (entire codebase compiles)
- **Tests**: `make test` or `make test:specific_test`
- **Format**: `make format` (auto-format with clang-format)

---

## Coding Conventions

### C Code Style

**Indentation**:
- **4 spaces** (soft tabs, never hard tabs)
- Use spaces, not tabs

**Bracing** (Modified One True Brace Style):
```c
// Opening brace: end of line
if (condition) {
    return true;
} else {
    return false;
}

// Always use braces, even for single statements
if (condition) {
    return false;  // Good
}

if (condition) return false;  // Bad - missing braces
```

**Comments**:
- Prefer **C-style comments**: `/* */`
- Use liberally to explain **why**, not **what**
- Don't write obvious comments
- Think of comments as telling a story

**Line Length**:
- Generally **no line wrapping** (can be as long as needed)
- If wrapping, don't exceed **76 columns**
- clang-format configured for 1000 column limit (generous)

**Header Guards**:
- Use `#pragma once` (preferred)
- Not old-style include guards

**Preprocessor Directives**:
- Accept both: `#ifdef DEFINED` and `#if defined(DEFINED)`
- When uncertain, prefer `#if defined(DEFINED)`
- For readability, indent after `#` (e.g., `#    if defined(FOO)`)

**Naming Conventions**:
- **Functions**: `snake_case`
- **Macros/Constants**: `SCREAMING_SNAKE_CASE`
- **Types**: `snake_case_t` or occasionally `CamelCase`
- **Variables**: `snake_case`
- **Enums**: `SCREAMING_SNAKE_CASE` for values

**Example**:
```c
/* Enums for foo */
enum foo_state {
    FOO_BAR,
    FOO_BAZ,
};

/* Returns a value */
int foo(void) {
    if (some_condition) {
        return FOO_BAR;
    } else {
        return -1;
    }
}
```

### Auto-Formatting

**Tool**: clang-format (LLVM)
- **Config**: `.clang-format` (Google-based style)
- **Usage**: `make format` or `clang-format -style=file -i file.c`
- **Install**:
  - Ubuntu: `sudo apt install clang-format`
  - Windows: Full LLVM installer
- **VSCode**: Use C/C++ extension or ClangFormat extension

**Note**: Some macros (like `LAYOUT_*`) are destroyed by clang-format. Wrap sensitive code:
```c
// clang-format off
#define LAYOUT_planck_grid( \
    k00, k01, k02, ... \
) { \
    { k00, k01, k02, ... } \
}
// clang-format on
```

### Python Code Style

**Tool**: yapf + flake8
- **Config**: `setup.cfg`
- **Column limit**: 256
- **Ignore**: E501 (long lines), E231 (conflicts with yapf)

---

## Build System

### Build Architecture

QMK uses a **GNU Make**-based build system with modular feature selection.

### Build Flow

```
make <keyboard>:<keymap>   # e.g., make planck/rev7:default

1. Makefile parses target → extracts KEYBOARD and KEYMAP
2. Includes paths.mk → sets directory variables
3. Includes builddefs/common_features.mk → feature conditionals
4. Compiles quantum core (action.c, keyboard.c, etc.)
5. Compiles keyboard-specific code (from keyboards/)
6. Compiles keymap-specific code (keymaps/*/keymap.c)
7. Links → produces .elf, .hex, .uf2 artifacts
```

### Key Build Files

| File | Purpose |
|------|---------|
| `Makefile` | Main orchestrator (350+ lines) |
| `paths.mk` | Directory path definitions |
| `builddefs/common_features.mk` | Feature toggle logic (700+ lines) |
| `builddefs/common_rules.mk` | Compiler flags, optimization |
| `builddefs/build_keyboard.mk` | Keyboard compilation |
| `keyboards/*/rules.mk` | Per-keyboard feature selection |
| `keyboards/*/config.h` | Per-keyboard hardware config |

### Build Variables (Feature Flags)

Enable/disable features in `rules.mk`:

```makefile
# Audio and visual features
AUDIO_ENABLE = yes              # Sound synthesis
RGB_MATRIX_ENABLE = yes         # RGB LED matrix control
RGBLIGHT_ENABLE = yes           # RGB LED strip control
BACKLIGHT_ENABLE = yes          # LED backlight

# Input features
MOUSEKEY_ENABLE = yes           # Mouse keys
COMBO_ENABLE = yes              # Key combinations
TAP_DANCE_ENABLE = yes          # Multi-tap sequences
KEY_OVERRIDE_ENABLE = yes       # Key overrides

# Communication features
UNICODE_ENABLE = yes            # Unicode input
MIDI_ENABLE = yes               # MIDI protocol

# Keyboard-specific
SPLIT_KEYBOARD = yes            # Split keyboard support
CUSTOM_MATRIX = lite            # Custom matrix scanning

# Advanced features
VIA_ENABLE = yes                # VIA configurator support
CONSOLE_ENABLE = yes            # Debug console
COMMAND_ENABLE = yes            # Magic commands
```

### Compiler Configuration

From `builddefs/common_rules.mk`:
- **Standard**: GNU C11, C++14
- **Optimization**: `-Os` (optimize for size)
- **Warnings**: `-Wall -Werror` (warnings are errors unless `ALLOW_WARNINGS=yes`)
- **LTO**: Link-time optimization with `-flto`

### Build Outputs

- **`.hex`**: Intel HEX format for bootloader flashing
- **`.elf`**: ELF executable (for debugging)
- **`.uf2`**: UF2 format for RP2040 drag-and-drop flashing
- **`.bin`**: Raw binary (some platforms)

### Include System

QMK uses a **magic include macro**:
```c
#include QMK_KEYBOARD_H  // Expands to correct keyboard header
```

This automatically resolves to the appropriate keyboard-specific headers based on build target.

### Build Targets

```bash
make keyboard:keymap              # Build specific keyboard/keymap
make keyboard:all                 # Build all keymaps for keyboard
make all:all                      # Build everything (slow!)
make test                         # Run all tests
make test:specific_test           # Run specific test
make format                       # Auto-format code
make clean                        # Clean build artifacts
```

---

## Testing Infrastructure

### Test Framework

- **Runner**: nose2 (Python test runner)
- **C++ Framework**: GoogleTest (via `lib/googletest/`)
- **Config**: `nose2.cfg`

### Running Tests

```bash
make test                    # Run all tests
make test:combo             # Run specific test
make test:all               # Run all tests verbosely
```

### Test Structure

Each test in `tests/{feature}/`:
```
tests/combo/
├── config.h              # Test-specific config overrides
├── test_combos.c         # Test implementation (GoogleTest)
├── keymap.c              # Test keyboard layout
└── rules.mk              # Test build configuration
```

### Writing Tests

**Example test structure**:
```c
#include "test_common.h"

// Test implementation
TEST(Combo, BasicCombo) {
    // Setup
    auto fixture = TestDriver();

    // Execute
    tap_key(key_a);
    tap_key(key_b);

    // Verify
    EXPECT_EQ(get_last_keycode(), KC_C);
}
```

### Test Common Infrastructure

`tests/test_common/` provides:
- `test_driver.cpp` - Mock USB HID driver
- `matrix.c` - Simulated key matrix
- `keymap.c` - Base test keymap
- Shared utilities for all tests

### CI/CD Workflows

GitHub Actions workflows in `.github/workflows/`:
- **`ci_build_major_branch.yml`**: Build all keyboards on main
- **`unit_test.yml`**: Run unit tests
- **`lint.yml`**: Code style checks (flake8, clang-format)
- **`format.yml`**: Auto-format PRs
- **`auto_approve.yml`**: Auto-approve bot changes

---

## Common Patterns

### Keyboard Configuration Pattern

**`keyboards/{name}/config.h`**:
```c
#pragma once
#include "config_common.h"

/* Matrix Configuration */
#define MATRIX_ROWS 4
#define MATRIX_COLS 12
#define MATRIX_ROW_PINS { D0, D5, B5, B6 }
#define MATRIX_COL_PINS { F1, F0, B0, C7, F4, F5, F6, F7, D4, D6, B4, D7 }

/* Diode Direction */
#define DIODE_DIRECTION COL2ROW

/* Audio Configuration */
#define AUDIO_PIN C6

/* RGB Configuration */
#define RGB_DI_PIN B5
#define RGBLED_NUM 9
```

**`keyboards/{name}/rules.mk`**:
```makefile
# MCU Configuration
MCU = atmega32u4

# Bootloader
BOOTLOADER = atmel-dfu

# Features
AUDIO_ENABLE = yes
MIDI_ENABLE = yes
BACKLIGHT_ENABLE = no
RGBLIGHT_ENABLE = yes

# Custom matrix if needed
CUSTOM_MATRIX = lite
SRC += matrix.c
```

### Keymap Definition Pattern

**`keyboards/{name}/keymaps/{keymap}/keymap.c`**:
```c
#include QMK_KEYBOARD_H

/* Layer definitions */
enum planck_layers {
    _QWERTY,
    _LOWER,
    _RAISE,
    _ADJUST
};

/* Custom keycodes */
enum planck_keycodes {
    QWERTY = SAFE_RANGE,
    LOWER,
    RAISE,
    MY_CUSTOM_KEY
};

/* Keymap layout */
const uint16_t PROGMEM keymaps[][MATRIX_ROWS][MATRIX_COLS] = {
    [_QWERTY] = LAYOUT_planck_grid(
        KC_TAB,  KC_Q,    KC_W,    KC_E,    KC_R,    KC_T,    KC_Y,    KC_U,    KC_I,    KC_O,    KC_P,    KC_BSPC,
        KC_ESC,  KC_A,    KC_S,    KC_D,    KC_F,    KC_G,    KC_H,    KC_J,    KC_K,    KC_L,    KC_SCLN, KC_QUOT,
        KC_LSFT, KC_Z,    KC_X,    KC_C,    KC_V,    KC_B,    KC_N,    KC_M,    KC_COMM, KC_DOT,  KC_SLSH, KC_ENT,
        KC_LCTL, KC_LGUI, KC_LALT, KC_APP,  LOWER,   KC_SPC,  KC_SPC,  RAISE,   KC_LEFT, KC_DOWN, KC_UP,   KC_RGHT
    ),

    [_LOWER] = LAYOUT_planck_grid(
        KC_TILD, KC_EXLM, KC_AT,   KC_HASH, KC_DLR,  KC_PERC, KC_CIRC, KC_AMPR, KC_ASTR, KC_LPRN, KC_RPRN, KC_DEL,
        _______, KC_F1,   KC_F2,   KC_F3,   KC_F4,   KC_F5,   KC_F6,   KC_UNDS, KC_PLUS, KC_LCBR, KC_RCBR, KC_PIPE,
        _______, KC_F7,   KC_F8,   KC_F9,   KC_F10,  KC_F11,  KC_F12,  _______, _______, _______, _______, _______,
        _______, _______, _______, _______, _______, _______, _______, _______, KC_HOME, KC_PGDN, KC_PGUP, KC_END
    )
};
```

### Hook/Callback Pattern

QMK provides **predictable callback hooks** for extensibility:

```c
/* Called every matrix scan (~1000 Hz) */
void matrix_scan_user(void) {
    // Custom per-scan logic
}

/* Called on keycode processing - return false to stop processing */
bool process_record_user(uint16_t keycode, keyrecord_t *record) {
    switch (keycode) {
        case MY_CUSTOM_KEY:
            if (record->event.pressed) {
                // Key pressed
                SEND_STRING("Hello QMK!");
            } else {
                // Key released
            }
            return false;  // Skip default processing
    }
    return true;  // Continue default processing
}

/* Called on layer change */
layer_state_t layer_state_set_user(layer_state_t state) {
    // React to layer changes
    return update_tri_layer_state(state, _LOWER, _RAISE, _ADJUST);
}

/* Called on encoder rotation */
bool encoder_update_user(uint8_t index, bool clockwise) {
    if (clockwise) {
        tap_code(KC_VOLU);
    } else {
        tap_code(KC_VOLD);
    }
    return false;
}

/* Called on LED state change */
bool led_update_user(led_t led_state) {
    // React to LED changes (Caps Lock, etc.)
    return true;
}

/* Keyboard initialization */
void keyboard_post_init_user(void) {
    // Run once at keyboard startup
    rgblight_enable();
}
```

### Feature Processor Pattern

Features implement `process_record_{feature}()`:

```c
// In quantum/process_keycode/process_combo.c
bool process_combo(uint16_t keycode, keyrecord_t *record) {
    // Handle combo-specific logic
    return true;  // Continue processing
}
```

These are chained in `quantum/quantum.c`:
```c
bool process_record_quantum(keyrecord_t *record) {
    if (!process_combo(...)) return false;
    if (!process_tap_dance(...)) return false;
    if (!process_unicode(...)) return false;
    // ...
    return process_record_user(...);
}
```

---

## Guidelines for AI Assistants

### Memory Constraints - Critical!

**ALWAYS** consider memory constraints:
- **AVR boards**: 32KB flash, 2.5KB RAM (typical)
- **Features add size**: AUDIO_ENABLE adds 8KB+, RGB adds 4KB+
- **Reference**: `docs/squeezing_avr.md` for optimization techniques

**Before adding features**:
1. Check current firmware size: Look for warnings in build output
2. Consider if feature can be made optional
3. Prefer compile-time over runtime configuration
4. Use `PROGMEM` for large constant data

### Code Modification Strategy

When modifying QMK code:

1. **Identify scope**:
   - Keymap change? → Only touch `keyboards/{name}/keymaps/{keymap}/`
   - Keyboard feature? → Modify `keyboards/{name}/config.h` or `rules.mk`
   - Core feature? → Discuss first, then modify `quantum/`

2. **Preserve existing style**:
   - Match surrounding code style
   - Use clang-format if uncertain
   - Don't reformat unrelated code

3. **Test incrementally**:
   - Build after each change: `make keyboard:keymap`
   - Fix compilation errors immediately
   - Test on actual hardware if possible

4. **Document changes**:
   - Update relevant `docs/` files
   - Add comments explaining **why**, not **what**
   - Update `info.json` if changing keyboard metadata

### Common Mistakes to Avoid

❌ **Don't**:
- Submit user keymaps (deprecated)
- Mix keyboard additions with core changes in one PR
- Skip testing compilation
- Add features enabled by default (memory constraints)
- Ignore existing code style
- Modify `LAYOUT_*` macros without protecting from clang-format
- Use hard tabs (use 4 spaces)
- Skip braces on single-line conditionals

✅ **Do**:
- Separate PRs for separate features
- Test with `make keyboard:keymap` before committing
- Use feature flags (`*_ENABLE` in `rules.mk`)
- Follow file naming: `keyboards/foo/bar/bar.c` (not `foo.c`)
- Update copyright headers
- Write unit tests for core features
- Use `#pragma once` for header guards
- Always use braces for conditionals

### Finding Relevant Code

**For keyboard-specific issues**:
1. Start in `keyboards/{manufacturer}/{keyboard}/`
2. Check `config.h` for hardware configuration
3. Check `rules.mk` for enabled features
4. Look at `keymaps/default/` for reference implementation

**For feature implementation**:
1. Check `quantum/process_keycode/process_{feature}.c`
2. Look for feature docs in `docs/feature_{feature}.md`
3. Search for tests in `tests/{feature}/`
4. Check `builddefs/common_features.mk` for build integration

**For hardware drivers**:
1. Check `drivers/{peripheral_type}/`
2. Look at platform abstraction in `platforms/{platform}/`
3. Reference existing similar drivers

### When to Ask for Help

**Discuss with maintainers before**:
- Implementing new core features
- Changing build system behavior
- Modifying matrix scanning or action processing
- Adding new platform support
- Making breaking changes

**Where to discuss**:
- Discord: https://discord.gg/qmk
- GitHub Issues: For bugs and feature requests
- GitHub Discussions: For questions

### Understanding QMK Program Flow

**Critical reading**: `docs/understanding_qmk.md`

**Simplified flow**:
```
1. main() [tmk_core/protocol/lufa/main.c]
   ↓
2. keyboard_init() [quantum/keyboard.c]
   ↓
3. keyboard_post_init_user() [user callback]
   ↓
4. while(1) keyboard_task()
   ↓
5. matrix_scan() [scan key matrix]
   ↓
6. process_record_quantum() [process each keypress]
   ↓
7. process_record_user() [user callback]
   ↓
8. Send USB HID reports
```

### Feature Flag Patterns

Features follow consistent patterns:

**In `keyboards/{name}/rules.mk`**:
```makefile
FEATURE_ENABLE = yes
```

**In `keyboards/{name}/config.h`**:
```c
#ifdef FEATURE_ENABLE
#    define FEATURE_PARAM 42
#endif
```

**In `quantum/`**:
```c
#ifdef FEATURE_ENABLE
bool process_record_feature(uint16_t keycode, keyrecord_t *record) {
    // Implementation
}
#endif
```

### Multi-Platform Considerations

When writing code that runs on multiple platforms:

```c
// Use platform checks
#if defined(__AVR__)
    // AVR-specific code
#elif defined(PROTOCOL_CHIBIOS)
    // ARM/ChibiOS code
#elif defined(PROTOCOL_PICO)
    // RP2040 code
#endif

// Or use platform abstraction
#include "gpio.h"  // Platform-agnostic GPIO
setPinOutput(pin);
writePinHigh(pin);
```

### Debugging Tips

**Enable console output**:
```makefile
# In rules.mk
CONSOLE_ENABLE = yes
```

```c
// In code
#include "print.h"
uprintf("Debug: value=%d\n", value);
```

**Check firmware size**:
```bash
make keyboard:keymap
# Output shows: "Firmware size: 24xxx/28672"
```

**Common build issues**:
- **"region 'text' overflowed"**: Firmware too large, disable features
- **Missing header**: Check if feature is enabled in `rules.mk`
- **Undefined reference**: Missing source file in `SRC +=` in `rules.mk`

---

## Key Files Reference

### Must-Read Files

| File | Lines | Purpose | When to Read |
|------|-------|---------|--------------|
| `readme.md` | 40 | Project overview | First |
| `docs/contributing.md` | 200+ | Contribution guide | Before contributing |
| `docs/coding_conventions_c.md` | 60 | C code style | Before coding |
| `docs/understanding_qmk.md` | - | Program flow | Understanding architecture |
| `quantum/quantum.c` | 600+ | Main quantum loop | Core development |
| `quantum/action.c` | 1300+ | Key action processing | Understanding keypress handling |
| `builddefs/common_features.mk` | 700+ | Feature system | Adding features |
| `Makefile` | 350+ | Build orchestration | Build system changes |

### Configuration Files

| File | Purpose |
|------|---------|
| `.clang-format` | C code formatting rules |
| `setup.cfg` | Python formatting (yapf, flake8) |
| `nose2.cfg` | Test runner configuration |
| `Doxyfile` | API documentation generation |
| `paths.mk` | Directory path definitions |
| `requirements.txt` | Python dependencies |
| `requirements-dev.txt` | Development tools |

### Platform-Specific Entry Points

| Platform | Main File |
|----------|-----------|
| AVR/LUFA | `tmk_core/protocol/lufa/main.c` |
| ARM/ChibiOS | `tmk_core/protocol/chibios/main.c` |
| RP2040 | `platforms/chibios/_wait.c` (via ChibiOS) |

---

## Useful Commands

### Building

```bash
# Build specific keyboard/keymap
make planck/rev7:default

# Build all keymaps for a keyboard
make planck/rev7:all

# Build with verbose output
make planck/rev7:default VERBOSE=1

# Clean build artifacts
make clean

# Build and flash
make planck/rev7:default:flash
```

### Testing

```bash
# Run all tests
make test

# Run specific test
make test:combo

# Run tests verbosely
make test:all

# Run with coverage (if configured)
make test COVERAGE=1
```

### Code Quality

```bash
# Auto-format code
make format

# Check formatting (CI mode)
make format-check

# Generate documentation
make docs

# Preview documentation
qmk docs -b
```

### Development

```bash
# List available keyboards
qmk list-keyboards

# List available keymaps
qmk list-keymaps -kb planck/rev7

# Create new keymap
qmk new-keymap -kb planck/rev7

# Compile using QMK CLI
qmk compile -kb planck/rev7 -km default

# Flash using QMK CLI
qmk flash -kb planck/rev7 -km default
```

### Information

```bash
# Show build options
make planck/rev7:default VERBOSE=1 | grep "ENABLE"

# Check firmware size
make planck/rev7:default | grep "Firmware size"

# Show QMK version
qmk --version

# Doctor (check QMK setup)
qmk doctor
```

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2025-11-17 | Initial comprehensive documentation |

---

## Additional Resources

### Documentation
- **QMK Docs**: https://docs.qmk.fm
- **Newbs Guide**: https://docs.qmk.fm/newbs
- **FAQ**: https://docs.qmk.fm/faq

### Community
- **Discord**: https://discord.gg/qmk
- **Reddit**: https://reddit.com/r/olkb
- **GitHub**: https://github.com/qmk/qmk_firmware

### Learning Resources
- **C Programming**: Essential for QMK development
- **Embedded Systems**: Understanding microcontrollers helps
- **Git**: Version control basics
- **Make**: Build system knowledge useful

---

**Note to AI Assistants**: This document is intended to provide comprehensive context for working with the QMK firmware codebase. When in doubt, consult the official documentation at https://docs.qmk.fm and discuss with maintainers on Discord or GitHub before making significant changes.
