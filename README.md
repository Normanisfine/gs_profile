# 3DGS Profiling Fork

This is a modified fork of [3D Gaussian Splatting](https://github.com/graphdeco-inria/gaussian-splatting) with **NVTX profiling markers** added for performance analysis with NVIDIA Nsight Systems.

## What I Modified

### 1. NVTX Profiling Markers

Added NVTX markers to the CUDA rasterizer to measure per-stage GPU kernel time:

| Stage | Marker | What It Measures |
|-------|--------|------------------|
| **Preprocessing** | `Preprocessing` | 3D→2D projection, covariance computation, SH→RGB |
| **TileBinning** | `TileBinning` | Prefix sum, tile-Gaussian pair generation |
| **Sorting** | `Sorting` | Radix sort by tile ID and depth |
| **AlphaBlending** | `AlphaBlending` | Per-tile alpha compositing |

**File Modified**: `submodules/diff-gaussian-rasterization/cuda_rasterizer/rasterizer_impl.cu`

### 2. FPS Benchmark Script

Added `render_fps.py` for measuring rendering throughput without profiling overhead.

## Key Files

| File | Description |
|------|-------------|
| [`submodules/diff-gaussian-rasterization/NVTX_MARKERS.md`](submodules/diff-gaussian-rasterization/NVTX_MARKERS.md) | Documentation of NVTX markers |
| [`render_fps.py`](render_fps.py) | FPS benchmarking script |
| [`submodules/diff-gaussian-rasterization/cuda_rasterizer/rasterizer_impl.cu`](submodules/diff-gaussian-rasterization/cuda_rasterizer/rasterizer_impl.cu) | Modified CUDA code with NVTX markers |

## Usage

### Run FPS Benchmark
```bash
python render_fps.py -m <path_to_trained_model> --iteration 30000
```

### Profile with Nsight Systems
```bash
nsys profile --trace=cuda,nvtx -o profile_output \
    python render_fps.py -m <path_to_trained_model> --iteration 30000
```

Then open the `.nsys-rep` file in Nsight Systems GUI to see timing breakdown by NVTX marker.

## Profiling Results (Garden Scene, 1036×1600)

| Stage | GPU Time |
|-------|----------|
| Preprocessing | 2.66 ms |
| TileBinning | 1.28 ms |
| Sorting | 4.28 ms |
| AlphaBlending | 11.62 ms |
| **Total** | **~19.8 ms** (~35 FPS) |

---

## Original Repository

This fork is based on the official 3D Gaussian Splatting implementation:
- **Paper**: [3D Gaussian Splatting for Real-Time Radiance Field Rendering](https://repo-sam.inria.fr/fungraph/3d-gaussian-splatting/)
- **Authors**: Bernhard Kerbl, Georgios Kopanas, Thomas Leimkühler, George Drettakis (INRIA)
- **Original Repo**: https://github.com/graphdeco-inria/gaussian-splatting

```bibtex
@Article{kerbl3Dgaussians,
  author       = {Kerbl, Bernhard and Kopanas, Georgios and Leimk{\"u}hler, Thomas and Drettakis, George},
  title        = {3D Gaussian Splatting for Real-Time Radiance Field Rendering},
  journal      = {ACM Transactions on Graphics},
  number       = {4},
  volume       = {42},
  month        = {July},
  year         = {2023},
}
```
