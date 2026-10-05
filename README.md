<div align="center">

# Mingyang Yao

### AIGC Application Engineer

AIGC 应用工程师 · Generative Image / Video / Multimodal Workflow Engineering

I turn model APIs and vision tools into reproducible, reviewable media workflows.

<a href="mailto:2017373873@qq.com">Email</a> ·
<a href="https://github.com/Jack51296">GitHub</a>

<br />

<img src="./assets/aigc.png" alt="AIGC" width="82" height="24" />
<img src="./assets/python.png" alt="Python" width="92" height="24" />
<img src="./assets/comfyui.png" alt="ComfyUI" width="102" height="24" />
<img src="./assets/vibe-coding.png" alt="Vibe Coding" width="124" height="24" />

</div>

我关注生成模型接入之后的工程化部分：把图像、视频和多模态任务拆成可校验的数据契约、可恢复的批处理、可解释的视觉一致性处理，以及能交给下游继续使用的交付物。

以下展示两个私有项目的技术与应用案例。源码、内部素材和凭证保持私有；主页只展示可核验的技术边界、实现状态和项目记录。

## What I Build

| 工程问题 | 我的处理方式 |
| --- | --- |
| **模型调用如何进入稳定流程** | 用 `OpenAI-compatible API`、Provider 抽象、`JSONL` 任务、限速、`Retry-After`、断点续跑和预算闸门，把一次性调用变成可复核批处理。 |
| **图像和视频如何保持结构一致** | 用 `OpenCV`、Lab 追色、`ORB`/`BFMatcher`/`RANSAC` 对齐、光流、`GroundingDINO` + `SAM 2.1`，以及 Blender 的相机与运动学约束。 |
| **生成结果如何变成可交付资产** | 用 `Pydantic`/`JSON Schema`/`SQLite` 记录输入、状态和产物；用 QC 闸门、人工审核状态、`CSV`、`MD5/SHA256`、`S3/BlobStore` 和 Labkit 交付。 |

## Engineering Pillars

- **Contract-driven workflows：** `scene.json`、JSON Schema、JSONL 和七列 CSV 让模型输入、图像处理和视频交付都有明确边界。
- **Reliable orchestration：** `Python`、`Typer`、`ThreadPoolExecutor`、Provider 适配、全局限速、重试、幂等和单项失败隔离支撑长时间批处理。
- **Visual consistency：** `OpenCV Lab` 追色、`ORB + RANSAC` 几何配准、`Canny` 线稿匹配、光流分析、Blender 无头渲染共同约束画面结构。
- **Reviewable delivery：** 自动技术检查与人工审片状态分离，最终交付包含报告、审核页、清单和可追溯文件，而不是只留下模型输出。

## Technical Stack

| 方向 | 具体技术 |
| --- | --- |
| **生成与模型接入** | `GPT Image / gpt-image-2` · `images/edits` · `OpenAI-compatible API` · `HTTPX` · `Requests` |
| **视觉模型** | `TransNetV2` · `PySceneDetect` · `GroundingDINO` · `SAM 2.1` · `DA3` · `MapAnything¹` · `CenterFace` · `YuNet` |
| **视觉算法** | `OpenCV` · `ORB` · `BFMatcher` · `RANSAC` · `Canny` · `Lab Color Space` · `Optical Flow` |
| **3D 与视频** | `Blender 5.1` · `bpy` · `Workbench` · `EEVEE` · `FFmpeg / ffprobe` · `H.264` · `FBX / GLB` |
| **推理与工作流** | `PyTorch` · `CUDA 12.8` · `Transformers` · `ONNX Runtime` · `Typer` · `Pydantic 2` · `Agent Skills` · `JSON Schema / JSONL` |
| **数据与交付** | `SQLite / WAL` · `NumPy` · `Pillow` · `OpenPyXL` · `boto3 / S3` · `BlobStore` · `Chrome CDP` · `MD5 / SHA256` |

¹ MapAnything 已实现可选适配；安装与部署状态见项目技术说明。

## Selected Private Projects

### Whitebox Studio

**Procedural whitebox video production and V2V workflow system**

**问题：** 故事、结构树和参考视频需要变成可编辑、可复现、可审片的白模与 V2V 准备包。

- **契约与编译：** 以 `scene.json` 为共享数据源，把路线和机位意图编译为逐帧关键帧，再交给 Blender 建场。
- **视觉与媒体：** 抽帧、切镜、光流、主体检测/分割、深度与相机位姿分析后，使用 `Blender 5.1.2`、`bpy`、`Workbench`/`EEVEE` 和 `FFmpeg` 输出 H.264、FBX、GLB 与镜头表。
- **可靠执行：** Provider 适配、独立 Vision Worker、JSON/NPZ 文件交换、输入哈希幂等、断点续跑、单任务失败隔离和渲染前后 QC 闸门让失败路径可查。
- **证据边界：** 项目记录包含真实 CLI、Blender/FFmpeg 渲染和 mock 模型的端到端验收；Cosmos Transfer 2.5、Wan2.2 VACE/ComfyUI 为已编写但未部署的可选适配。

