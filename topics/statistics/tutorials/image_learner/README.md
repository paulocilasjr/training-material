# Tutorial - Galaxy Image Learner: HAM10000 Skin Lesion Classification

A Galaxy tutorial for building and evaluating a deep learning image classifier using the HAM10000 dataset with the Image Learner tool (Dang et al., 2025)

## Overview

This tutorial demonstrates how to use the Galaxy Image Learner tool to build a deep learning classifier for skin lesion classification. The HAM10000 dataset contains 10,015 dermoscopic images across 7 types of pigmented skin lesions. Using Galaxy's Image Learner, we train a transfer learning model, evaluate its performance, and interpret the results.

## Key Features

- **Dataset**: HAM10000 - 10,015 dermoscopic images of skin lesions
- **Task**: Multi-class image classification (7 lesion types)
- **Model**: Deep learning with transfer learning (caformer_s18_384)
- **Metrics**: Accuracy, ROC-AUC, F1-score, Cohen's Kappa
- **Approach**: No-code machine learning using Galaxy tools

## Dataset Information

- **Total Images**: 10,015
- **Image Resolution**: 96x96 pixels
- **Lesion Classes**: 7 types (melanoma, nevus, basal cell carcinoma, etc.)
- **Metadata**: Patient age, sex, lesion location, diagnostic information
- **Source**: [Zenodo](https://zenodo.org/records/17114688)

## Model Performance

- **Accuracy**: 90.36%
- **ROC-AUC**: 0.9880
- **Macro F1**: 0.9063
- **Cohen's Kappa**: 0.8875

## Tutorial Duration

Estimated time: 2 hours

## Contributors

- Khai Dang (first author, ORCID: https://orcid.org/0009-0007-1912-8644)
- Paulo Cilas Morais Lyra Junior (ORCID: https://orcid.org/0000-0002-4403-6684)
- Junhao Qiu (ORCID: https://orcid.org/0009-0005-4322-3401)
- Jeremy Goecks (ORCID: https://orcid.org/0000-0002-4583-5226)

## Files

- `tutorial.md` - Main tutorial content
- `tutorial.bib` - Bibliography file
- `data-library.yaml` - Data library configuration
- `workflows/` - Galaxy workflow files
- `faqs/` - Frequently asked questions

## Related Resources

- [HAM10000 Dataset on Zenodo](https://zenodo.org/records/17114688)
- [Galaxy Learning and Modeling (GLEAM)](https://usegalaxy.org)
- [Cancer Galaxy](https://cancer.usegalaxy.org)

