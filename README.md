# DuSub-GS

Dual-Subspace Motion Modeling for Robust Dynamic Gaussian Splatting 


\abstract{
Reconstructing dynamic scenes from monocular videos remains a challenging problem due to motion modeling methods generally suffering from temporal instability, boundary artifacts, or geometric collapse. 
% 
To address this issue, we propose a principled motion decomposition framework for monocular dynamic scene reconstruction based on dynamic Gaussian Splatting. Our method explicitly separates motion into two complementary components: a Chebyshev field that models globally stable, low-frequency motion trends, and a Residual Motion field that captures localized, high-frequency dynamics outside the spectral subspace.
%
The novel Chebyshev field module leverages spectral representations on bounded, non-periodic temporal intervals to ensure temporal coherence and suppress boundary oscillations, while the residual motion field focuses on structured local dynamics without interfering with global stability. We further design a joint optimization objective that enforces appearance consistency, geometric plausibility, and residual regularization, ensuring stable and physically meaningful reconstruction.
%
Extensive experiments on challenging monocular dynamic scenes demonstrate that the proposed framework significantly improves temporal stability, reconstruction fidelity, and motion boundary sharpness compared to existing dynamic Gaussian and neural deformation methods. 
% Our results highlight the importance of explicit motion decomposition as a foundation for robust and interpretable dynamic scene reconstruction.
}


# DuSub-GS: Dual-Subspace Motion Modeling for Robust Dynamic Gaussian Splatting

