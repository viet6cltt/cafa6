# CAFA-6 Protein Function Prediction

This repository contains the complete pipeline used for the CAFA-6 Protein Function Prediction challenge.
The proposed approach leverages precomputed ESM-2 (650M) protein embeddings, wide MLP classifiers, and an IA-aware ensemble strategy, followed by CAFA-compliant post-processing.

## 1. Repository Structure
```text
├── preprocessing/
│   ├── embedding-cafa6.ipynb
│   ├── go-selected-cafa6.ipynb
│   ├── split_data.ipynb
│   └── taxon.ipynb
│
├── training/
│   ├── training_c95.ipynb
│   └── training_c99.ipynb
│
├── submission/
│   ├── predict.ipynb
│   └── submit.ipynb
│
├── models/
│   ├── c95/
│   └── c99/
│
├── data/
│   ├── raw/
│   └── processed/
│
└── README.md
```

## 2. Environment Setup
Requirements

Python ≥ 3.9

PyTorch ≥ 2.0

CUDA-enabled GPU (recommended for training)

Install dependencies
`pip install torch numpy pandas scikit-learn tqdm obonet`

## 3. Preprocessing Pipeline

All preprocessing steps are executed once and reused across experiments.

### 3.1 Protein Embedding Preparation

File: `preprocessing/embedding-cafa6.ipynb`

Purpose:

Load precomputed ESM-2 650M embeddings

Avoid redundant and expensive embedding extraction

Important Note:

This pipeline does NOT retrain or re-run ESM-2.
Precomputed embeddings are directly loaded to significantly reduce runtime.

Input:

Precomputed ESM-2 embeddings (dimension = 1280)

Output:

`train_embeds.npy`

`test_embeds.npy`

How to run:

`jupyter notebook preprocessing/embedding-cafa6.ipynb`

### 3.2 Gene Ontology Label Selection (C95 & C99)

File: `preprocessing/go-selected-cafa6.ipynb`

Purpose:

Load GO hierarchy (go-basic.obo)

Propagate GO annotations to parent terms

Construct label vocabularies under different coverage settings

Label Sets:

C95: ~6,413 most frequent GO terms

C99: ~15,000 GO terms (covering most annotated labels)

Output:

Label vocabularies for C95 and C99

Multi-label targets per protein

How to run:

`jupyter notebook preprocessing/go-selected-cafa6.ipynb`

### 3.3 Train / Validation Split

File: `preprocessing/split_data.ipynb`

Purpose:

Split proteins into training and validation sets

Prevent sequence similarity leakage

Ensure consistent splits across C95 and C99

Method:

Group-based splitting using cluster information

Output:

Train / validation indices shared by both models

How to run:

`jupyter notebook preprocessing/split_data.ipynb`

### 3.4 Taxonomy Processing

File: `preprocessing/taxon.ipynb`

Purpose:

Process protein taxonomy metadata

Reduce noise from rare taxa

Key Steps:

Map taxon IDs to species level

Keep the top 134 most frequent taxa

Map all remaining taxa to a default category

Output:

Taxonomy index per protein

Used to construct a 64-dimensional taxon embedding

How to run:

`jupyter notebook preprocessing/taxon.ipynb`

## 4. Model Training

Two independent models are trained to handle different GO label distributions.

### 4.1 Training on C95 Labels

File: `training/training_c95.ipynb`

Purpose:

Train a stable classifier for frequent GO terms

Characteristics:

Lower false positive rate

Strong performance on common biological functions

Output:

Best model checkpoint saved to models/c95/

How to run:

`jupyter notebook training/training_c95.ipynb`

### 4.2 Training on C99 Labels

File: `training/training_c99.ipynb`

Purpose:

Train a classifier covering rare and high-IA GO terms

Characteristics:

Higher recall on rare labels

Requires stricter post-processing

Output:

Best model checkpoint saved to models/c99/

How to run:

`jupyter notebook training/training_c99.ipynb`

Model Architecture (C95 & C99)

Input:

Protein embedding: 1280

Taxonomy embedding: 64

Network:

MLP: 4096 → 4096

GELU activation

Dropout + LayerNorm

Training:

Loss: Asymmetric Loss

Optimizer: AdamW

Scheduler: OneCycleLR

Model selection metric: IA-weighted F-max

## 5. Inference and Ensemble
### 5.1 Raw Prediction

File: `submission/predict.ipynb`

Purpose:

Run inference using both trained models

Align C95 predictions into the C99 label space

Output:

Raw probability scores for each protein–GO pair

How to run:

`jupyter notebook submission/predict.ipynb`

### 5.2 IA-aware Ensemble

Implemented in: `submission/predict.ipynb`

Strategy:

Low IA: prioritize C95 predictions

Medium IA: conditional combination

High IA: trust C99 predictions only when confident

This strategy reduces noisy predictions on rare labels while preserving recall.

## 6. Post-processing and Submission
### 6.1 Post-processing

Steps:

No manual GO propagation (CAFA auto-repair enabled)

Probability threshold: 0.2

Keep top 250 GO terms per protein

Clip scores and round to 3 decimal places

### 6.2 Submission File Generation

File: submission/submit.ipynb

Output Format:

Protein_ID    GO_term    score


TSV format

No header

Directly uploadable to CAFA-6

How to run:

`jupyter notebook submission/submit.ipynb`

## 7. Full Execution Order

Run notebooks in the following order:


`preprocessing/embedding-cafa6.ipynb`
`preprocessing/go-selected-cafa6.ipynb`
`preprocessing/split_data.ipynb`
`preprocessing/taxon.ipynb`

`training/training_c95.ipynb`
`training/training_c99.ipynb`
`submission/predict.ipynb`
`submission/submit.ipynb`

**Notes**

Path configuration notice:
The file paths used in the notebooks are provided as examples and must be adjusted to match your local directory structure before execution.
Please update dataset paths, embedding locations, and output directories according to your own environment.