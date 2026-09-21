# Heart Failure Model Validation

> Improving Validation Design for Mortality Classification Models
>
> 심부전 사망 분류모델의 검증 설계 개선 및 성능 비교

Re-evaluation of mortality classification models on the Heart Failure Clinical
Records dataset after identifying inconsistencies in the original validation
design.

**한국어 요약:** 기존 coursework에서는 모델마다 평가 방식이 달라 성능을
직접 비교하기 어려웠습니다. 특히 RBF SVM의 낮은 오분류율이 training-set
prediction 결과였음을 확인했습니다. 이에 모든 모델을 동일한 repeated
cross-validation 조건에서 재평가하고, 지표별 trade-off를 다시 해석했습니다.

## Highlights

- Identified that the original 2.68% RBF SVM error came from predicting the
  training data.
  - 기존 RBF SVM의 2.68% 오분류율이 학습자료 재예측 결과임을 확인했습니다.
- Re-evaluated five classifiers with identical stratified repeated
  cross-validation folds and fold-internal preprocessing.
  - 다섯 분류모델을 동일한 반복 교차검증과 fold 내부 전처리 조건에서 재평가했습니다.
- Compared ROC-AUC, sensitivity, specificity, balanced accuracy, and accuracy
  instead of relying on one error rate.
  - 단일 오분류율 대신 여러 성능지표를 함께 비교했습니다.
- Preserved the original coursework and documented why the conclusion changed.
  - 원본 coursework를 보존하고 결론이 달라진 이유를 기록했습니다.

## 1. Project Overview

This repository separates two records. The original coursework shows what was
submitted at the time. The validation update documents a later audit of its
model-evaluation design.

The review found that LDA, QDA, SVM, and ANN results had been produced under
different evaluation conditions. The updated analysis keeps the same five model
families and six continuous clinical variables, but evaluates every model on
the same repeated folds. The project is about improving the comparison, not
retrofitting a more favorable result.

## 2. Dataset

The Heart Failure Clinical Records dataset contains 299 patients:

- 96 deaths and 203 survivors
- 12 recorded predictors in the source dataset
- 6 continuous clinical variables used here
- no missing values in the analysis variables

The six variables are age, creatinine phosphokinase, ejection fraction,
platelets, serum creatinine, and serum sodium.

The source also includes time, a follow-up-duration variable. It is excluded
from the classifiers because treating follow-up duration as an ordinary
predictor could mix outcome-related information into the prediction task. This
project does not perform time-to-event modeling or estimate mortality at a
fixed time horizon.

Observed group differences were visible in the exploratory analysis. The death
group was older on average, had lower ejection fraction, higher serum
creatinine, and lower serum sodium. These are descriptive differences, not
causal effects or adjusted independent risk factors.

![Clinical variable distributions by outcome](figures/group_boxplots.png)

## 3. Research Question

> When LDA, QDA, SVM, and ANN models are evaluated under the same resampling
> design, how do their discrimination performance and error profiles differ?

## 4. Original Analysis and Validation Issue

The original results were not directly comparable:

- LDA and QDA used leave-one-out cross-validation.
- Linear and RBF SVM performance came from predictions on the training data.
- ANN used one 60:40 train/test split.
- ANN scaling was performed before splitting, which leaked preprocessing
  information from the test set.
- QDA priors were supplied in the opposite order from the factor levels.

The RBF SVM's reported 2.68% misclassification rate was therefore a training
error, not an estimate of performance on unseen patients. These issues are
retained in the public coursework copy and explained in
[validation_changes.md](docs/validation_changes.md).

The original Henze-Zirkler tests did not support multivariate normality, and
Box's M test also rejected covariance homogeneity. The updated Rmd re-runs these
checks. LDA and QDA remain useful educational comparators, but their assumptions
are not fully supported.

## 5. Updated Validation Design

- Stratified 5-fold cross-validation
- 5 repeats, producing 25 resamples
- Identical training folds for all models
- Centering and scaling estimated inside each training fold
- Died explicitly defined as the positive class
- ROC-AUC used for tuning
- Sensitivity, specificity, balanced accuracy, and accuracy also reported
- Seed fixed at 1 before fold creation and each model fit

The updated analysis does not add a new model, search multiple seeds, or conduct
a new tuning competition.

## 6. Key Results

The table below matches the freshly rendered analysis.

| Model | ROC-AUC | Sensitivity | Specificity | Balanced Accuracy | Accuracy |
|---|---:|---:|---:|---:|---:|
| LDA | 0.766 | 0.391 | 0.903 | 0.647 | 0.739 |
| QDA | 0.743 | 0.275 | 0.897 | 0.586 | 0.698 |
| Linear SVM | 0.770 | 0.370 | 0.909 | 0.640 | 0.736 |
| RBF SVM | 0.788 | 0.562 | 0.818 | 0.690 | 0.736 |
| ANN | 0.764 | 0.612 | 0.827 | 0.720 | 0.759 |

![ROC-AUC across repeated cross-validation resamples](figures/model_auc_resamples.png)

## 7. Interpretation

