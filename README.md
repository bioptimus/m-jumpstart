# M-Optimus — Spatial Transcriptomics from Histology

**Predict spatial gene expression directly from H&E slides — optionally refined with bulk RNA-seq.**

[M-Optimus](https://docs.bioptimus.com/documentation/models/m-optimus) is a multimodal foundation model for histology developed by [Bioptimus](https://www.bioptimus.com). Its headline capability is predicting **spatial gene expression** directly from a routine H&E tile, recovering an expensive molecular readout from a low-cost slide. The package also includes a dedicated tissue segmentation model.

| Capability | Output |
|---|---|
| **Tile embeddings** | 1536-dimensional feature vector (same as [H-Optimus-1](https://docs.bioptimus.com/documentation/models/h-optimus)) |
| **Spatial gene expression** | Predicted expression for 6,002 genes per tile (Ensembl IDs) |
| **Multimodal refinement** | Optional bulk RNA input improves predictions ~4% |

## Getting started

The tutorial notebook **[`M-Optimus.ipynb`](M-Optimus.ipynb)** is an end-to-end, runnable walkthrough: it downloads a demo TCGA-LUAD slide, generates tissue masks, extracts embeddings, predicts spatial gene expression (image-only and refined with bulk RNA), and visualizes the results.

1. Install the SDK: `pip install "bioptimus-sdk[torch]"` (the `[torch]` extra is only needed for the `local` backend).
2. Open `M-Optimus.ipynb`.
3. In **Section 2 (Configuration)**, choose your deployment backend (see below).
4. Run the cells top to bottom.

## Choosing a deployment option

The notebook runs the same pipeline against any of three backends, selected via the `Backend` enum. Pick **one** and configure it in both the **Configuration** cell (Section 2) and the **`Inference(...)`** call (Section 4) — only one backend block may be active at a time.

| Backend | When to use | Key parameters |
|---|---|---|
| **`remote`** (`Backend.REMOTE`) | You have a running Bioptimus FastAPI server (Docker or bare-metal) | `api_url` |
| **`aws`** (`Backend.AWS`) | You have a deployed SageMaker endpoint | `endpoint_name`, `region_name` |
| **`local`** (`Backend.LOCAL`) | You have a CUDA GPU plus the `.pt2` checkpoints and assets CSV (no server) | `checkpoints`, `device`, `assets_root` |

> **Not sure which to pick?** Given a **server URL** → `remote`. Deployed on **AWS SageMaker** → `aws`. Have a **GPU machine plus the model files** → `local` (the notebook's default).

### Backend-specific setup

- **`remote`** — Set `API_URL` (e.g. `http://0.0.0.0:8080`); `utils.check_server(API_URL)` verifies connectivity.
- **`aws`** — Set `ENDPOINT_NAME` and `REGION_NAME`. Requires an AWS account with an active Marketplace subscription to M-Optimus, an IAM role with **AmazonSageMakerFullAccess**, and a SageMaker execution role ARN. Tissue segmentation is bundled on the same endpoint at no extra cost.
- **`local`** — Requires a CUDA GPU compatible with the exported `.pt2` models (default `sm_86` — A10G, RTX 3090) and three paths, all provided by Bioptimus:
  - `M_OPTIMUS_CHECKPOINT` — the M-Optimus `.pt2` checkpoint
  - `TISSUE_SEG_CHECKPOINT` — the tissue-segmentation `.pt2` checkpoint
  - `ASSETS_ROOT` — the M-Optimus **assets CSV** listing the input/output genes the model expects (used to align gene columns)

  The `checkpoints` dict keys (`"m-optimus"`, `"tissue-seg"`) are fixed identifiers — do not rename them.

## Prerequisites

| Requirement | Details |
|---|---|
| **Python** | 3.12+ |
| **SDK** | `bioptimus-sdk` (`[torch]` extra for the `local` backend) |
| **Backend** | One of `remote`, `aws`, or `local` (see above) |
| **Disk space** | ~2 GB slide + ~500 MB outputs for the demo |

The notebook downloads its demo data automatically. To use your own slides, drop whole-slide images (`.svs`, `.tiff`, `.ndpi`, and others) into the WSI directory; bulk RNA-seq is optional and late-bound from a TSV/CSV of **TPM-normalized** values keyed by Ensembl gene ID.

## Documentation

| Resource | URL |
|---|---|
| M-Optimus model page | https://docs.bioptimus.com/documentation/models/m-optimus |
| Spatial transcriptomics guide | https://docs.bioptimus.com/guides/workflows/spatial-transcriptomics |
| SDK overview | https://docs.bioptimus.com/guides/get-started/sdk |
| Inference facade | https://docs.bioptimus.com/guides/get-started/inference-facade |
| Cohorts guide | https://docs.bioptimus.com/guides/workflows/cohort |
| Visualizing results | https://docs.bioptimus.com/guides/get-started/visualizing-results |
| Choosing a model | https://docs.bioptimus.com/documentation/models/choosing-a-model |
