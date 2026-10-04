<div align="center">

# Mingyang Yao

### AIGC Application Engineer

AIGC 应用工程师

I build production workflows for generative image, video, and multimodal content.

<a href="mailto:2017373873@qq.com">Email</a> ·
<a href="https://github.com/Jack51296">GitHub</a>

<br />

<img src="./assets/python.png" alt="Python 3.11+" width="120" height="24" />
<img src="./assets/opencv.png" alt="OpenCV 4.x" width="110" height="24" />
<img src="./assets/blender.png" alt="Blender 5.1" width="112" height="24" />
<img src="./assets/ffmpeg.png" alt="FFmpeg" width="88" height="24" />
<img src="./assets/pytorch.png" alt="PyTorch CUDA" width="128" height="24" />
<img src="./assets/pydantic.png" alt="Pydantic 2.x" width="118" height="24" />

</div>

我将生成式模型接入图像与视频生产流程：从模型 API、Python 自动化、视觉处理与 3D 渲染，到 Agent Skills、质量检查和数据交付。

以下展示两个项目的技术与应用案例，核心实现保持私有。

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

**落地场景：** 故事与长镜头预演、人物 / 车辆运动及机位验证、参考视频镜头反推、白模转真人 V2V 制作准备。

- **正向构建：** 结构树 / 故事需求 → `scene.json` → 路线与机位编译 → Blender 无头渲染 → H.264、FBX、GLB 与镜头表。
- **视频反推：** 抽帧、切镜、光流、主体占幅和结构线分析；GroundingDINO + SAM 2.1 提取主体，DA3 等后端提供深度与相机位姿。
- **V2V 编排：** 规划 → 取帧 / 生图 → 引用绑定 → 提交包；使用 Provider 适配与独立 Vision Worker，失败时按回退链处理。
- **生产控制：** 渲染前后质量闸门、SQLite 输入哈希幂等、断点续跑、单任务失败隔离、成本记录和独立人工审核状态。

**技术与案例：** [Whitebox Studio](./WHITEBOX-STUDIO.md)

### Batch Image Generation Pipeline

**API-driven image generation, color matching, alignment and dataset delivery**

**落地场景：** 素材集光影改写、二次元平涂转像素、白描 / 线稿等风格转换，以及对齐后的输入输出图像对交付。

- **批量生成：** 素材 → JSONL → GPT Image `images/edits` → 直出 / 追色双版本；`ThreadPoolExecutor` 并发、全局限速、429 `Retry-After` 和失败重试。
- **追色与对齐：** OpenCV Lab 色度统计匹配；ORB + BFMatcher + RANSAC 估计相似变换；线稿使用灰度 / Canny 双路匹配。
- **风格规则：** 像素风最近邻、线稿和细线平涂双三次、普通风格双线性；逐图记录处理规则与 `ok / crop / unaligned` 状态。
- **数据交付：** 七列 CSV、人工审核页、跨批次 MD5 去重、完整性清单、boto3 / S3-compatible BlobStore 与 Labkit 数据集准备。

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
