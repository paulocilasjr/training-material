# Tutorial - Galaxy Multimodal Learner: HANCOCK HNSCC Recurrence Prediction

A Galaxy tutorial for building and evaluating a multimodal recurrence prediction model using the HANCOCK dataset with the Multimodal Learner tool (Dorrich et al., 2025)

## Overview

This tutorial demonstrates how to use the Galaxy Multimodal Learner tool to build a recurrence prediction model for head and neck squamous cell carcinoma (HNSCC). The tutorial follows the HANCOCK use case, combining tabular clinical variables, ICD text, and CD3/CD8 immunohistochemistry images in a single, late-fusion model. We train a multimodal model, evaluate ROC AUC performance, and compare results to the published benchmark.

## Key Features

- **Dataset**: HANCOCK multimodal cohort (clinical, text, and imaging data)
- **Task**: Binary classification (recurrence vs no recurrence)
- **Model**: AutoGluon multimodal late fusion (FT-Transformer, ELECTRA, CAFormer)
- **Metrics**: ROC AUC, PR AUC, precision, recall, F1
- **Approach**: No-code machine learning using Galaxy tools

## Dataset Information

- **Patients**: 763 HNSCC cases with multimodal data
- **Modalities**:
  - Clinical tabular variables (demographics, staging, labs)
  - ICD code text (free-text descriptions)
  - CD3 and CD8 TMA core images
- **Label**: Recurrence status (0 = no recurrence, 1 = recurrence/progression)
- **Split**: Predefined train/test partition from HANCOCK
- **Source**: [Zenodo (main record)](https://zenodo.org/records/17933596)
- **Train split CSV**: https://zenodo.org/records/17933596/files/HANCOCK_train_split.csv
- **Test split CSV**: https://zenodo.org/records/17933596/files/HANCOCK_test_split.csv
- **Images ZIP**: https://zenodo.org/records/17727354/files/tma_cores_cd3_cd8_images.zip

## Model Performance

- **Median ROC AUC**: 0.74 (Multimodal Learner)
- **Reference ROC AUC**: 0.79 (HANCOCK benchmark)

## Tutorial Duration

Estimated time: 1 hour

## Contributors

- Alyssa Pybus (ORCID: https://orcid.org/0000-0002-9722-3977)
- Paulo Cilas Morais Lyra Junior (ORCID: https://orcid.org/0000-0002-4403-6684)
- Junhao Qiu (ORCID: https://orcid.org/0009-0005-4322-3401)
- Khai Dang (ORCID: https://orcid.org/0009-0007-1912-8644)
- Jeremy Goecks (ORCID: https://orcid.org/0000-0002-4583-5226)

## Files

- `tutorial.md` - Main tutorial content
- `tutorial.bib` - Bibliography file
- `data-library.yaml` - Data library configuration
- `slides.html` - Slides for the tutorial

## Related Resources

- [HANCOCK Dataset on Zenodo (main record)](https://zenodo.org/records/17933596)
- [HANCOCK Train Split CSV](https://zenodo.org/records/17933596/files/HANCOCK_train_split.csv)
- [HANCOCK Test Split CSV](https://zenodo.org/records/17933596/files/HANCOCK_test_split.csv)
- [HANCOCK CD3/CD8 Images ZIP](https://zenodo.org/records/17727354/files/tma_cores_cd3_cd8_images.zip)
- [GLEAM Toolkit on GitHub](https://github.com/goeckslab/gleam)
- [Galaxy Learning and Modeling (GLEAM)](https://usegalaxy.org)
