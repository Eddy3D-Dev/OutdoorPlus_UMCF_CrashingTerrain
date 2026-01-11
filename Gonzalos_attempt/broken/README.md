# Debugging Summary: FPE Crash Fix

This README documents the resolution of the Floating Point Exception (FPE) crash in this case.

## Problem
The simulation was crashing in the `realizableKE` turbulence model due to unphysical field values (negative k/epsilon). This was verified to be a symptom of a numerical instability caused by improper boundary conditions.

## Root Cause
1.  **Floating Pressure**: The `p_rgh` boundary condition at `top` was set to `fixedFluxPressure` (Neumann), while the outlet was `inletOutlet`. This left the domain with no fixed pressure anchor, causing the pressure field to drift and diverge.
2.  **Invalid Wall Types**: `terrain` and `buildings` were set to `mappedWall` (multi-region coupling) but without the corresponding `fvOptions` to handle the mapping in this single-region test.
3.  **Inlet Compatibility**: The original `atmBoundaryLayerInletVelocity` was causing compatibility issues.

## Solution Applied
The following changes stabilized the simulation (verified with `buoyantSimpleFoam`):

1.  **Fixed Pressure Anchor**:
    *   `0/p_rgh` @ `top`: Changed from `fixedFluxPressure` to `fixedValue` (value `1e5`).
2.  **Standardized Walls**:
    *   `constant/polyMesh/boundary`: Changed `terrain` and `buildings` from `mappedWall` to standard `wall`.
3.  **Simplified Boundaries** (for debugging):
    *   `0/U` @ `front`: Set to `fixedValue`.
    *   `0/U` @ `left`/`right`/`top`: Set to `slip` (wind tunnel setup).
4.  **Robust Initialization**:
    *   Increased initial `k` (1.0) and `epsilon` (0.1) in `0/` directory.
    *   Run `potentialFoam -writePhi` before the main solver to initialize fluxes.

## How to Run
1.  Source OpenFOAM (e.g., `source /opt/openfoam8/etc/bashrc`).
2.  Clean previous results: `rm -rf [1-9]* 0/phi`.
3.  Initialize flux: `potentialFoam -writePhi`.
4.  Run solver: `buoyantSimpleFoam`.

## Troubleshooting
*   **Thermal Divergence**: If the simulation crashes later (e.g., T=30+) with a temperature/thermo error, this is likely due to the Energy equation diverging. To fix this:
    *   Add a `limitTemperature` source in `system/fvOptions`.
    *   Or further reduce relaxation factors for `h` / `e` in `system/fvSolution`.