RBF SVM has the highest mean ROC-AUC. ANN has the highest sensitivity and
balanced accuracy. LDA and linear SVM have high specificity but low sensitivity
for deaths.

These results do not support selecting one model from accuracy or training error
alone. The preferred model depends on the evaluation objective and the relative
cost of false negatives and false positives.

**한국어 요약:** RBF SVM은 평균 ROC-AUC가 가장 높았고, ANN은 sensitivity와
balanced accuracy가 가장 높았습니다. 따라서 단일 지표만으로 특정 모델이
우월하다고 결론내리지 않고, 평가 목적과 오류 비용에 따른 trade-off를
해석했습니다.

### Exploratory threshold analysis

The pooled repeated-CV out-of-fold predictions produced an AUC of approximately
0.782 for RBF SVM and 0.725 for ANN. Exploratory Youden thresholds were about
0.323 and 0.215, respectively.

Each patient contributes one out-of-fold prediction per repeat. The pooled ROC
curves therefore reuse patients across repeats. They are descriptive summaries,
not independent validation results, and the thresholds are not clinical
cut-offs.

![Exploratory pooled out-of-fold ROC curves](figures/roc_rbf_vs_ann.png)

## 8. What Changed from the Original Coursework

| Item | Original | Updated |
|---|---|---|
| SVM evaluation | Training error | Repeated cross-validation |
| Model comparison | Different validation schemes | Same resampling folds |
| ANN preprocessing | Scaled before train/test split | Estimated inside training folds |
| ANN implementation | `neuralnet` | `nnet` within the shared `caret` workflow; not an exact reproduction |
| QDA priors | Reversed relative to factor-level order | Empirical priors in the common workflow |
| Metrics | Mainly misclassification rate | AUC, sensitivity, specificity, balanced accuracy, accuracy |
| Conclusion | RBF SVM appeared dominant | Model choice depends on the evaluation objective |

## 9. What I Did

- Reviewed the validation procedures in the original coursework.
- Identified that the reported model performances were not directly comparable.
- Detected optimistic evaluation from training-set prediction.
- Redesigned the comparison around identical stratified repeated-CV folds.
- Moved preprocessing inside each training fold.
- Evaluated multiple metrics rather than relying on accuracy alone.
- Interpreted metric trade-offs and documented the remaining limitations.

## 10. Limitations

- Sample size is 299, including 96 deaths.
- The dataset is observational.
- No external validation was performed.
- No nested cross-validation was performed. Hyperparameter selection and
  performance estimation are not fully independent, so modest optimistic bias
  may remain.
- Threshold analysis is exploratory and reuses patients across CV repeats.
- The target is DEATH_EVENT classification rather than formal time-to-event
  modeling.
- The analysis does not predict mortality at a defined time horizon.
- Only six continuous variables were retained for comparability with the
  coursework.
- The updated ANN uses nnet rather than the coursework's neuralnet
  implementation to support the common resampling workflow.

The repository does not claim clinical deployment, medical-device performance,
or independent risk-factor inference.

## 11. Repository Structure

```text
heart-failure-model-validation/
├── README.md
├── analysis/
│   ├── heart_failure_validation.Rmd
│   └── heart_failure_validation.html
├── original_coursework/
│   ├── analysis_original.Rmd
│   └── original_report_redacted.pdf
├── data/
│   ├── README.md
│   └── raw/
│       └── heart_failure_clinical_records_dataset.csv
├── figures/
│   ├── group_boxplots.png
│   ├── model_auc_resamples.png
│   └── roc_rbf_vs_ann.png
├── docs/
│   └── validation_changes.md
├── renv/
│   └── activate.R
├── renv.lock
├── .renvignore
├── .Rprofile
├── .gitignore
└── LICENSE
```

## 12. Reproducibility

From the repository root:

```r
install.packages("renv")
renv::restore()
rmarkdown::render("analysis/heart_failure_validation.Rmd")
```

The render reads only the repository's relative data path and recreates the
three PNG files in figures. The generated HTML includes session information.
Package versions are recorded in renv.lock.

## 13. Data Source and License

Dataset: Heart Failure Clinical Records

- [UCI Machine Learning Repository](https://archive.ics.uci.edu/dataset/519/heart+failure+clinical+records)
- Dataset DOI: [10.24432/C5Z89R](https://doi.org/10.24432/C5Z89R)
- Dataset license: CC BY 4.0
- Source study: Ahmad et al. (2017)
- Reuse and machine-learning study: Chicco and Jurman (2020)

The CSV is redistributed under CC BY 4.0. Repository code and original written
material are released under the MIT License. The MIT License does not replace
the dataset's attribution terms.

## 14. References

- Ahmad, T., Munir, A., Bhatti, S. H., Aftab, M., & Raza, M. A. (2017).
  [Survival analysis of heart failure patients: A case study](https://doi.org/10.1371/journal.pone.0181001).
  PLOS ONE, 12(7), e0181001.
- Chicco, D., & Jurman, G. (2020).
  [Machine learning can predict survival of patients with heart failure from serum creatinine and ejection fraction alone](https://doi.org/10.1186/s12911-020-1023-5).
  BMC Medical Informatics and Decision Making, 20, 16.