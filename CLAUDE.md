# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a GATE (Geant4 Application for Tomographic Emission) simulation for PET (Positron Emission Tomography) imaging systems. The project simulates a total-body PET scanner based on the uEXPLORER system, developed by researchers at Brigham and Women's Hospital/Harvard Medical School and UC Davis.

**Contact**: Auer, Benjamin Ph.D <bauer@bwh.harvard.edu>

## Running Simulations

### Production Runs (Full Statistics)
```bash
Gate main.mac
```

### Visualization Mode (Low Statistics)
```bash
# Interactive mode for geometry visualization and particle tracking
QT_QPA_PLATFORM=xcb Gate

# Then in the GATE prompt:
Idle> /control/execute main_visu_working.mac

# To restart with new particles:
Idle> /control/execute restart_working.mac

# To exit:
Idle> exit
```

### Analysis
```bash
python ./runAnalysis.py output
```

## Simulation Architecture

### Macro File Hierarchy

The simulation is organized through a main macro file that executes sub-macros in a specific order:

**main.mac** (production) or **main_visu_working.mac** (visualization):
1. **mac/verbose.mac** - Controls output verbosity levels
2. **mac/world.mac** - Defines the experimental world volume (must contain entire PET system)
3. **mac/pet_head.mac** - Defines detector geometry hierarchy (Head → Module → Block → Crystal → LSO layer)
4. **mac/pet_digitizer.mac** - Configures digitizer chain (adder, readout, energy resolution, coincidence sorter)
5. **mac/cylindrical_phantom.mac** - Defines attenuation phantom
6. **mac/physics.mac** or **mac/physics_visu.mac** - Physics list and production cuts
7. **mac/output.mac** - Output configuration (ROOT files, statistics)
8. **mac/sources.mac** or **mac/sources_visu.mac** - Radiation sources (F-18, O-15)
9. **mac/visu.mac** (visualization only) - Viewer setup before initialization
10. **mac/visu_draw.mac** (visualization only) - Drawing after initialization

### Critical Execution Order

1. Material database must be loaded before geometry
2. Geometry must be defined before `/gate/run/initialize`
3. Visualization viewer setup (visu.mac) must run **BEFORE** initialization
4. Visualization drawing (visu_draw.mac) must run **AFTER** initialization
5. Sources must be defined after initialization
6. Output configuration should precede initialization

### Volume Hierarchy

```
world (gray wireframe)
└── cylindricalPET (rotating scanner, Air)
    ├── head (8 heads in ring, Air)
    │   └── module (4x4 array, Air)
    │       └── block (5x5 array, Air)
    │           └── crystal (5x5 array, Air)
    │               └── LSO (sensitive detector, LSO material)
    └── phantom (water cylinder for attenuation)
```

- **Sensitive Detector**: Only LSO layer records interactions (`/gate/LSO/attachCrystalSD`)
- **System Attachment**: Hierarchy attached to cylindricalPET system via `/gate/systems/cylindricalPET/...`

### Digitizer Chain

1. **adder** - Sums energy deposits in crystal
2. **readout** - Groups signals at depth 1
3. **energyResolution** - Applies 26% FWHM at 511 keV
4. **energyFraming** - Energy window 350-650 keV
5. **CoincidenceSorter** - 120 ns coincidence window, takeWinnerOfGoods policy
6. **delay** - Delayed coincidences (500 ns offset)

Output collections: `Singles_LSO`, `Coincidences`, `delay`

### Physics Configuration

- **Physics List**: emstandard_opt3 (electromagnetic processes)
- **Production Cuts**:
  - World: 1 km (high cut, particles not tracked in air)
  - Phantom: 1.0 mm
  - LSO: 1.0 mm
- **Visualization Physics** (physics_visu.mac): Lower cuts for better particle tracking visibility

### Output Files

Located in `output/` directory:
- **pet.root** - ROOT file with Singles and Coincidences trees
- **stat.txt** - Simulation statistics (events, runtime, particle counts)
- **digit_summary.txt** - Summary of Singles and Coincidences

ROOT file flags (mac/output.mac):
- Hits: disabled (setRootHitFlag 0)
- Singles: enabled (setRootSinglesFlag 1)
- Coincidences: enabled (setRootCoincidencesFlag 1)