[![Project Page](https://img.shields.io/badge/Project-Page-2ea44f)](https://dusub-gs.github.io/)
[![Code](https://img.shields.io/badge/Code-GitHub-24292f)](https://github.com/EllisonKing/DuSub-GS)
[![Paper](https://img.shields.io/badge/Paper-Project_Page-b31b1b)](https://dusub-gs.github.io/)
[![arXiv](https://img.shields.io/badge/arXiv-Project_Page-b31b1b)](https://dusub-gs.github.io/)

Official implementation of **DuSub-GS: Dual-Subspace Motion Modeling for Robust Dynamic Gaussian Splatting**.

DuSub-GS decomposes dynamic Gaussian motion into two complementary subspaces: a Chebyshev field for globally stable low-frequency motion and a residual motion field for localized high-frequency dynamics. This decomposition improves temporal stability while preserving the flexibility needed for complex non-rigid motion.

> The project page is available at https://dusub-gs.github.io/. The paper and arXiv badges above point to the project page; replace them with the direct paper/arXiv URLs after final publication if desired.

## Abstract

Monocular dynamic scene reconstruction is difficult because deformation models often trade temporal stability for local flexibility, leading to jitter, boundary artifacts, or unstable geometry. DuSub-GS addresses this by decomposing Gaussian motion into a globally stable Chebyshev transformation field and a localized residual motion field. The Chebyshev branch captures smooth low-frequency trends over bounded non-periodic time intervals, while the residual branch focuses on high-frequency local dynamics outside the spectral subspace. A joint optimization objective combines appearance, geometry, and residual regularization to produce temporally coherent and detail-preserving dynamic Gaussian reconstructions.

## Framework

![DuSub-GS framework](assets/framework.png)

## Installation

```bash
git clone https://github.com/EllisonKing/DuSub-GS.git
cd DuSub-GS
git submodule update --init --recursive

conda create -n dusubgs python=3.10
conda activate dusubgs

pip install torch==2.7.1 torchvision==0.22.1 torchaudio==2.7.1 \
  --index-url https://download.pytorch.org/whl/cu128
pip install -r requirements.txt
pip install -e submodules/diff-gaussian-rasterization/
pip install -e submodules/simple-knn/
```

The local development environment is recorded in `install-envs.sh`. Adjust the PyTorch CUDA wheel according to your CUDA driver if needed.

## Data Preparation

The following paths are the dataset paths used in our experiments. They are written as repository-relative data roots; place the datasets there or create symlinks with the same layout.

```text
HyperNeRF:        data_3dgs/HyperNeRF/vrig
NeRF-DS:          data_3dgs/NeRF-DS
DyCheck-iPhone:   data_3dgs/processed_iphone
```

The HyperNeRF and Neural 3D Video processing scripts are copied from the E-D3DGS-style preprocessing pipeline. The DyCheck-iPhone COLMAP conversion follows the MoDec-GS DyCheck processing pipeline and is included in this repository as `scripts/dycheck2colmap.py`.

### HyperNeRF

A processed HyperNeRF scene should contain `rgb/2x`, `camera`, `dataset.json`, `metadata.json`, `scene.json`, and `points3D_downsample2.ply`. Example scene:

```text
data_3dgs/HyperNeRF/vrig/vrig-peel-banana
```

To prepare COLMAP metadata and the initial point cloud:

```bash
python scripts/pre_hypernerf.py --videopath data_3dgs/HyperNeRF/vrig/vrig-peel-banana
python scripts/downsample_point.py \
  data_3dgs/HyperNeRF/vrig/vrig-peel-banana/points3d.ply \
  data_3dgs/HyperNeRF/vrig/vrig-peel-banana/points3D_downsample2.ply
```

If the released scene already contains `points3D_downsample2.ply`, the second command is not needed.

### NeRF-DS

A processed NeRF-DS scene follows the same Nerfies/DyCheck-style layout expected by the loader. Example scene:

```text
data_3dgs/NeRF-DS/bell_novel_view
```

The scene should contain:

```text
camera/
rgb/2x/
dataset.json
metadata.json
scene.json
points3D_downsample2.ply
```

To prepare COLMAP metadata and the initial point cloud:

```bash
bash colmap.sh data_3dgs/NeRF-DS/bell_novel_view nerfds
python scripts/downsample_point.py \
  data_3dgs/NeRF-DS/bell_novel_view/points3d.ply \
  data_3dgs/NeRF-DS/bell_novel_view/points3D_downsample2.ply
```

If the released scene already contains `points3D_downsample2.ply`, the second command is not needed.

### Neural 3D Video

For Neural 3D Video, set `N3V_SCENE_DIR` to an existing local scene directory and run the E-D3DGS-style preprocessing script:

```bash
python scripts/pre_n3v.py --videopath "$N3V_SCENE_DIR"
python scripts/downsample_point.py \
  "$N3V_SCENE_DIR/points3d.ply" \
  "$N3V_SCENE_DIR/points3D_downsample.ply"
```

If the Neural 3D Video scene stores its raw point cloud under another `.ply` name, pass that file as the first argument to `scripts/downsample_point.py`.

The current data root does not include a verified Neural 3D Video path, so no synthetic path is listed here.

For the `coffee_martini` scene, remove `cam13.mp4` and the corresponding pose before running preprocessing.

### DyCheck-iPhone

The processed iPhone data used in our experiments is under:

```text
data_3dgs/processed_iphone
```

Example scene:

```text
data_3dgs/processed_iphone/apple
```

The expected scene layout includes:

```text
camera/
colmap/
covisible/2x/
depth/2x/
rgb/2x/
splits/
dataset.json
metadata.json
scene.json
points3D_downsample2.ply
```

To regenerate COLMAP metadata:

```bash
bash colmap.sh data_3dgs/processed_iphone/apple dycheck
```

The processed iPhone scenes used in our experiments already include `points3D_downsample2.ply`.

## Training

### HyperNeRF

```bash
CUDA_VISIBLE_DEVICES=0 python train.py \
  -s data_3dgs/HyperNeRF/vrig/vrig-peel-banana \
  --port 6000 \
  --model_path output-dusubgs/hypernerf/vrig-peel-banana \
  --expname hypernerf/vrig-peel-banana \
  --configs arguments/hypernerf-new/vrig-peel-banana.py \
  -r 1
```

### NeRF-DS

```bash
CUDA_VISIBLE_DEVICES=0 python train.py \
  -s data_3dgs/NeRF-DS/bell_novel_view \
  --port 7000 \
  --model_path output-dusubgs/NeRF-DS/bell_novel_view \
  --expname NeRF-DS/bell_novel_view \
  --configs arguments/NeRF-DS-new/bell_novel_view.py \
  -r 1
```

You can also use the provided batch scripts after checking the dataset root, output root, and scene list:

```bash
bash train_hypernerf.sh 0
bash train_nerfds_1.sh 0
```

## Rendering

Render test views only:

```bash
CUDA_VISIBLE_DEVICES=0 python render.py \
  --model_path output-dusubgs/hypernerf/vrig-peel-banana \
  --configs arguments/hypernerf-new/vrig-peel-banana.py \
  --skip_train \
  --skip_video
```

Render train views, test views, and video trajectory:

```bash
CUDA_VISIBLE_DEVICES=0 python render.py \
  --model_path output-dusubgs/hypernerf/vrig-peel-banana \
  --configs arguments/hypernerf-new/vrig-peel-banana.py
```

Rendered outputs are saved under:

```text
<model_path>/train/ours_<iteration>/
<model_path>/test/ours_<iteration>/
<model_path>/video/ours_<iteration>/
```

## Evaluation

Standard PSNR / SSIM / LPIPS evaluation:

```bash
CUDA_VISIBLE_DEVICES=0 python metrics.py \
  -m output-dusubgs/hypernerf/vrig-peel-banana
```

The results are written to:

```text
<model_path>/results.json
<model_path>/per_view.json
```

## Links

- Project page: https://dusub-gs.github.io/
- Code: https://github.com/EllisonKing/DuSub-GS
- Paper: see the project page
- arXiv: see the project page

## Citation

```bibtex
@article{hu2026dusubgs,
  title={Dual-Subspace Motion Modeling for Robust Dynamic Gaussian Splatting},
  author={Hu, Bingbing and Li, Yan and Yang, Yong and Guo, Shihui and Yao, Junfeng and Wang, Hesheng},
  journal={International Journal of Computer Vision},
  year={2026}
}
```

Please update the BibTeX with the final publication metadata once it is available.

## Acknowledgements

This codebase builds on prior work including [3D Gaussian Splatting](https://github.com/graphdeco-inria/gaussian-splatting), [4DGaussians](https://github.com/hustvl/4DGaussians), [E-D3DGS](https://github.com/JeongminB/E-D3DGS), [MoDec-GS](https://github.com/skwak-kaist/MoDec-GS), and [DyCheck](https://github.com/KAIR-BAIR/dycheck). We thank the authors for releasing their code and datasets.

## License

Please see `LICENSE.md`. This repository inherits non-commercial research-use restrictions from its upstream dependencies where applicable.

