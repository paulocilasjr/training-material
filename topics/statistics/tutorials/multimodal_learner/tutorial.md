---
layout: tutorial_hands_on
level: Intermediate
title: Gleam Multimodal Learner - Head and Neck cancer Recurrence Prediction with HANCOCK
zenodo_link: https://zenodo.org/records/17933596
questions:
  - "How do we combine clinical, text, and image modalities to predict HNSCC recurrence?"
  - "How do we configure Multimodal Learner with a published train/test split?"
  - "How do we interpret ROC AUC and class-wise metrics for recurrence prediction?"
objectives:
  - "Import the HANCOCK metadata and CD3/CD8 image archive into Galaxy."
  - "Train a multimodal model with tabular, text, and image backbones."
  - "Evaluate test performance and compare to the HANCOCK benchmark."
time_estimation: "1h"
key_points:
- Multimodal Learner fuses tabular, text, and image modalities in a single run.
- The HANCOCK dataset provides a predefined train/test split for recurrence prediction.
- ROC AUC and class-wise metrics help explain model performance and error patterns.
contributors:
- paulocilasjr
- afpybus
- khaivandangusf2210
- qchiujunhao
- jgoecks
tags:
- HNSCC
- Multimodal Learning
- GLEAM
- HANCOCK Dataset
- Recurrence Prediction
---

