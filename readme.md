# Outlier Detection

A comparative study of five outlier-detection algorithms evaluated on eight benchmark datasets (four multivariate ODDS datasets and four 2D clustering datasets with noise).

---

## Algorithms

The project benchmarks five outlier detection methods:

- **One-Class SVM** — a kernel-based method that learns a tight decision boundary around normal training data in a high-dimensional feature space, flagging points that fall outside it as anomalies.
- **Isolation Forest** — an ensemble of random trees that isolates anomalies by recursively partitioning the feature space, outliers require fewer splits to isolate and receive higher anomaly scores.
- **Local Outlier Factor** — a density-based method that compares each point's local neighborhood density to that of its neighbors, points in significantly sparser regions receive higher outlier scores.
- **DBSCAN** — a clustering algorithm that groups points reachable within a radius eps and minimum neighbor count, points unreachable by any dense region are labeled as noise and treated as outliers.
- **Deadwood** — a tree-based ensemble method included as the novel algorithm under evaluation, using forest structure to assign outlier labels directly without requiring a contamination estimate.

---

## Datasets

### ODDS

Four datasets from the [Outlier Detection DataSets (ODDS)](https://shebuti.com/outlier-detection-datasets-odds/) repository, chosen for their diversity in size, dimensionality, and outlier ratio.

| Name | Points | Dimensions | Outliers |
|---|---|---|---|
| Cardio | 1 831 | 21 | 176 (9.6 %) |
| Ionosphere | 351 | 33 | 126 (36.0 %) |
| Speech | 3 686 | 400 | 61 (1.65 %) |
| Vowels | 1 456 | 12 | 50 (3.4 %) |

---

### 2D Clustering

Four 2D datasets drawn from the [SIPU and Graves benchmark suites](https://clustering-benchmarks.gagolewski.com/weave/suite-v1.html). Their low dimensionality makes them well suited for visual inspection of each algorithm's decision boundaries and failure modes.

| Name | Points | Outliers |
|---|---|---|
| Aggregation | 788 | 0 (0.0 %) |
| Flame | 240 | 12 (5.0 %) |
| Ring | 1 030 | 30 (2.91 %) |
| Zigzag | 280 | 30 (10.71 %) |

---

## Description

The notebook is organised into three sections:

1. **Data Loading & Processing** — loaders for `.mat` and compressed 2D files, with per-dataset summary statistics (point count, dimensionality, outlier rate).
2. **Algorithms** — self-contained wrappers for each method that return both anomaly scores (for AUC evaluation) and binary predictions (for threshold-based metrics).
3. **Experiments** — hyperparameter sensitivity sweeps plotting AUC and F1 across parameter grids, followed by best-configuration evaluation tables for each algorithm and dataset.

---

## Metrics

Each algorithm is evaluated across five metrics to capture both ranking quality and classification performance:

- **AUC** — area under the ROC curve; measures ranking quality independent of the decision threshold.
- **Accuracy** — share of correctly classified points; can be misleading on imbalanced datasets.
- **Precision** — fraction of flagged points that are true outliers.
- **Recall** — fraction of true outliers that were successfully flagged.
- **F1** — harmonic mean of precision and recall; the primary metric for imbalanced evaluation.

---

## Sensitivity & Robustness Analysis

To assess how each algorithm responds to its key hyperparameters, sensitivity sweeps were conducted across all four ODDS datasets, tracking AUC and F1 as individual parameters were varied while others were held at their defaults.

**One-Class SVM** proved the most sensitive overall. The `nu` parameter, which controls the expected fraction of outliers, had a strong effect on both precision and recall. Small values caused the model to flag too few points, while large values over-flagged inliers. The choice of `gamma` introduced additional instability, particularly on high-dimensional datasets like Speech, where the default `scale` heuristic did not always produce well-separated decision boundaries.

**Isolation Forest** was the most robust algorithm in the study. AUC remained largely stable across a wide range of `n_estimators` values, and performance converged quickly, suggesting that even modest ensemble sizes are sufficient for most datasets. The `contamination` parameter had a more noticeable effect on F1 than on AUC, as it directly controls the decision threshold rather than the underlying anomaly scores.

**Local Outlier Factor** showed moderate sensitivity to `n_neighbors`. Too small a neighborhood caused noisy, unstable density estimates, while too large a value over-smoothed local structure and reduced recall on datasets with tight outlier clusters. The optimal range (10–20 neighbors) was fairly consistent across datasets, making LOF relatively predictable to tune despite its local nature.

**DBSCAN** exhibited the highest sensitivity of all methods. The `eps` parameter was particularly difficult to set: values even slightly below the optimal produced dramatically fragmented clusters and a large number of false positives, while slightly larger values caused genuine outliers to be absorbed into clusters. Sensitivity to `min_samples` was comparatively mild, though it interacted with `eps` in ways that made joint tuning non-trivial. Performance variance across datasets was the highest of any method tested.

**Deadwood** was the most stable algorithm across all sweeps. Varying `M` (the number of trees) produced smooth, monotonically improving AUC up to around M=5, after which returns diminished. The absence of a contamination parameter removed an entire axis of sensitivity that affected the other methods, contributing to its consistently low variance across datasets and outlier ratios.

A key robustness finding emerged from the 2D experiments: algorithms that performed well on ODDS datasets did not always transfer cleanly to non-convex cluster geometries. DBSCAN and LOF adapted well to ring and zigzag structures, while Isolation Forest and One-Class SVM, both of which implicitly assume relatively compact normal regions, struggled with datasets where inliers formed irregular shapes. This geometry sensitivity is worth considering when selecting a method for real-world data where the structure of normal behaviour is unknown in advance.

---

## Conclusion

The results highlight that no single algorithm dominates across all settings. 

**Isolation Forest** proved the most consistent performer on high-dimensional ODDS datasets, benefiting from its ensemble nature and robustness to the curse of dimensionality. 

**Local Outlier Factor** excelled on datasets with clearly separated density regions but degraded on high-dimensional data where distance metrics become unreliable. 

**One-Class SVM** was competitive on lower-dimensional datasets but showed sensitivity to kernel and `nu` hyperparameter choices, making careful tuning essential. 

**DBSCAN**, while not designed specifically for outlier detection, performed well on the 2D datasets where cluster structure was visually apparent, though its reliance on `eps` and `min_samples` made it brittle across varying dataset geometries.

**Deadwood** demonstrated competitive performance without requiring a contamination estimate upfront, which is a meaningful practical advantage since the true outlier rate is rarely known in real-world deployments. Its performance was particularly stable across varying outlier ratios, suggesting it may generalise better to unseen data distributions.

The 2D datasets also revealed an important qualitative insight: visual inspection of decision boundaries exposed failure modes that aggregate metrics alone would obscure. Algorithms achieving similar AUC scores can behave very differently in terms of which regions of the feature space they flag, underscoring the value of visualisation alongside quantitative evaluation.

Overall, the findings suggest that ensemble and density-based methods remain strong baselines for outlier detection, while Deadwood presents a promising direction for scenarios where contamination rates are unknown. Future work could extend this comparison to semi-supervised settings, explore algorithm ensembling strategies, and evaluate robustness under concept drift.

---