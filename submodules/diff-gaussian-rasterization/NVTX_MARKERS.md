# NVTX Markers for Gaussian Splatting Profiling

This document describes the NVTX markers added to the diff-gaussian-rasterization module for profiling with nsys.

## File Modified

- `cuda_rasterizer/rasterizer_impl.cu`

## NVTX Markers

### 1. Preprocessing
**Marker:** `Preprocessing`

**Location:** Around `FORWARD::preprocess()` call

**Function:** Performs per-Gaussian operations before rasterization:
- Transform 3D Gaussians to 2D screen space
- Compute 3D covariance matrices from scale/rotation
- Project to 2D covariance matrices
- Frustum culling (remove Gaussians outside view)
- Convert spherical harmonics to RGB colors
- Compute bounding rectangles for tile overlap

### 2. TileBinning
**Marker:** `TileBinning`

**Location:** Around prefix sum and `duplicateWithKeys()` calls

**Function:** Assigns Gaussians to screen tiles:
- Compute prefix sum of tiles touched per Gaussian
- Generate tile-Gaussian key-value pairs
- Each Gaussian maps to multiple tiles it overlaps

### 3. Sorting
**Marker:** `Sorting`

**Location:** Around `cub::DeviceRadixSort::SortPairs()` and `identifyTileRanges()` calls

**Function:** Sorts Gaussians for front-to-back rendering:
- Radix sort by tile ID and depth
- Identify start/end ranges for each tile's Gaussian list

### 4. AlphaBlending
**Marker:** `AlphaBlending`

**Location:** Around `FORWARD::render()` call

**Function:** Per-tile alpha compositing:
- Each tile processes its Gaussians front-to-back
- Compute Gaussian contribution per pixel
- Accumulate colors with alpha blending
- Early termination when transmittance is low

## Usage with nsys

```bash
nsys profile --trace=cuda,nvtx -o profile_output python render.py ...
```

Then view in Nsight Systems GUI to see timing breakdown by marker.