## Common Issues and Solutions

### Visualization Crashes

**Problem**: Vector assertion error `std::vector<_Tp, _Alloc>::operator[]` when running visualization

**Root Causes**:
1. Event accumulation limit exceeded (default: 10 events)
2. GateHitTree vector access out of bounds when too many events are kept
3. Digitizer processing conflicts with visualization

**Solutions**:
1. Set unlimited event accumulation before running:
   ```
   /vis/scene/endOfEventAction accumulate -1
   ```
2. Use low-activity source for visualization (sources_visu.mac: 1000 Bq instead of 100000 Bq)
3. Run short time slices (1 second instead of 60+ seconds)
4. Consider disabling output.mac in visualization mode (add `/gate/output/allowNoOutput`)

### Missing Digit Output

**Problem**: No Singles or Coincidences in output ROOT file

**Common Causes**:
1. Digitizer not initialized before `/gate/run/initialize`
2. Energy window cuts out all events (check 350-650 keV window)
3. Output flags disabled in output.mac
4. Simulation terminated early (check for crashes)
5. Source activity too low or acquisition time too short
6. `/gate/output/allowNoOutput` called (suppresses output warnings but doesn't create files)

**Debug Steps**:
1. Check stat.txt for event counts
2. Enable verbose output: `/run/verbose 2`, `/event/verbose 1`
3. Verify Singles_LSO collection exists: check digitizer chain configuration
4. Ensure `setRootSinglesFlag` and `setRootCoincidencesFlag` are set to 1

### Modified Files (Git Status)

Current modifications indicate ongoing visualization debugging:
- `mac/output.mac` - Output configuration changes
- `mac/visu.mac` - Visualization settings adjustments
- `main.mac` - Main simulation changes

## Material Database

**data/GateMaterials.db** contains all material definitions. Common materials:
- LSO (Lutetium Oxyorthosilicate) - detector crystal
- Air - used for container volumes
- Water - phantom material

Add new materials here before using them in geometry definitions.

## Source Definitions

**Production** (mac/sources.mac):
- F-18 line source: 100,000 Bq, 6586.2 s half-life, 0.5 mm radius, 34 cm height
- O-15 line source: 100,000 Bq, 122.24 s half-life

**Visualization** (mac/sources_visu.mac):
- Reduced to ~1,000 Bq for manageable event counts

Alternative sources:
- Voxelized phantoms from https://github.com/BenAuer2021/Phantoms-For-Nuclear-Medicine-Imaging-Simulation
- STL-based XCAT phantom from https://github.com/BenAuer2021/Mesh-based-Human-Phantom-for-Simulation

## Reconstruction

Analysis and reconstruction use external tools:
- **Analysis**: Python script `runAnalysis.py` (requires uproot, gatetools, matplotlib, itk)
- **Reconstruction**: CASToR software (https://castor-project.org/documentation_v3)

CASToR provides GATE-to-CASToR conversion tools for histogram and list-mode reconstruction with Time-Of-Flight modeling.

## Key GATE Commands

### Random Number Generation
```
/gate/random/setEngineName MersenneTwister
/gate/random/setEngineSeed auto
```

### Acquisition Control
```
/gate/application/setTimeStart 0 s
/gate/application/setTimeSlice 60 s
/gate/application/setTimeStop 120 s
/gate/application/startDAQ
```

### Geometry Repetition
- `cubicArray` - Repeat volumes in 3D grid
- `ring` - Repeat volumes in circular pattern
- `orbiting` - Rotate geometry during acquisition

## Dependencies

- **GATE** (Geant4 Application for Tomographic Emission)
- **Geant4** (physics simulation toolkit)
- **ROOT** (data analysis framework for output files)
- **Python packages**: uproot, gatetools, matplotlib, scipy, numpy, itk, click

## Troubleshooting Workflow

1. **Always test with low statistics first**: 1 second acquisition, 1000 Bq source
2. **Check geometry**: Use visualization mode to verify detector and phantom placement
3. **Verify digitizer chain**: Ensure Singles_LSO collection is created
4. **Monitor statistics**: Check stat.txt for reasonable event counts
5. **Validate output**: Verify ROOT file contains expected trees before long simulations
