# Glioma IDH Classifier

An end-to-end baseline for predicting glioma IDH mutation status from bulk RNA-seq gene expression. The repository provides validated data loading, leakage-safe feature selection, stratified evaluation, model persistence, ranked genes, a synthetic demonstration, and automated tests.

> Research and education only. This is not a diagnostic tool and must not be used for patient care.

## Quick start

```bash
git clone https://github.com/palrabheru/glioma-idh-classifier.git
cd glioma-idh-classifier
python -m venv .venv
# Windows: .venv\Scripts\activate
# macOS/Linux: source .venv/bin/activate
python -m pip install -r requirements.txt
python train.py --demo --output-dir outputs/demo
pytest -q
```

## Train with real data

The expression CSV/TSV needs one row per sample, a unique sample ID, and numeric gene columns. The clinical CSV/TSV needs matching IDs and a binary IDH-status column.

```bash
python train.py --expression data/expression.tsv.gz \
  --clinical data/clinical.tsv --sample-column sample_id \
  --target-column idh_status --positive-label Mutant \
  --output-dir outputs/tcga_run
```

Files may be gzip compressed. If genes are rows, transpose the expression matrix first. Default mutant labels include `1`, `true`, `yes`, `mutant`, `mutated`, and `idh-mutant`; otherwise set `--positive-label`.

## Modeling design

After a stratified train/test split, the pipeline fits median imputation, variance filtering, ANOVA feature selection, standardization, and class-balanced logistic regression on the training set only. This avoids the common error of choosing genes using the held-out samples.

| Output | Description |
|---|---|
| `model.joblib` | Complete fitted pipeline |
| `metrics.json` | Held-out performance and run details |
| `predictions.csv` | Sample-level labels and probabilities |
| `feature_importance.csv` | Selected genes ranked by coefficient magnitude |
| `confusion_matrix.png` | Held-out confusion matrix |

```text
train.py              Command-line entry point
src/idh_model.py      Data validation, model, and output logic
tests/                Automated tests
data/                 Local data (not committed)
```

The random seed defaults to `42`. Strong performance can reflect known biological separation, but coefficients alone do not establish causal biomarkers. Before reporting a diagnostic result, use patient-level splitting, external validation, appropriate batch handling, uncertainty estimates, comparisons with clinical covariates, and prospective validation. Carefully harmonize TCGA labels and sample identifiers, and avoid mixing aliquots from the same patient across splits.

## License

Add a license before reuse or redistribution.
