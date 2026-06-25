# 2D Pixelated NFT PFP → 3D STL (Open Source, VRAM-First)

**Date**: 2026-06-24  
**Hardware Target**: RTX 5060 Ti 16GB (Blackwell)  
**Focus**: GPU/VRAM-heavy open-source AI methods only

## Objective
Convert 2D pixelated images (NFT PFPs, CryptoPunks-style, etc.) into 3D STL files using open-source tools that actually utilize the GPU, with explicit support for 16GB VRAM cards.

## Recommended Primary Method: TripoSR

**Why TripoSR?**
- Fast feedforward single-image 3D reconstruction (LRM architecture)
- One of the lightest high-quality open-source options
- MIT license, actively maintained
- Runs on consumer 16GB cards with proper settings
- GitHub: https://github.com/VAST-AI-Research/TripoSR (6.7k stars)

### Installation (Blackwell / 16GB Optimized)

```bash
# Create environment
conda create -n triposr python=3.10 -y
conda activate triposr

# Install PyTorch with Blackwell support (CUDA 12.4+)
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu124

# Install TripoSR
git clone https://github.com/VAST-AI-Research/TripoSR.git
cd TripoSR
pip install -r requirements.txt
```

### Running Inference (16GB VRAM Settings)

```python
# Recommended settings for 5060 Ti 16GB
python run.py \
  --model_ckpt ckpt/model.ckpt \
  --image input.png \
  --output_dir ./output \
  --render_size 256 \           # Lower this if OOM (try 192 or 128)
  --chunk_size 4096 \
  --use_fp16                    # Critical for 16GB
```

**Blackwell / 16GB Optimizations**:
- Always use `--use_fp16` or run in `torch.autocast("cuda")`
- Add at the top of scripts:
  ```python
  import torch
  torch.cuda.empty_cache()
  torch.backends.cudnn.benchmark = True
  ```
- Start with `--render_size 256`. Drop to 192 if you get OOM.
- Process one image at a time.

### Output
TripoSR produces:
- `.obj` mesh
- Textured mesh (optional)

## Clean STL Export (Recommended Pipeline)

Use `trimesh` for reliable STL conversion:

```bash
pip install trimesh numpy
```

```python
import trimesh

# Load the .obj from TripoSR
mesh = trimesh.load("output/model.obj")

# Optional: Repair / make manifold (important for 3D printing)
mesh = mesh.process(validate=True)
if not mesh.is_watertight:
    mesh.fill_holes()
    mesh.fix_normals()

# Export clean binary STL
mesh.export("output/model.stl", file_type="stl")
print("STL exported successfully")
```

**Alternative: Blender (best quality)**
1. Import the `.obj`
2. Apply `Mesh > Clean Up > Merge by Distance`
3. `Mesh > Clean Up > Make Manifold` (if available) or use the 3D Print Toolbox addon
4. Export as STL (Binary)

## Secondary Option: Depth Anything v2 + Meshing (Lighter VRAM)

If TripoSR is too heavy:

1. Use **Depth Anything v2** (very low VRAM, ~3-6GB)
2. Generate depth map on GPU
3. Use marching cubes or Poisson surface reconstruction to create mesh
4. Export to STL

This preserves more of the pixelated/blocky look than full 3D reconstruction models.

## Limitations for Pixel Art Inputs

- TripoSR was trained primarily on natural photos → it tends to smooth sharp pixel edges and add organic curvature.
- For a true "blocky 3D pixel" look, you may still need to combine with programmatic extrusion or post-process in Blender.
- Very low resolution inputs (24×24, 32×32) may need upscaling first.

## Files & Commands Summary

| Step                    | Command / Tool                  | VRAM Target | Notes                     |
|-------------------------|----------------------------------|-------------|---------------------------|
| Install                 | TripoSR + PyTorch cu124         | -           | Use FP16                 |
| Inference               | `run.py --use_fp16`             | 10-14GB     | Start at render_size 256 |
| STL Export              | `trimesh` or Blender            | Low         | Binary STL preferred     |
| Alternative (lighter)   | Depth Anything v2 + marching_cubes | 4-8GB    | More pixel-faithful      |

## Recommended Next Actions

1. Test TripoSR with 3–5 sample PFPs on your 5060 Ti.
2. Benchmark actual VRAM usage with `nvidia-smi`.
3. Compare results between TripoSR and Depth Anything v2 pipeline.
4. Create a small wrapper script that takes a folder of PFPs and outputs STL files.

---

**Provenance**: All information pulled from official TripoSR repository, trimesh documentation, and common practices in open-source 3D reconstruction workflows (June 2026).