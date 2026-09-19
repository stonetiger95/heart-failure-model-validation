# Validation Changes

## Purpose

This document records why the portfolio analysis differs from the original
coursework. The original files are preserved as historical records; incorrect
or incomparable results were not silently overwritten.

## Issues identified

### 1. RBF SVM training-set evaluation

The coursework trained the SVM on the full dataset and predicted the same
records. The reported 2.68% RBF SVM misclassification rate was therefore a
training error. It cannot be compared directly with cross-validated or held-out
performance.

Update: both SVMs are evaluated using the same stratified 5-fold,
5-repeat cross-validation folds as the other models.

### 2. Different validation schemes

LDA and QDA used leave-one-out cross-validation, SVM used training-data
prediction, and ANN used one 60:40 split. Differences in reported error could
therefore reflect validation design rather than model behavior.

Update: all five models use the same 25 resamples. Metrics are averaged across
those resamples.

### 3. QDA prior ordering

The coursework factor levels were ordered Survived, Died, but the supplied prior
vector used death and survival proportions in the opposite order. This assigned
class priors to the wrong levels.

Update: QDA is fit in the shared caret workflow with Died explicitly set as the
positive class and empirical priors used by the model. The original coursework
code remains unchanged except for public-release anonymization.

### 4. ANN preprocessing leakage

The coursework standardized the full dataset before splitting it into training
and test sets. The test-set distribution therefore influenced preprocessing.

Update: caret estimates centering and scaling inside each training fold, then
applies those transformations to the corresponding assessment fold.

### 5. ANN implementation in the update

The coursework used the `neuralnet` package. The validation update uses `nnet`
inside the shared `caret` workflow so that the ANN is evaluated with the same
resampling folds and fold-internal preprocessing as the other classifiers. The
updated ANN is therefore a standardized comparison, not an exact reproduction
of the original network implementation.

## Revised comparison

| Item | Original | Updated |
|---|---|---|
| SVM evaluation | Training error | Repeated cross-validation |
| Model comparison | Mixed validation schemes | Identical resampling folds |
| Preprocessing | Inconsistent; ANN leakage | Fold-internal centering and scaling |
| ANN implementation | `neuralnet` | `nnet` in the shared `caret` workflow |
| Metrics | Mainly error rate | AUC, sensitivity, specificity, balanced accuracy, accuracy |
| Interpretation | RBF SVM appeared dominant | Model choice depends on the evaluation objective |

## Scope retained

The update does not add model families, run a seed search, perform nested
cross-validation, or claim external validation. LDA, QDA, linear SVM, RBF SVM,
and ANN remain the only classifiers. The pooled out-of-fold ROC and threshold
analysis is explicitly exploratory because patients recur across CV repeats.

The time variable is excluded. It records follow-up duration, and this repository
does not turn the coursework into a survival analysis or fixed-horizon mortality
model.