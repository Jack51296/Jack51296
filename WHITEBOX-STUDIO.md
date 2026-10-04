# Whitebox Studio

**Private case study · Procedural whitebox video production and V2V workflow system**

[返回个人主页](./README.md)

**项目场景：** 将故事、结构树或参考视频转为可编辑的白模场景，用于镜头预演、路线与机位验证、批量白模视频制作和 V2V 提交准备。核心实现保持私有。

Whitebox Studio is a Python and Blender platform for building, analyzing, rendering, reviewing and packaging whitebox video. It treats `scene.json` as the shared data contract for forward construction, reverse analysis, rendering, quality control and text deliverables.

## Production Scenarios

### Story-driven previsualization

The forward workflow turns a structure tree, story brief or long-take brief into a validated scene contract. Route intent, camera intent and subject constraints are compiled into frame-level keyframes instead of being hand-authored independently for every shot.

Typical uses include:

- fast previsualization of character, vehicle and environment relationships;
- testing camera grammar, shot scale, motion limits and story beats before a final production;
- generating batches of scene variations with the same reproducible rules;
- exporting review videos, storyboards, director cards, route maps and shot transition tables.

### Reference-video analysis

The reverse workflow analyzes an authorized reference video and produces a structured scene draft:

```text
video → frame extraction → shot detection → optical flow / occupancy / structure lines
      → camera-motion and geometry analysis → scene.json draft → static gates
```

This supports shot breakdown, camera-motion study, whitebox reconstruction and later V2V planning. The system records which detector or solver actually ran and why a fallback was selected.

### Whitebox-to-live-action V2V

The V2V Skill v3 workflow keeps the shot package explicit:

```text
planning → frame extraction → image generation → binding and packaging → submission package
```

The default execution is mock or export-only. The package preserves image references, prompts, manifests and failure states so a human can submit the package to a video platform without silently changing the shot contract.

## Concrete Technology

### Core runtime and contracts

`Python 3.11+` · `Typer CLI` (`wbs`) · `Pydantic 2` · `JSON Schema` · `PyYAML` · `Jinja2` · `HTTPX` · `NumPy` · `OpenCV` · `Pillow` · `OpenPyXL` · `SQLite` · `pytest` · `ruff`

`scene.json`, `story_plan` and `longtake_plan` are validated contracts. SQLite records input hashes, step states, model usage, cost records and human review states, allowing idempotent reruns and isolated failure recovery.

### 3D and media pipeline

`Blender 5.1.2` · `Blender Python API` · `Workbench` · `EEVEE` · `FFmpeg` · `H.264` · `FBX` · `GLB` · headless rendering

The builder creates scenes, writes frame-level keyframes, audits the active camera and renders without a GUI. The export path supports FBX, GLB and shot transition tables.

### Vision and analysis routes

| Capability | Technology | Role |
| --- | --- | --- |
| Shot detection | `TransNetV2` · `PySceneDetect` · builtin fallback | Detect cuts with a registered fallback chain |
| Face privacy | `CenterFace` · OpenCV DNN · YuNet path | Detect faces for review and optional masking |
| Text-prompted subjects | `GroundingDINO` | Produce subject boxes from prompts |
| Subject masks | `SAM 2.1` | Refine boxes into masks in the isolated worker |
| Depth and camera pose | `DA3` | Estimate multi-view depth and camera pose |
| Metric geometry | `MapAnything` | Optional multi-view metric reconstruction |
| Motion | `OpenCV` optical flow and camera-motion analysis | Estimate movement and camera behavior |
| Runtime isolation | `ONNX Runtime` + `.venv-vision` | Keep Torch / Transformers dependencies out of the production interpreter |

`TransNetV2`, `PySceneDetect`, `CenterFace`, `GroundingDINO`, `SAM 2.1` and `DA3` have registered implementation paths. `MapAnything` is registered as an optional Apache-licensed geometry route. Heavy models communicate with the main CLI through JSON / NPZ file exchange; missing models, timeouts and worker failures fall back to builtin, motion or heuristic routes.

### Provider and deployment layer

`OpenAI-compatible provider` · `Mock provider` · `Manual video export` · `HTTP generic video adapter` · `Cosmos Transfer 2.5 adapter` · `Wan2.2 VACE adapter` · `Docker` · `NVIDIA Container Toolkit` · GPU / CPU fallback

The provider layer reads credentials from environment variables, supports dry-run and paid-call gates, and records model identifiers without printing credentials. Cosmos Transfer 2.5 and Wan2.2 VACE are integration points, not default live services.

### Model runtime and supporting dependencies

`PyTorch` · `torchvision` · `CUDA 12.8` · `Transformers` · `Safetensors` · `Hugging Face Hub` · `ONNX` · `ONNX Runtime` · `NumPy / NPZ`

The independent vision environment also declares `einops`, `OmegaConf`, `imageio`, `trimesh`, `MoviePy`, `addict`, `e3nn`, `SciPy`, `Matplotlib` and `pycolmap` as supporting model dependencies. The main environment declares `click`, `platformdirs` and `tqdm` for the optional vision path. These are environment dependencies rather than separate product features.

### Implementation status

| 技术 | 当前项目状态 |
| --- | --- |
| Blender / FFmpeg / scene contracts / QC | 核心代码已实现，项目文档记录本机渲染与端到端验证 |
| TransNetV2 / GroundingDINO / SAM 2.1 / DA3 / CenterFace | 可选后端已实现；项目文档记录权重下载、SHA256 校验和视觉环境自检 |
| MapAnything Apache | 可选适配与模型登记已实现，包与权重未在本机安装 |
| Cosmos Transfer 2.5 / Wan2.2 VACE / ComfyUI | Provider 适配已编写，服务与工作流未部署 |
| Docker / NVIDIA Container Toolkit / Mesa llvmpipe | 容器与 GPU / CPU 部署配置已编写，镜像尚未在本机构建验证 |

The registered `SAM 3` adapter requires externally authorized weights; those weights are not installed. `COLMAP` / `MegaSaM` camera-track imports are supported as external NPZ inputs, not presented as local model deployments.

## Quality and Production Controls

- **Pre-render gates:** motion limits, collision / penetration checks, occlusion-aware composition, camera grammar, pacing and setup readability.
- **Post-render checks:** media integrity, editing, shot scale, speed, orientation, construction consistency and variation.
- **Reproducibility:** input-hash idempotency, checkpoint resume, batch thread orchestration and one-job failure isolation.
- **Review integrity:** technical checks are automated; sampled visual review, normal-speed viewing and curation remain explicit human states.
- **Delivery:** render reports, comparison dashboards, cost reports, V2V packages and optional model exports.

## Skills

The private project includes four reusable Agent Skills:

- `whitebox-batch-forward`: structure-tree batch construction;
- `whitebox-story-studio`: story-driven short films and long-take planning;
- `whitebox-reverse-gated`: authorized reference-video analysis and gated reconstruction;
- `generate-whitebox-v2v-package`: Skill v3 planning, image preparation, binding, packaging and submission.

This project demonstrates how generative-media capabilities become a repeatable production tool: structured inputs, explicit contracts, model adapters, fallbacks, QC gates and inspectable delivery artifacts.
