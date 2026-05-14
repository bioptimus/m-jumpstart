# M-Optimus — AWS Marketplace SageMaker Jumpstart

This repository contains a sample notebook for deploying **M-Optimus** from AWS Marketplace using Amazon SageMaker.

M-Optimus is a multimodal foundation model for histology developed by [Bioptimus](https://www.bioptimus.com). It integrates whole slide histology images with bulk RNA-seq data to produce rich multimodal representations. The package also includes a dedicated tissue segmentation model.

## Notebook

**[`M-optimus.ipynb`](M-optimus.ipynb)** — end-to-end walkthrough covering:

1. Subscribing to the model package on AWS Marketplace
2. Real-time inference (embedding mode, prediction mode, tissue segmentation)
3. Batch inference (embedding and prediction transform jobs)
4. Clean-up

## Pre-requisites

- An AWS account with an active subscription to the M-Optimus model package on AWS Marketplace
- An IAM role with **AmazonSageMakerFullAccess** (and Marketplace subscribe permissions if not yet subscribed)
- A SageMaker Notebook Instance, SageMaker Studio, or EC2 instance with Jupyter
- A SageMaker execution role ARN (when running outside SageMaker, `get_execution_role()` will not work — hardcode the role ARN in the session setup cell)

## Input data format

Input data is expected to already be in the JSON format accepted by the API — no additional conversion is required.

### Batch inputs (`data/input/batch/`)

| File | Mode | Description |
|------|------|-------------|
| `m_embed_inputs.jsonl` | `embedding` | One JSON record per line; each record has `model_name: "m-optimus-stx"` and `mode: "embedding"`. No `bulk_rna` required. |
| `m_predict_inputs.jsonl` | `prediction` | One JSON record per line; each record has `model_name: "m-optimus-stx"`, `mode: "prediction"`, and a `bulk_rna` vector (19 374 floats). |

### Real-time inputs (`data/input/real-time/`)

| File | Model | Description |
|------|-------|-------------|
| `m_embed_input.json` | `m-optimus-stx` | Single record with `mode: "embedding"`. No `bulk_rna` required. |
| `m_predict_input.json` | `m-optimus-stx` | Single record with `mode: "prediction"` and a `bulk_rna` vector (19 374 floats). |
| `tissue_seg_input.json` | `tissue-seg` | Single record with a 512×512 tile, `resolution: 8.0`, `mode: "prediction"`. |

### JSON record fields

| Field | Type | Description |
|-------|------|-------------|
| `image_data` | string | Base64-encoded PNG tile |
| `bulk_rna` | list\[float\] \| null | Bulk RNA-seq expression vector (required for prediction mode, `null` for embedding) |
| `slide_name` | string | Identifier for the source slide |
| `x`, `y` | int | Top-left coordinates of the tile in pixels |
| `width`, `height` | int | Tile dimensions in pixels (224×224 for M-Optimus, 512×512 for tissue segmentation) |
| `tissue_ratio` | float | Fraction of the tile covered by tissue |
| `patch_idx` | int | Index of this tile within the slide |
| `resolution` | float | Extraction resolution in µm/px (0.5 for M-Optimus, 8.0 for tissue segmentation) |
| `model_name` | string | Server-side dispatch key (`"m-optimus-stx"` or `"tissue-seg"`) |
| `mode` | string | `"embedding"` or `"prediction"` |

## Expected outputs

| Mode | Output dimension | Description |
|------|-----------------|-------------|
| Embedding | 1 536 | Tile feature vector |
| Prediction | 6 002 | Multimodal prediction output |
| Tissue segmentation | 262 144 (512 × 512) | Flattened binary tissue mask |

## Naming conventions

| SageMaker resource | Name |
|-------------------|------|
| Endpoint | `m-optimus` |
| Batch model | `m-optimus` |
| Embedding transform job | `m-optimus-embed-<timestamp>` |
| Prediction transform job | `m-optimus-predict-<timestamp>` |

## Dependencies

```
sagemaker==2.254.1
boto3==1.42.2
```

Install via the `%pip install` cells at the top of the notebook, or run:

```bash
pip install sagemaker==2.254.1 boto3==1.42.2
```
