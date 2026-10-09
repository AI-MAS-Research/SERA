# SERA

SERA combines schema-aware semantic representations, admissible historical context, temporal graph evidence, fusion scoring, and risk-constrained alert calibration for transaction fraud detection.

## Repository structure

```text
SERA/
|-- datasets/              data preparation instructions, schemas, and checksums
|-- model/
|   |-- SERA.py            core model components and pipeline
|   `-- __init__.py
|-- training/              data loading, preprocessing, evaluation, and configuration
|   `-- configs/           dataset-specific model settings
|-- main.py                command-line entry point
|-- requirements.txt
|-- requirements-tested.txt
`-- README.md
```

## Installation

Use Python 3.10 and install the dependencies:

```bash
python -m pip install -r requirements.txt
```

The tested dependency versions are listed in [requirements-tested.txt](requirements-tested.txt). Select a PyTorch build compatible with the local CPU or CUDA environment.

## Data preparation

Follow [datasets/README.md](datasets/README.md) to prepare BankSim, Sparkov, IEEE-CIS, and the three IBM-AML datasets. Transaction data is downloaded separately. Expected CSV schemas and reference checksums are included under `datasets/`.

## Running the model

List available datasets and check imports and configuration:

```bash
python main.py --list-datasets
python main.py --dataset banksim --check-only
```

Verify a prepared dataset, then train and evaluate:

```bash
python main.py --dataset banksim --verify-data
python main.py --dataset banksim --seed 2026 --output outputs/sera
```

Use an external data directory:

```bash
python main.py --dataset banksim --datasets-dir /path/to/datasets --output outputs/sera
```

For a smaller functional run:

```bash
python main.py --dataset banksim --window-rows 20000 --output outputs/smoke
```

Changing `--window-rows` changes the evaluation protocol. `--check-only` checks imports and configuration without training; `--verify-data` verifies the CSV header, size, and SHA-256. Run `python main.py --help` for all options.

## Model components

- Schema-aware field encoding represents transaction attributes according to their meaning and availability.
- Historical context aggregates admissible prior observations using learned weights.
- Temporal graph discovery constructs relational features and inspectable patterns.
- Fusion scoring combines base, semantic, historical, and graph features using XGBoost.
- Independent calibration uses held-out negative examples to select an alert threshold under the configured false-positive and violation-probability budgets.

The implementation is in `model/SERA.py`. Import `SERA` with `from model import SERA`; `SERA(config).fit_evaluate(workspace)` runs the pipeline. The command-line entry prepares the workspace automatically. This entry point uses local numerical models; external LLM orchestration is not included.

## Configuration and evaluation

Dataset-specific parameters are stored in `training/configs/datasets.json`. Data is divided chronologically into training, tuning, calibration, and test segments. Preprocessing is fitted on the training segment, and model selection uses the tuning segment.

Reports include precision, recall, F1, AP, AUC-ROC, accuracy, FPR, and calibration diagnostics. Results are saved to `outputs/sera/<dataset>_full_seed<seed>.json` with the run configuration. Cache files remain under the selected output directory. `MAS_DATASET_DIR` and `MAS_OUTPUT_DIR` can override the default data and output locations.
