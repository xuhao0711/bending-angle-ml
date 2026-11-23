[简体中文](README.md) | [English](README.en.md)

# Bending Angle Classification: Training and Plotting in One Notebook

`bend_angle_ml.ipynb` includes 21 classification models, Optuna hyperparameter optimization, nested cross-validation, a full set of evaluation metrics, figures based on the original plotting templates, and SHAP explanations for all models. Both the code and input data are embedded in the notebook, so it does not depend on the original project directory or other local Python scripts.

## Getting started

1. Set up Python 3.11 and install the dependencies:

   ```bash
   python -m pip install -r requirements.txt
   ```

2. Open `bend_angle_ml.ipynb` in Jupyter or VS Code and select the Python kernel with the required dependencies installed.
3. Make sure Arial is installed. Alternatively, set `FONT_DIRECTORY` in the first configuration cell to your own font directory.
4. For the first run, change `PROFILE` from `"full"` to `"smoke"`, restart the kernel, and run all cells from the beginning.
5. For a full training run, change `PROFILE` back to `"full"` and run the notebook from the beginning.

If you download only the notebook, you can first run the data preparation cell to generate `bend_angle_reproduction/requirements.txt`, then follow the instructions to install the dependencies.

## Configuration

| Profile | Final search trials per model | Search trials within each outer training subset | Purpose |
|---|---:|---:|---|
| `smoke` | 5 | 1 | Check the code, data, and plotting workflow with smaller models |
| `quick` | 15 | 3 | Explore the workflow with a limited search budget |
| `full` | 100 | 20 | Use the full training search budget |

The random seed is 42. The full configuration requests up to five cross-validation folds. The actual number of folds is adjusted to the available samples or events per class and recorded in the run logs. With the embedded dataset, cycle-grouped evaluation typically uses three outer folds, two nested inner folds, and three inner folds for the final search. The requested fold count should not be reported as the actual count without checking the run logs.

`RUN_MODE="train_and_plot"` runs training and generates figures. If outputs from this notebook already exist with the same configuration, set it to `"saved_results"` to regenerate the figures. This mode does not load old models from the original author's computer.

`RESUME=True` resumes existing Optuna searches. To start a separate run, choose a new `OUTPUT_DIRECTORY`. The sampling sequence after resuming an interrupted search is not guaranteed to match an uninterrupted run trial by trial.

## Data and validation limits

- The notebook embeds 196 windows and an NPZ archive containing sampling times, source row indices, and event identifiers. These data come from 28 bending-and-return events across five target angle classes.
- The full 16-feature input contains 13 raw response points, the window peak, the time-weighted mean, and the time-weighted root mean square (RMS). The notebook independently checks the calculation of all three summary statistics.
- The seven windows from the same event overlap extensively and must not be treated as seven independent experimental samples.
- Cycle-grouped evaluation assigns complete events to a fixed held-out test set. A random-window split is included for comparison, with shared events and shared original sampling points recorded.
- Median imputation, scaling, and probability calibration are fitted only within the relevant training partitions. Models are selected using outer cross-validation macro-F1 on the training data, rather than ranked by held-out test performance.
- The current data do not provide validation across devices, collection dates, or material batches. SHAP attributions do not establish causal evidence for material mechanisms.

## Figures

The original plotting templates and their source hashes are preserved. Data adaptations and explicit code replacements are recorded during execution. The figures include:

- Percentage confusion matrices, classification ROC curves, and ROC curves for the outer cross-validation folds for each model.
- Optuna optimization histories, hyperparameter importance plots, and 2D and 3D search plots.
- Radar charts and polar plots of classification metrics.
- Combined SHAP plots for all 21 models, with class-specific interaction networks for models that support tree-based interaction explanations.
- Confusion matrix panels for three model families, confusion matrix and ROC panels for nine models, and SHAP panels for the top three models.

Confusion matrices are normalized by row using the true class. Each row sums to 100% before rounding, and values are displayed to one decimal place. Diagonal entries show the recall for each class. The original colors, class order, and layout are preserved.

All figures are exported as SVG and PDF using Arial, with editable text. The notebook adds text category tags to the SVG files and writes `按类别统一调整文字.jsx`, a helper script for manually selecting or modifying text in the same category in Adobe Illustrator. The notebook itself does not launch Illustrator or use computer-use automation.

The Taylor plot template is retained as an optional supplementary diagnostic for angle errors, with `ENABLE_TAYLOR=False` by default. It is not used to rank classification models.

If a plot cannot be generated, for example because the objective values do not vary and hyperparameter importance cannot be estimated, the missing plot is recorded without inventing values. The SHAP output scale depends on the explainer; see the notes for each model. Absolute SHAP values on different output scales cannot be compared directly.

## Outputs and repository contents

Outputs are saved in `bend_angle_reproduction/` under the current working directory. The main outputs are:

- `model_comparison.csv` and complete per-class evaluation metrics.
- Hyperparameters, training logs, saved models, and predictions from the fixed evaluation split for each model.
- Individual figures in `figures/original_templates/`.
- Combined figure panels in `figures/paper_groups/`.
- Explanation tensors, background sample indices, and additivity checks in `all_models_shap/`.

This repository contains a clean source notebook with previous execution outputs cleared. Font files, saved models, and local caches are excluded from the upload package, and `.gitignore` excludes generated outputs by default. Original template filenames and hashes are listed in `sources_manifest.json`.

## Validation record

The scope of the run executed during final packaging is recorded in `validation_status.json`. Passing the `smoke` profile verifies that the integrated workflow can execute end to end. It does not replace the full 100-trial searches required for the paper's reported results.