**技术与案例：** [Whitebox Studio](./WHITEBOX-STUDIO.md)

### Batch Image Generation Pipeline

**API-driven image generation, color matching, alignment and dataset delivery**

**问题：** 图像模型能生成候选结果，但颜色、构图、轮次、审核和下游数据集交付仍需要稳定的工程控制。

- **批处理可靠性：** 素材与提示词编译为 JSONL，调用 GPT Image `images/edits`；`ThreadPoolExecutor`、全局限速、429 `Retry-After`、超时重试和已有产物跳过支持中断后继续。
- **视觉一致性：** OpenCV Lab 生成 plain/`_colormatched` 双版本；`ORB + BFMatcher + RANSAC` 完成相似变换与必要裁切，结果明确分为 `ok / crop / unaligned`。
- **风格感知：** 像素风使用 `INTER_NEAREST`，线稿使用 `INTER_CUBIC` + Canny 候选，一般风格使用 `INTER_LINEAR`，每对结果记录实际规则和特征域。
- **交付与证据：** 七列 CSV、review page、MD5 manifest、`boto3/S3-compatible BlobStore` 和 Labkit 准备把生成结果变成可检查的输入/输出图像对；项目记录中的 `style0914` 批次包含 61 张源图和 122 个 processed 文件。

**技术与案例：** [Batch Image Generation Pipeline](./BATCH-IMAGE-PIPELINE.md)

## Full Stack & Implementation Notes

<details>
<summary><strong>完整技术栈、模型适配与工程机制</strong></summary>

### Generative AI and providers

`GPT Image` · `Images Edits API` · `OpenAI-compatible APIs` · `Mock Providers` · `Manual / HTTP Generic Providers` · `Cosmos Transfer 2.5 adapter` · `Wan2.2 VACE adapter`

### Computer vision and geometry

`OpenCV` · `ORB` · `BFMatcher` · `RANSAC` · `Canny` · `Lab Color Space` · `Optical Flow` · `Similarity Transform` · `TransNetV2` · `PySceneDetect` · `CenterFace` · `YuNet` · `ONNX Runtime` · `GroundingDINO` · `SAM 2.1` · `DA3` · `MapAnything`

### Model runtime

`PyTorch` · `torchvision` · `CUDA 12.8` · `Transformers` · `Safetensors` · `Hugging Face Hub` · `ONNX` · `ONNX Runtime` · isolated `.venv-vision` · JSON / NPZ file exchange

### 3D, video and media

`Blender 5.1` · `Blender Python API / bpy` · `Workbench` · `EEVEE` · `FFmpeg` · `ffprobe` · `H.264` · `FBX` · `GLB` · `Headless Rendering`

### Python and data contracts

`Python 3.11+` · `Typer` · `Pydantic 2` · `NumPy` · `Pillow` · `HTTPX` · `Requests` · `Jinja2` · `PyYAML` · `OpenPyXL` · `JSONL` · `CSV` · `JSON Schema` · `SQLite`

### Agent and workflow engineering

`Agent Skills` · `Prompt Templates` · `Provider Abstraction` · `Vision Worker` · `File-based IPC` · `ThreadPoolExecutor` · `Checkpoint Resume` · `Idempotent Jobs` · `Retry Policies` · `Rate Limiting` · `Budget Gates`

### Infrastructure and delivery

`Dockerfile` · `NVIDIA Container Toolkit` · `Mesa llvmpipe` · `GPU / CPU Fallback` · `pytest` · `ruff` · local CI scripts · `boto3` · `S3-compatible BlobStore` · `KFS / Ceph` · `Labkit` · `MD5 / SHA256` · `websocket-client` · `Chrome CDP` · `code-server` · `tqdm`

**实现状态：** TransNetV2、PySceneDetect、GroundingDINO、SAM 2.1、DA3、CenterFace 等后端均有实现与回退路径；MapAnything 适配已实现，包与权重未在本机安装。Cosmos Transfer 2.5、Wan2.2 VACE / ComfyUI 是已编写的可选 Provider，服务未部署。Dockerfile 与 GPU / CPU 部署配置已编写，镜像尚未在本机构建验证。模型调用默认 mock，视频提交默认导出提交包。

</details>

## Engineering Principles

- **先约束，再执行：** 用 `scene.json`、JSON Schema、JSONL 和 CSV 契约校验昂贵操作的输入。
- **可复现运行：** 记录输入哈希、随机种子、检查点、模型版本和产物清单。
- **质量进入流程：** 自动检查报告技术状态，画面审美与最终采用保留独立人工判断。

> One more batch should still be reproducible.

<div align="center">

`Python` · `OpenCV` · `Blender` · `FFmpeg` · `GPT Image` · `SAM 2.1` · `GroundingDINO` · `DA3` · `MapAnything` · `Agent Skills`

</div>