> <comment-title>Multimodal Learner Tool</comment-title>
>
> The Multimodal Learner tool described in this tutorial is currently available on:
> [Galaxy US Server](https://usegalaxy.org)
> Statistics and Visualization > Machine Learning > Multimodal Learner
>
> and [Cancer-Galaxy](https://cancer.usegalaxy.org)
> Galaxy Learning and Modeling tools > Multimodal Learner
>
{:  .comment}

In this tutorial, we use the HANCOCK dataset ({% cite Dorrich2025 %}) to build a multimodal model that predicts head and neck cancer recurrence by combining clinical variables, ICD text, and CD3/CD8 immunohistochemistry images. The data are provided as training and test tables plus an image archive ({% cite Hancock2025Data %}). We will train a late-fusion model with GLEAM Multimodal Learner and evaluate performance with ROC AUC and class-wise metrics.

> <agenda-title></agenda-title>
>
> In this tutorial, we will cover:
>
> 1. TOC
> {:toc}
>
{: .agenda}

# Dataset Overview and Modalities

The HANCOCK cohort includes 763 head and neck squamous cell carcinoma (HNSCC) cases with multimodal data ({% cite Dorrich2025 %}). For this tutorial, the train and test tables include the following key columns:

- **target**: recurrence label (0 = no recurrence, 1 = recurrence/progression)
- **icd_codes**: free-text ICD descriptions
- **CD3_image_path** and **CD8_image_path**: filenames for TMA core images

## Modalities Used in the Model

| Modality | Source | Description |
|---|---|---|
| Tabular | Clinical + pathology + labs | Numeric and categorical patient variables |
| Text | ICD codes | Free-text diagnosis strings |
| Image | CD3/CD8 TMA cores | Immunohistochemistry images |

The training and test tables are provided as published splits, which we use to compare against the HANCOCK benchmark.

# Using Multimodal Learner Tool

## Prepare environment and get the data

> <hands-on-title>Environment and Data Upload</hands-on-title>
>
> 1. Create a new history for this tutorial. For example, name it *HANCOCK Multimodal Recurrence*.
>
>    {% snippet faqs/galaxy/histories_create_new.md %}
>
> 2. Import the dataset files from Zenodo:
>
>    ```
>    https://zenodo.org/records/17933596/files/HANCOCK_train_split.csv
>    https://zenodo.org/records/17933596/files/HANCOCK_test_split.csv
>    https://zenodo.org/records/17727354/files/tma_cores_cd3_cd8_images.zip
>    ```
>
>    {% snippet faqs/galaxy/datasets_import_via_link.md %}
>
>> <tip-title>Data Type</tip-title>
>> - For the `.zip` file, set the datatype to `zip`
>> - For the `.csv` files, set the datatype to `csv`
>>
>    {: .tip}
>
> 3. Check that the data formats are assigned correctly. If not, follow the Changing the datatype tip:
>
>    {% snippet faqs/galaxy/datasets_change_datatype.md datatype="datatypes" %}
>
{: .hands_on}

## Tool setup and run

After preparing the train and test tables, configure Multimodal Learner with the following parameters.

> <hands-on-title>Configure Multimodal Learner for HANCOCK</hands-on-title>
>
> 1. {% tool [Multimodal Learner](toolshed.g2.bx.psu.edu/repos/goeckslab/multimodal_learner/multimodal_learner/0.1.0) %} with the following parameters:
>    - {% icon param-file %} *Training dataset (CSV/TSV)*: `HANCOCK_train_split.csv`
>    - {% icon param-select %} *Target / Label column*: `target`
>    - {% icon param-toggle %} *Provide separate test dataset?*: `Yes`
>    - {% icon param-file %} *Test dataset (CSV/TSV)*: `HANCOCK_test_split.csv`
>    - {% icon param-select %} *Text backbone*: `google/electra-base-discriminator`
>    - {% icon param-toggle %} *Use image modality?*: `Yes`
>    - {% icon param-file %} *ZIP file containing images*: `tma_cores_cd3_cd8_images.zip`
>    - {% icon param-select %} *Image backbone*: `caformer_b36.sail_in22k_ft_in1k`
>    - {% icon param-toggle %} *Drop rows with missing images?*: `No`
>    - {% icon param-select %} *Quality preset*: `High quality`
>    - {% icon param-select %} *Primary evaluation metric*: `ROC AUC`
>    - {% icon param-text %} *Random seed*: `42`
>    - {% icon param-toggle %} *Advanced: customize training settings?*: `Yes`
>    - {% icon param-text %} *Validation fraction*: `0.2`
>    - {% icon param-toggle %} *Enable k-fold cross-validation*: `Yes`
>    - {% icon param-text %} *Number of CV folds*: `5`
>    - {% icon param-text %} *Binary classification threshold*: `0.25`
>
> 2. Run the tool and review the HTML report.
>
{: .hands_on}

> <tip-title>Recommended Multimodal Learner configuration (and why)</tip-title>
>
> | Parameter | Rationale |
> |---|---|
> | Text backbone | ELECTRA base performs well on clinical free text |
> | Image backbone | CAFormer b36 matches the HANCOCK benchmark setup |
> | ROC AUC metric | Robust for imbalanced binary outcomes |
> | 5-fold CV | Estimates stability across folds |
> | Threshold 0.25 | Matches the recurrence decision rule used in the use case |
>
{: .tip}

## Tool Output Files

After the run finishes, you should see these outputs in your history:

- **Multimodal Learner analysis report (HTML)**: Summary metrics, ROC/PR curves, and diagnostic plots
- **Multimodal Learner metric results (JSON)**: Machine-readable metrics for train/validation/test
- **Multimodal Learner training config (YAML)**: Full run settings for reproducibility

# Interpreting the Multimodal Learner Report

The HTML report provides a comprehensive overview of the experiment:

- **Performance summary**: ROC AUC, PR AUC, precision, recall, and F1 on each split
- **Diagnostics**: ROC and PR curves, confusion matrix, and class-wise metrics
- **Configuration**: Backbones, modality usage, and training parameters

# Results and Benchmark Comparison

The HANCOCK study reported an ROC AUC of 0.79 for recurrence prediction using engineered multimodal features ({% cite Dorrich2025 %}). Using the raw modalities and a late-fusion model, Multimodal Learner achieves a median ROC AUC of 0.74 on the held-out test set.

| Metric | HANCOCK (reference) | Multimodal Learner |
|---|---:|---:|
| ROC AUC | 0.79 | 0.74 |

Key observations:

- Performance is close to the published benchmark while using raw images and text.
- Class-wise metrics often show stronger performance for the negative class, highlighting the challenge of detecting recurrence.
- The report makes threshold effects and error patterns easy to inspect.

# Conclusion

In this tutorial, we used the GLEAM Multimodal Learner to build a recurrence prediction model for HNSCC by combining tabular, text, and image data from the HANCOCK dataset. The workflow demonstrates how a no-code, standardized approach can reproduce a published benchmark while providing transparent, reproducible reporting.
