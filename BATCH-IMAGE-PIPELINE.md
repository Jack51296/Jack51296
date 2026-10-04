# Batch Image Generation Pipeline

**Private case study · API-driven image generation, color matching, alignment and dataset delivery**

[返回个人主页](./README.md)

**项目场景：** 对已有素材批量进行光影或风格改写，保留来源与轮次，在追色、几何对齐和人工审核后输出可送标的输入 / 输出图像对。核心实现保持私有。

This Python pipeline turns source image collections into repeatable generation jobs and delivery-ready image pairs. It covers lighting edits, style conversion, color matching, geometric alignment, review and dataset handoff.

## Production Scenarios

- Lighting and relighting transformations across an image collection.
- Style conversion such as flat illustration → pixel art, line art, ink or other prompt-defined styles.
- Multi-round generation where each source image keeps a stable batch identity and run number.
- Preparing aligned input/output pairs for review, annotation and downstream dataset import.

## End-to-end Flow

```text
source assets
  → survey / normalize / prompt-table lookup
  → validated JSONL jobs
  → GPT Image images/edits
  → plain + _colormatched outputs
  → seven-column delivery CSV
  → style-aware geometric alignment
  → review page / MD5 manifest / BlobStore / Labkit handoff
```

## Concrete Technology

### API orchestration

`Python` · `Requests` · `GPT Image / gpt-image-2` · `Images Edits API` · OpenAI-compatible gateway / KLink backend · `JSONL` · `CSV` · `ThreadPoolExecutor` · `tqdm`

The generator uses concurrent workers with a global rate limit. It handles request timeouts, HTTP 429 `Retry-After`, bounded retries, existing-output skipping and checkpoint-style continuation. Per-item statuses distinguish successful, skipped, blocked and failed jobs so one bad request does not invalidate a batch.

### Color matching

`OpenCV` · `NumPy` · `Pillow` · `Lab Color Space`

When enabled, each generated image produces a direct version and a `_colormatched` version. Color statistics are adjusted in Lab space, and the delivery rules choose the appropriate output for each style or tag. Styles with their own palette, such as pixel art or ink, can disable color matching at batch level.

### Geometric alignment

`OpenCV ORB` · `BFMatcher` · Hamming distance · `estimateAffinePartial2D` · `RANSAC` · `Canny`

The alignment Skill estimates scale, rotation and translation between source and generated image, crops when required, and classifies each pair as `ok`, `crop` or `unaligned`. Each pair records the selected interpolation and feature domain for later audit.

Style rules are explicit:

| Target style | Upscale rule | Feature domain |
| --- | --- | --- |
| Pixel art | `INTER_NEAREST` | grayscale |
| Line art / white drawing | `INTER_CUBIC` | grayscale and Canny edge candidates |
| Fine-line flat illustration | `INTER_CUBIC` | grayscale |
| General lighting / painterly style | `INTER_LINEAR` | grayscale |

The result is a stable input/output pair in the original image space, with unreliable matches routed to review instead of being silently accepted.

### Delivery and local tools

`OpenPyXL` · `boto3` · S3-compatible `BlobStore` · `MD5` manifests · `websocket-client` · Chrome CDP · `code-server` automation · KFS / Ceph workspace

The pipeline builds and verifies delivery CSVs, review pages and transfer manifests, then uploads aligned outputs to S3-compatible storage or prepares them for Labkit dataset import. MD5 checks make cross-machine transfers auditable.

The local tools also perform cross-batch MD5 duplicate checks and build skip lists before JSONL execution. The project keeps A/B quality and style-rule regression runbooks so processing decisions can be compared against saved reports and review artifacts.

## Reusable Agent Skills

- `lighting-batch-jsonl`: survey assets, convert to PNG, build prompt-driven JSONL and validate rows;
- `lighting-delivery-csv`: build, edit and verify the seven-column delivery contract;
- `lighting-destyle-align`: run ORB / similarity alignment, style-aware interpolation and review export;
- `lighting-delivery-pipeline`: connect generation, delivery and alignment into an operational runbook.

## Engineering Characteristics

- **Batch-safe:** source images, prompts, model settings, run numbers and output paths are recorded per task.
- **Recoverable:** retries and existing-output checks allow interrupted batches to continue without regenerating successful outputs.
- **Style-aware:** interpolation, color matching and feature domains are selected from explicit rules instead of one global image-processing setting.
- **Delivery-oriented:** generated media is accompanied by CSV contracts, reports, review artifacts and integrity manifests.

The project is a production workflow around an image model API: the model generates candidates, while Python, OpenCV, Skills and delivery contracts make the result reviewable and usable downstream.
