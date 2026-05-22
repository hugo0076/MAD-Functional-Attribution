# Mechanistic Anomaly Detection via Functional Attribution

This repo provides an implementation for our ICML paper: [Mechanistic Anomaly Detection via Functional Attribution](https://arxiv.org/abs/2604.18970).

## Setup

```bash
pip install -r requirements.txt
```

1. Download the BackdoorBench attack result (e.g., CIFAR-10 Blended 5%) and unzip it to a folder. (Available at [this link](https://github.com/SCLBD/backdoorbench))
2. Run `prepare_data.py` pointing to the unzipped folder:

```bash
python prepare_data.py --bbench_dir path/to/unzipped_folder
```

This downloads the CIFAR-10 test set via torchvision, splits it class-balanced into trusted (2500), sampling (2500), and test (5000) subsets, and pairs test images with their backdoor counterparts from BackdoorBench.

After setup, `data/` will contain:

```
data/
├── attack_result.pt    # BackdoorBench PreActResNet18 checkpoint
├── clean_test/         # Clean CIFAR-10 test images (class-folder: 0/, 1/, ..., 9/)
├── bd_test/            # Anomalous test images (same structure)
├── trusted/            # Trusted reference images (known-clean)
└── sampling/           # Images used for SGLD parameter updates
```

## Run

**Step 1: SGLD sampling** (requires GPU)

```bash
python run_sgld.py --data_dir data --output loss_traces.npz
```

This runs RMSprop-SGLD via [devinterp](https://github.com/timaeus-research/devinterp) and saves per-sample loss traces. Default hyperparameters match the paper (Section 5.1):
- Localization: 10000, NBeta: 100, LR: 1e-6
- 2000 draws, 250 burn-in

Override any hyperparameter via CLI args (see `python run_sgld.py --help`).

**Step 2: Analyze**

```bash
python analyze.py --input loss_traces.npz
```

Prints detection metrics:

```
Method           |   AUROC | Mean (Benign)     | Mean (Anomalous)
-----------------------------------------------------------------
Mean Corr        |  0.XXXX |          X.XXXX   |           X.XXXX
CLC              |  0.XXXX |          X.XXXX   |           X.XXXX
Mean CCC         |  0.XXXX |          X.XXXX   |           X.XXXX
Class CCC        |  0.XXXX |          X.XXXX   |           X.XXXX
```

- **Mean Corr**: average Pearson correlation of each test sample's loss trace to all trusted samples.
- **CLC** (Class-Level Correlation): average correlation per trusted class, take the maximum.
- **Mean CCC**: average Concordance Correlation Coefficient to all trusted samples.
- **Class CCC**: mean CCC per trusted class, take the maximum.
