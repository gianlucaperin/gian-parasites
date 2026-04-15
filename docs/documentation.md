# Parasites — Mutable Instruments Eurorack Module Firmware

## Overview

**Parasites** is an alternative firmware project for [Mutable Instruments](https://mutable-instruments.net/) Eurorack synthesizer modules, originally developed by Émilie Gillet. This repository (fork of `mqtthiqs/parasites`) contains firmware source code, hardware designs, and bootloaders for 18 Eurorack modules spanning two hardware platforms: **AVR** microcontrollers and **STM32 ARM** processors.

The Parasites firmware enhances existing features, adds new modes and functions, and introduces hidden capabilities — all while retaining as much of the factory functionality as possible.

## License

| Component | License |
|---|---|
| Code (AVR projects) | GPL 3.0 |
| Code (STM32F projects) | MIT |
| Hardware designs | CC-BY-SA-3.0 |

Original author: Émilie Gillet (emilie.o.gillet@gmail.com)

---

## Repository Structure

```
parasites/
├── README.md                          # Module listing and license info
├── presentation.org                   # Detailed feature documentation (Clouds, Frames, Tides)
├── __init__.py                        # Python package marker
│
├── ── Module Firmware ──────────────
├── braids/                            # Macro-oscillator (STM32F10x)
├── branches/                          # Dual Bernoulli gate (AVR ATMega88)
├── clouds/                            # Texture synthesizer (STM32F4xx)
├── edges/                             # Quad chiptune digital oscillator (AVR ATMega32A4)
├── elements/                          # Modal synthesizer (STM32F4xx)
├── frames/                            # Keyframer/mixer (STM32F10x)
├── grids/                             # Topographic drum sequencer (AVR ATMega328P)
├── peaks/                             # Dual trigger converter (STM32F10x)
├── rings/                             # Resonator (STM32F4xx)
├── streams/                           # Dual dynamics gate (STM32F10x)
├── tides/                             # Tidal modulator (STM32F10x)
├── warps/                             # Meta-modulator (STM32F4xx)
├── yarns/                             # MIDI interface (STM32F10x)
│
├── ── Hardware-Only Modules ────────
├── links/                             # Utility module (buffer, mixer)
├── ripples/                           # 2-pole/4-pole filter
├── shades/                            # Triple attenuverter
├── shelves/                           # EQ filter
├── volts/                             # +5V power supply
│
├── ── Support Libraries ────────────
├── stmlib/                            # STM32 shared library (git submodule)
├── avrlib/                            # AVR shared library (git submodule)
├── avrlibx/                           # Extended AVR library (git submodule)
│
├── ── Bootloaders ──────────────────
├── stm_audio_bootloader/              # STM32 firmware update via audio (QPSK)
├── avr_audio_bootloader/              # AVR firmware update via audio (FSK)
│
└── ── Tooling ──────────────────────
    └── tools/
        ├── hex2sysex/                 # MIDI SysEx firmware encoder
        ├── hexfile/                   # Hex file utilities
        ├── learning/                  # Learning resources
        ├── midi/                      # MIDI utilities
        └── optimization/             # Performance optimization tools
```

---

## Hardware Platforms

### STM32F10x (72 MHz, Medium Density)

| Parameter | Value |
|---|---|
| Crystal | 8 MHz |
| CPU Clock | 72 MHz |
| Family | STM32F10x |
| Flash Density | Medium (md) |
| Upload | JTAG or audio bootloader (QPSK) |

**Modules**: Braids, Frames, Peaks, Streams, Tides, Yarns

### STM32F4xx (168 MHz, High Performance)

| Parameter | Value |
|---|---|
| CPU Clock | 168 MHz |
| Family | STM32F4xx |
| Upload | JTAG or audio bootloader (QPSK) |
| Flash | Large allocation (`APPLICATION_LARGE`) |

**Modules**: Clouds, Elements, Rings, Warps

### AVR

| MCU | Module | Clock |
|---|---|---|
| ATMega88 | Branches | 8 MHz |
| ATMega328P | Grids | 16 MHz |
| ATMega32A4 | Edges | 32 MHz |

Upload via ISP programmer or audio bootloader (FSK).

---

## Module Details

### Braids — Macro-Oscillator

**Platform**: STM32F10x (72 MHz)

A digital macro-oscillator featuring over 40 synthesis algorithms organized into two rendering paths:

- **Analog emulation**: Sawtooth, square, triangle with morphing
- **Digital synthesis** (40+ shapes):
  - Ring modulation (triple, interactive)
  - Saw morphing and comb filters
  - Digital filters (LP, BP, PK)
  - Physical modeling: struck bells, drums, plucked strings, bowed, blown, fluted
  - Spectral: vowel synthesis, formant-based FOF, harmonics
  - Wavetable synthesis with morphing
  - Feedback FM and chaotic FM
  - Noise variants: filtered, particle, granular
  - Granular cloud

**Key source files**:
- `braids/braids.cc` — Main firmware entry point
- `braids/macro_oscillator.cc/h` — Main oscillator interface
- `braids/digital_oscillator.cc/h` — Digital synthesis algorithms
- `braids/analog_oscillator.cc/h` — Analog emulation
- `braids/settings.cc/h` — Parameter storage
- `braids/ui.cc/h` — User interface logic
- `braids/resources/` — Python-generated wavetables and lookup tables
- `braids/drivers/` — Hardware abstraction (ADC, DAC, display, gate I/O)
- `braids/test/` — Unit tests

---

### Clouds — Texture Synthesizer

**Platform**: STM32F4xx (168 MHz)

A granular texture processor with 6 playback modes:

1. **Granular** — Wavetable-based grain synthesis with overlapping windowed samples
2. **Stretch** — Phase vocoder (PVoc/STFT) time-stretching
3. **Looping Delay** — Long delay with modulation and pitch shifting
4. **Spectral** — Frequency-domain processing and morphing
5. **OliVerb** — Advanced reverb with shimmer, feedback, internal LFO modulation, and pre-delay with clock sync
6. **Resonestor** — Polyphonic resonator (4 voices × 2 channels) with Karplus-Strong string synthesis, chord modes, and trigger-based plucking

**Memory architecture**: 32 KB CCM reverb buffer, 2× downsampling for computation efficiency.

**Key source files**:
- `clouds/clouds.cc` — Main firmware entry point
- `clouds/dsp/granular_processor.h` — Main processing engine
- `clouds/dsp/grain.h` — Individual grain control
- `clouds/dsp/pvoc/phase_vocoder.h` — Time-frequency analysis
- `clouds/dsp/resonestor.h` — Polyphonic resonator
- `clouds/dsp/fx/oliverb.h` — Advanced reverb
- `clouds/dsp/fx/` — Effects (diffuser, pitch shifter, reverb)
- `clouds/dsp/sample_rate_converter.h` — SRC

---

### Elements — Modal Synthesizer

**Platform**: STM32F4xx (168 MHz)

A physical modeling synthesizer using source → resonator topology:

**Exciter types**:
- Strike (impulse-based contact)
- Blow (sustained breath)
- Bow (friction-based)
- Pluck (finger-like sharp attack)

**Resonator**: Modal filter bank using Schroeder reverberator with multiple delay lines, post-resonator reverb for spatial effects.

Supports up to 4 voices in code (UI-limited to monophonic).

**Key source files**:
- `elements/elements.cc` — Main firmware
- `elements/dsp/part.h` — Monophonic voice manager
- `elements/dsp/voice.h` — Individual voice processor
- `elements/dsp/exciter.cc/h` — Excitation synthesis
- `elements/dsp/resonator.cc/h` — Modal filter implementation
- `elements/dsp/tube.cc/h` — Waveguide synthesis
- `elements/dsp/ominous_voice.h` — Alternative FM-based DSP engine
- `elements/dsp/fx/reverb.h` — Schroeder reverberator
- `elements/samples/` — Impulse responses

---

### Rings — Resonator

**Platform**: STM32F4xx (168 MHz)

A polyphonic resonator (up to 4 voices) with 6 resonator models:

1. **Modal resonator** — Schroeder reverberator-based
2. **Sympathetic string** — Coupled oscillators
3. **String** — Karplus-Strong with strike
4. **FM voice** — Frequency modulation synthesis
5. **Quantized string** — MIDI-note quantized strings
6. **String + Reverb** — Hybrid model

**Key source files**:
- `rings/dsp/part.h` — Polyphonic voice manager
- `rings/dsp/resonator.h` — Resonance processor
- `rings/dsp/string.h` — Karplus-Strong synthesis
- `rings/dsp/fm_voice.h` — FM synthesis voice
- `rings/dsp/string_synth_part.h` — String ensemble
- `rings/dsp/plucker.h` — Plucking excitation
- `rings/dsp/strummer.h` — String strumming interface

---

### Warps — Meta-Modulator

**Platform**: STM32F4xx (168 MHz)

A signal modulator with oversampled processing (4×–6× oversampling):

- Saturating amplifier with noise gate
- Quadrature oscillator modulation
- Sample rate conversion (polyphase FIR filters)
- Vocoder (amplitude modulation synthesis)

**Processing chain**: Input stage (noise gate, soft clipping) → Core (quadrature modulation or vocoding) → SRC → Post-gain and limiting.

**Key source files**:
- `warps/dsp/modulator.h` — Main modulation engine
- `warps/dsp/oscillator.h` — Wavetable oscillator
- `warps/dsp/quadrature_oscillator.h` — I/Q oscillator
- `warps/dsp/vocoder.h` — Amplitude modulation synthesis
- `warps/dsp/filter_bank.h` — Frequency-domain filters
- `warps/dsp/limiter.h` — Output limiter

---

### Tides — Tidal Modulator

**Platform**: STM32F10x (72 MHz)

A function generator / LFO / envelope with:

- Range selection: high (audio rate), medium, low (slow LFO)
- Modes: AD (attack-decay), looping, AR (attack-release)
- Wavetable morphing (sine → triangle → square → sawtooth)
- Clock detection and synchronization with pattern predictor
- Freeze capability
- Easing curves: step, linear, quartic in/out, sine, bounce

**Parasites additions**: Quantizer mode, Two-Bumps mode, harmonic oscillator enhancements.

**Key source files**:
- `tides/generator.h` — Main envelope/LFO generator
- `tides/tides.cc` — Main firmware

---

### Frames — Keyframer/Mixer

**Platform**: STM32F10x (72 MHz)

A keyframe animation sequencer and 4-channel mixer:

- Up to 64 keyframes per sequence
- 4 output channels
- Multiple easing curves (step, linear, quartic, sine, bounce)
- 8 palette slots for storage
- Euclidean rhythm generation
- Polyphonic LFO

**Parasites additions**: Sequencer step-editing mode, Shift Register sequencer mode (random canon generator).

**Key source files**:
- `frames/keyframer.cc/h` — Keyframe interpolation engine
- `frames/euclidean.cc/h` — Euclidean rhythm generation
- `frames/poly_lfo.cc/h` — Polyphonic LFO

---

### Peaks — Dual Trigger Converter

**Platform**: STM32F10x (72 MHz)

A dual-channel function generator with 12+ processor modes:

- Envelope (ADSR-like)
- LFO with tap tempo
- Drum synthesis: bass, snare, high-hat, FM drums
- Pulse shaper and randomizer
- Bouncing ball physics simulation
- Mini sequencer
- Number station (DTMF-style tone generation)

**Key source files**:
- `peaks/drums/` — Drum synthesis engines
- `peaks/modulations/` — Modulation sources
- `peaks/number_station/` — Tone generation
- `peaks/pulse_processor/` — Pulse shaping

---

### Streams — Dual Dynamics Gate

**Platform**: STM32F10x (72 MHz)

A dual-channel dynamics processor:

- Compression / expansion
- Envelope following with transient detection
- Vactrol response emulation
- ADSR envelope
- Lorenz attractor-based modulation (via SVF)

**Key source files**:
- `streams/processor.h` — Dual processing engine
- `streams/compressor.h` — Dynamics compression
- `streams/follower.h` — Envelope follower
- `streams/vactrol.h` — Vactrol response emulation
- `streams/svf.h` — State-variable filter / Lorenz generator

---

### Yarns — MIDI Interface

**Platform**: STM32F10x (72 MHz)

A MIDI-to-CV/Gate converter:

- 4 independent parts/voices
- Polyphonic, duophonic, monophonic layouts
- Custom pitch tables
- Clock swing and tempo control
- Wavetable oscillator with BLEP synthesis (saw, square, triangle, sine, noise)
- Trigger shapes: square, linear, exponential, ring, steps
- VCA envelope

**Key source files**:
- `yarns/multi.h` — Multi-mode MIDI processor
- `yarns/voice.h` — Individual voice with oscillator

---

### Grids — Topographic Drum Sequencer

**Platform**: AVR ATMega328P

A pattern-based drum sequencer:

- Topographic drum sequencing (X/Y/randomness parameter space)
- 32-step patterns with 3 parallel tracks (kick, snare, hat)
- Density control per track
- Drums vs. Euclidean output modes
- Swing and clock division
- MIDI interface

**Key source files**:
- `grids/pattern_generator.cc/h` — Drum pattern generation
- `grids/clock.cc/h` — Clock management

---

### Edges — Quad Chiptune Digital Oscillator

**Platform**: AVR ATMega32A4 (32 MHz)

A 4-voice chiptune synthesizer:

- Shapes: triangle, NES triangle, pitched noise, NES noise
- Voice allocation (up to 4 voices)
- Wavetable morphing
- MIDI note handling
- Hardware timer-based oscillator

**Key source files**:
- `edges/digital_oscillator.cc/h` — Chiptune synthesis
- `edges/timer_oscillator.cc/h` — Timer-based oscillators
- `edges/midi_handler.cc/h` — MIDI processing
- `edges/voice_allocator.cc/h` — Voice scheduling
- `edges/audio_buffer.cc/h` — Audio buffering

---

### Branches — Dual Bernoulli Gate

**Platform**: AVR ATMega88

A simple dual-channel probabilistic gate:

- Gate logic with probability control
- Dual-channel independent operation
- LED feedback (red/green/off states)
- Analog input scanning

**Source**: `branches/branches.cc`

---

### Hardware-Only Modules

These modules have no firmware — only hardware designs:

| Module | Description |
|---|---|
| **Links** | Utility (buffer, mixer) |
| **Ripples** | Liquid 2-pole BP, 2-pole LP, 4-pole LP filter |
| **Shades** | Triple attenuverter |
| **Shelves** | EQ filter |
| **Volts** | +5V power supply |

---

## Support Libraries

### stmlib

Git submodule from `https://github.com/mqtthiqs/stmlib.git`.

Common library for all STM32-based modules providing:

- `stmlib/system/` — STM32 system initialization, clocks, IRQ handling
- `stmlib/utils/` — DSP utilities (ring buffers, random, interpolation)
- `stmlib/dsp/` — Higher-level DSP: filters, oscillators, delay lines
- `stmlib/third_party/STM/CMSIS/` — ARM CMSIS DSP library

### avrlib

Git submodule from `https://github.com/pichenettes/avril.git`.

Common library for AVR modules (Grids, Branches): GPIO, ADC, UART, SPI, timing.

### avrlibx

Git submodule from `https://github.com/pichenettes/avrilx.git`.

Extended AVR library for Edges: enhanced I/O, timer/interrupt management, utility functions.

---

## Build System

### Toolchain Requirements

| Platform | Toolchain | Path |
|---|---|---|
| STM32 (ARM) | GCC ARM 4.8.3 (`arm-none-eabi-gcc`) | `/usr/local/arm-4.8.3/` (configurable via `TOOLCHAIN_PATH`) |
| AVR | `avr-gcc` + AVRDUDE | System default |
| Resource generation | Python 2.5+ | System default |
| Build orchestration | GNU Make | System default |

### Makefile Structure

Each module has its own makefile that specifies hardware parameters and includes the shared build rules:

```makefile
# Example: braids/makefile
F_CRYSTAL      = 8000000L
F_CPU          = 72000000L
SYSCLOCK       = SYSCLK_FREQ_72MHz
FAMILY         = f10x
DENSITY        = md
TARGET         = braids
PACKAGES       = braids braids/drivers stmlib/utils stmlib/system
RESOURCES      = braids/resources

include stmlib/makefile.inc
```

### Build Targets

```bash
# Compile firmware
make -f braids/makefile

# Clean build artifacts
make -f braids/makefile clean

# Upload via JTAG
make -f braids/makefile upload

# Generate .wav for audio bootloading (STM32 = QPSK, AVR = FSK)
make -f braids/makefile wav

# Generate .syx for MIDI update (Yarns)
make -f yarns/makefile syx

# Generate sample rate conversion filters (Warps)
make -f warps/makefile src
```

### Resource Generation

Each module's `resources/` directory contains Python scripts that generate embedded lookup tables, wavetables, and filter coefficients as C++ source:

```bash
python braids/resources/resources.py    # Generates braids/resources.cc and .h
python clouds/resources/src_filters.py  # Generates SRC filter coefficients
```

---

## Firmware Update Methods

| Method | Encoding | Modules | Description |
|---|---|---|---|
| **WAV (QPSK)** | Quadrature Phase-Shift Keying | All STM32 modules | 48 kHz 16-bit audio file played to module via audio jack |
| **WAV (FSK)** | Frequency-Shift Keying | All AVR modules | Audio-encoded firmware for AVR bootloader |
| **SysEx (MIDI)** | MIDI System Exclusive | Yarns (and others) | Firmware delivered over MIDI using `hex2sysex.py` |
| **JTAG** | Direct flash | All modules | Direct upload for development |

### Audio Bootloader Parameters

**STM32 (QPSK)**:
- Sample rate: 48000 Hz
- Bit rate: 12000
- Clock rate: 6000
- Page size: 256

**AVR (FSK)**:
- Sample rate: 40000 Hz
- Bit rate: 16
- Sync bytes: 8
- Zero bits: 4
- Page size: 128

---

## DSP Techniques

### Oscillator Synthesis (Braids)

- **Analog emulation** — Waveform morphing (saw/square/triangle)
- **Ring modulation** — Frequency mixing via multiplication
- **Comb filtering** — Feedback comb filter with sample & hold
- **Digital filters** — LP, BP, peak with feedback
- **Formant synthesis** — Vowel modeling with resonant filters
- **Karplus-Strong** — Plucked string physical modeling
- **FM synthesis** — Basic, feedback, and chaotic frequency modulation
- **Wavetable** — Lookup-table morphing with interpolation
- **Granular** — Particle-based texture synthesis

### Granular Processing (Clouds)

- **Grain synthesis** — Overlapping windowed sample playback
- **Phase vocoder** — STFT-based time-stretching
- **WSOLA** — Waveform Similarity Overlap-Add
- **Spectral morphing** — Frequency-domain interpolation
- **Pitch shifting** — Via phase vocoder or time-stretching

### Physical Modeling (Elements, Rings)

- **Modal resonance** — Parallel resonators at harmonic frequencies
- **Karplus-Strong strings** — Time-domain plucked string synthesis
- **Waveguide synthesis** — Tube/pipe modeling
- **Schroeder reverberator** — Cascaded delay + diffusion networks

### Filtering

- **State-Variable Filter (SVF)** — All-pass, LP, BP, HP modes
- **Biquad filters** — IIR cascade
- **Comb filters** — Feedback delay for resonance
- **Polyphase FIR** — Multi-rate sample rate conversion

### Sample Rate Conversion (Warps, Clouds)

- Polyphase FIR filters with pre-generated coefficients
- Configurable oversampling ratios (3×, 4×, 6×)

---

## Code Conventions

### Architecture

- **Namespacing**: Each module has its own namespace (`namespace braids { }`)
- **Block-based processing**: Audio rendered in blocks of 16–96 samples
- **No dynamic allocation**: Fixed-size buffers throughout for real-time safety
- **Static allocation**: No `malloc`/`new` in audio paths

### Naming

| Element | Convention | Example |
|---|---|---|
| Classes | PascalCase | `MacroOscillator`, `GranularProcessor` |
| Methods | CamelCase | `Render()`, `Process()` |
| Getters/Setters | snake_case | `set_pitch()`, `get_parameter()` |
| Constants | SCREAMING_SNAKE_CASE | `kNumParts`, `kBlockSize` |
| Files | snake_case | `digital_oscillator.cc`, `macro_oscillator.h` |

### DSP Class Pattern

```cpp
class DspEngine {
 public:
  void Init();                              // One-time initialization
  void Process(const Input& in, Output* out, size_t size);  // Per-block
  inline void set_parameter(Type value) { param_ = value; }

 private:
  // State variables persist across blocks
  // Fixed-size arrays only — no heap allocation
};
```

### Fixed-Point Arithmetic

Integer DSP dominates (16-bit and 32-bit fixed-point). Implicit Q-notation scaling (e.g., `>> 15` divides by 32768). Table lookups use fractional phase indexing for interpolation.

---

## Testing

Test files are located in each module's `test/` directory:

- `braids/test/braids_test.cc` — Oscillator unit tests
- `clouds/test/clouds_test.cc` — Granular processor tests
- `elements/test/elements_test.cc` — Modal synthesis tests

Build and run tests:

```bash
make -f braids/test/makefile
```

Tests use a custom C++ framework (no external test dependencies).

---

## Hardware Design

Each module includes a `hardware_design/` directory containing schematics, PCB layouts, bills of materials, and mechanical drawings (typically in KiCAD or Eagle format). These are reference designs, not firmware.

---

## Build Summary

| Module | Platform | Toolchain | Bootloader | Update Method |
|---|---|---|---|---|
| Braids | STM32F10x | ARM GCC | QPSK | WAV / JTAG |
| Clouds | STM32F4xx | ARM GCC | QPSK | WAV / JTAG |
| Elements | STM32F4xx | ARM GCC | QPSK | WAV / JTAG |
| Rings | STM32F4xx | ARM GCC | QPSK | WAV / JTAG |
| Warps | STM32F4xx | ARM GCC | QPSK | WAV / JTAG |
| Tides | STM32F10x | ARM GCC | QPSK | WAV / JTAG |
| Frames | STM32F10x | ARM GCC | QPSK | WAV / JTAG |
| Peaks | STM32F10x | ARM GCC | QPSK | WAV / JTAG |
| Streams | STM32F10x | ARM GCC | QPSK | WAV / JTAG |
| Yarns | STM32F10x | ARM GCC | SysEx | SysEx (MIDI) |
| Grids | ATMega328P | AVR GCC | FSK | WAV / ISP |
| Branches | ATMega88 | AVR GCC | FSK | WAV / ISP |
| Edges | ATMega32A4 | AVR GCC | FSK | WAV / ISP |
