---
layout: tutorial_hands_on
level: Intermediate
title: Galaxy Image Learner - Building a Deep Learning Classifier using the HAM10000 Dataset
zenodo_link: https://zenodo.org/records/17114688
questions:
- How can the HAM10000 dataset be prepared for the Image Learner tool?
- How can an image classification model be trained, validated, and tested using GLEAM Image Learner?
- How can the model's performance be evaluated to verify robustness and accuracy?
objectives:
- Use the HAM10000 dataset to build an image classification model.
- Train a deep learning classifier using the GLEAM Image Learner tool in Galaxy.
- Evaluate the model's performance using standard machine learning metrics.
time_estimation: 30m
key_points:
- Use Galaxy tools (Image Learner) to build a deep learning model for skin lesion classification based on the HAM10000 dataset.
- Understand the dataset composition and the importance of data augmentation to handle class imbalance.
- Confirm the robustness of the model by evaluating its performance metrics including accuracy, ROC-AUC, and F1-score.
contributors:
- dangkhai
- paulocilasjr
- qchiujunhao
- jgoecks
tags:
- HAM10000 Dataset
- Image Classification
- Deep Learning
- Image Learner
- Skin Lesion Classification
---


> <comment-title>Image Learner Tool</comment-title>
>
> The Image Learner tool described in this tutorial is currently available on: 
> [Galaxy US Server](https://usegalaxy.org)
> Statistics and Visualization > Machine Learning > Image Learner
>
> and [Cancer-Galaxy](https://cancer.usegalaxy.org)
> Galaxy Learning and Modeling tools > Image Learner
>
{:  .comment}

In this tutorial, we will use the HAM10000 ("Human Against Machine with 10,000 training images") dataset to develop a deep learning classifier for dermoscopic skin lesion classification. The goal is to accurately classify seven types of pigmented skin lesions using the GLEAM Image Learner tool.

To achieve this, we will follow three essential steps: (i) upload the HAM10000 images and metadata to Galaxy, (ii) set up and run the Image Learner tool to train a deep learning model, and (iii) evaluate the model's predictive performance by analyzing key performance metrics such as accuracy, ROC-AUC, and confusion matrices.

![Workflow overview for HAM10000 classification in GLEAM Image Learner](../../images/skin_tutorial/"IMAGE\ LEARNER"\ workflow\ diagram.png "Workflow overview for HAM10000 classification in GLEAM Image Learner")

> <agenda-title></agenda-title>
>
> In this tutorial, we will cover:
>
> 1. TOC
> {:toc}
>
{: .agenda}

> <comment-title>Background</comment-title>
>
> The [HAM10000 dataset](https://zenodo.org/records/17114688) contains 10,015 dermoscopic images collected from multiple sources and standardized for visual contrast and color. The dataset includes metadata with patient age, sex, lesion site, and diagnostic type for each image.
>
> The dataset covers seven types of pigmented skin lesions:
> 1. Melanoma (mel)
> 2. Melanocytic nevus (nv)
> 3. Basal cell carcinoma (bcc)
> 4. Actinic keratosis (akiec)
> 5. Benign keratosis (bkl)
> 6. Dermatofibroma (df)
> 7. Vascular lesion (vasc)
>
> The dataset is highly imbalanced, with melanocytic nevi representing more than 60% of the data. To mitigate this imbalance during training, we employ data augmentation techniques.
>
{:  .comment}

# Dataset Composition and Class Imbalance

Before we begin the hands-on session, here's a brief explanation of the dataset and the challenges associated with its use. The HAM10000 dataset is a benchmark dataset in dermoscopy and skin cancer research, providing standardized images for model development and validation.

## Dataset Overview

The HAM10000 dataset comprises:
- **Total Images**: 10,015 dermoscopic images
- **Image Size**: 96x96 pixels (in this version)
- **Image Format**: PNG
- **Lesion Types**: 7 categories
- **Metadata**: Patient age, sex, lesion location, and diagnostic information

## Class Imbalance Challenge

The dataset exhibits significant class imbalance, which is a common challenge in medical imaging:

| Lesion Type | Count | Percentage |
|---|---|---|
| Melanocytic nevus (nv) | 6,705 | 67.0% |
| Melanoma (mel) | 1,113 | 11.1% |
| Basal cell carcinoma (bcc) | 514 | 5.1% |
| Actinic keratosis (akiec) | 327 | 3.3% |
| Benign keratosis (bkl) | 1,099 | 11.0% |
| Dermatofibroma (df) | 115 | 1.1% |
| Vascular lesion (vasc) | 142 | 1.4% |

This imbalance can cause machine learning models to perform poorly on minority classes. The Image Learner tool provides built-in mechanisms to address this through data augmentation and class weighting strategies.

# Data Augmentation

To mitigate class imbalance and improve model generalization, **horizontal flip augmentation** was applied to the training dataset. This transformation:

- Creates additional training samples by horizontally flipping existing images
- Helps the model learn invariant features that are robust to orientation changes
- Is particularly useful for dermoscopic images where lesion orientation is not clinically significant
- Increases effective dataset size without requiring new data collection

![Example of horizontal flip augmentation applied to a skin lesion image.](../../images/skin_tutorial/Horizontal\ Flip\ augmentation.png "Example of horizontal flip augmentation. Adapted from {% cite Shetty2022 %}.")

> <tip-title>Data Augmentation Rationale</tip-title>
>
> Horizontal flip augmentation is a standard technique in computer vision that:
> - Increases model robustness to variations in lesion orientation
> - Effectively doubles the training set size
> - Helps mitigate the effects of class imbalance by providing more training examples
> - Is applied only to the training set, not validation or test sets, to ensure unbiased evaluation
>
> As demonstrated by {% cite Shetty2022 %}, data augmentation techniques including horizontal flips significantly improve classification accuracy for skin lesion detection in the HAM10000 dataset.
>
{: .tip}

# Model Configuration in GLEAM Image Learner

After uploading the dataset, configure the Image Learner parameters as follows. These settings are based on best practices for dermoscopic image classification and have been optimized for the HAM10000 dataset:

| Parameter | Value | Rationale |
|---|---|---|
| Task Type | Classification | Multi-class image classification task |
| Model Name | caformer_s18_384 | Efficient transformer-based model |
| Epochs | 30 | Sufficient for convergence without overfitting |
| Batch Size | 32 | Balances memory and gradient stability |
| Fine Tune | True | Leverage pre-trained features for better performance |
| Use Pretrained | True | Transfer learning from ImageNet-trained weights |
| Learning Rate | 0.001 | Conservative learning rate for fine-tuning |
| Random Seed | 42 | Reproducible results across runs |
| Data Split | 70/10/20 | Standard split for training/validation/test |
| Data Augmentation | Horizontal Flip | Address class imbalance and improve generalization |

![Model and training summary interface in GLEAM Image Learner.](../../images/skin_tutorial/Image\ Classification\ Results\ –\ Model\ and\ Training\ Summary.png "Model and training summary interface")

# Prepare Environment and Get the Data

> <comment-title>Preprocessing the raw data</comment-title>
>
> The raw HAM10000 dataset from the source has been preprocessed and made available on Zenodo to facilitate this tutorial:
> - Images have been resized to 96x96 pixels for computational efficiency
> - Images have been organized into class folders
> - Metadata has been compiled into CSV format for easy reference
>
> A preprocessing workflow including image resizing and metadata compilation can be found at: 
> [HAM10000 Preprocessing](https://zenodo.org/records/17114688)
>
{:  .comment}

> <hands-on-title> Environment and Data Upload </hands-on-title>
>
> 1. Create a new history for this tutorial. If you are not inspired, you can name it *HAM10000 Image Classification*.
>
>    {% snippet faqs/galaxy/histories_create_new.md %}
>
> 2. Import the dataset files from Zenodo
>
>    ```
>    https://zenodo.org/records/17114688/files/images_96.zip
>    https://zenodo.org/records/17114688/files/image_metadata.csv
>    ```
>
>    {% snippet faqs/galaxy/datasets_import_via_link.md %}
>
>    <tip-title>Data Type</tip-title>
>    - For the `.zip` file, set the datatype to `zip`
>    - For the `.csv` file, leave as `Auto-Detect` (it will be recognized as tabular)
>
>    {: .tip}
>
> 3. Check that the data formats are assigned correctly:
>    - The `.zip` file should have type `zip`
>    - The `.csv` file should have type `tabular`
>
>    If they are not, follow the Changing the datatype tip:
>
>    {% snippet faqs/galaxy/datasets_change_datatype.md datatype="datatypes" %}
>
> 4. Add tags to the datasets for better organization:
>    - Add tag `HAM10000 images` to the images_96.zip file
>    - Add tag `HAM10000 metadata` to the image_metadata.csv file
>
>    {% snippet faqs/galaxy/datasets_add_tag.md %}
>
{: .hands_on}

# Using Image Learner Tool

> <hands-on-title> Task description </hands-on-title>
>
> 1. {% tool [Image Learner](https://toolshed.g2.bx.psu.edu/view/goeckslab/image_learner/c5150cceab47) %} with the following parameters:
>    - {% icon param-file %} *"Input image collection (ZIP)"*: `images_96.zip`
>    - {% icon param-file %} *"Image metadata (CSV)"*: `image_metadata.csv`
>    - {% icon param-select %} *"Task"*: `Classification`
>    - {% icon param-select %} *"Model"*: `caformer_s18_384`
>    - {% icon param-text %} *"Number of epochs"*: `30`
>    - {% icon param-text %} *"Batch size"*: `32`
>    - {% icon param-select %} *"Fine tune pretrained model"*: `Yes`
>    - {% icon param-text %} *"Learning rate"*: `0.001`
>    - {% icon param-text %} *"Random seed"*: `42`
>    - {% icon param-text %} *"Data augmentation"*: `horizontal_flip`
>    - {% icon param-select %} *"Data split (train/validation/test)"*: `70/10/20`
>
> 2. Run the tool
>
{: .hands_on}

# Tool Output Files

After training and testing your model, you should see several new files in your history list:

- **Image Learner Best Model**: The trained model file that can be reused for predictions without retraining.

- **Image Learner Model Report**: An interactive HTML report containing all evaluation plots, performance metrics, and model visualizations.

- **Training History**: A file documenting the training progress (accuracy and loss for each epoch).

- **Predictions (Test Set)**: CSV file with predictions and confidence scores for test images.

For this tutorial, we will focus on the Image Learner Model Report and the performance metrics.

# Image Learner Model Report

The Image Learner HTML report provides a comprehensive and interactive overview of the trained model's performance. This report documents key aspects of the model's training and evaluation process, offering insights into how well the model performed on the training, validation, and test datasets.

## Report Structure

The report typically contains the following sections:

### Model Summary
- Model architecture and configuration
- Training parameters and hyperparameters
- Dataset composition and splits

### Training Performance
- Training and validation loss curves
- Training and validation accuracy curves
- Overall training dynamics

### Test Set Evaluation
The test set evaluation provides comprehensive metrics for assessing final model performance:

![Test Performance Summary - Accuracy and Loss Progression](../../images/skin_tutorial/Test\ Performance\ Summary.png "Test Performance Summary")

### Classification Metrics

After training, the tool generates detailed evaluation metrics:

| Metric | Value | Interpretation |
|---|---|---|
| Accuracy | 0.9036 | 90.36% of test samples correctly classified |
| Precision | 0.9102 | When model predicts positive, it's correct 91.02% of the time |
| Recall | 0.9036 | Model identifies 90.36% of actual positive cases |
| F1-Score | 0.9063 | Balanced measure of precision and recall |
| ROC-AUC | 0.9880 | 98.80% area under the ROC curve - excellent discrimination |
| Cohen's Kappa | 0.8875 | Very good inter-rater agreement; strong classification performance |

### ROC-AUC Curves

The Receiver Operating Characteristic (ROC) curve plots the true positive rate against the false positive rate at different classification thresholds. The Area Under the Curve (AUC) metric summarizes this performance in a single value between 0 and 1, where 1.0 represents perfect classification.

![ROC-AUC curves for all seven lesion classes](../../images/skin_tutorial/ROC-AUC\ Curves.png "ROC-AUC Curves for all lesion classes")

> <tip-title>Interpreting ROC-AUC</tip-title>
>
> - **AUC > 0.9**: Excellent discrimination between classes
> - **AUC 0.8-0.9**: Very good discrimination
> - **AUC 0.7-0.8**: Good discrimination
> - **AUC 0.6-0.7**: Fair discrimination
> - **AUC < 0.6**: Poor discrimination
>
> Our model achieved AUC = 0.9880, indicating excellent ability to distinguish between different skin lesion classes.
>
{: .tip}

### Confusion Matrix

The confusion matrix provides a detailed breakdown of correct and incorrect predictions for each class, showing where the model makes errors.

![Confusion matrix showing classification results for all lesion classes](../../images/skin_tutorial/Confusion\ Matrix.png "Confusion matrix of model predictions")

> <tip-title>Interpreting the Confusion Matrix</tip-title>
>
> - **Diagonal elements**: Correct predictions (True Positives and True Negatives)
> - **Off-diagonal elements**: Misclassifications (False Positives and False Negatives)
> - **High values on diagonal**: Good overall classification performance
> - **Pattern analysis**: Can identify which classes are confused with each other
>
{: .tip}

# Model Performance Analysis

## Overall Assessment

The trained model demonstrates strong performance on the HAM10000 skin lesion classification task:

**Accuracy = 0.9036** - The model correctly classifies approximately 90% of test samples, which is a strong result for a 7-class classification problem on dermoscopic images.

**ROC-AUC = 0.9880** - This excellent score indicates that the model has strong discriminative power across all lesion classes.

**Macro F1 = 0.9063** - The macro F1-score shows balanced performance across all classes, which is particularly important given the class imbalance in the original dataset.

**Cohen's Kappa = 0.8875** - This metric indicates very good agreement beyond chance, confirming the model's reliability.

## Class-Specific Performance

The confusion matrix reveals performance patterns for each lesion type:

- **Well-classified lesions**: Some classes show very high accuracy with minimal confusion
- **Challenge areas**: Certain morphologically similar lesions may show higher misclassification rates
- **Clinical implications**: Classes with lower performance may require additional expert review in clinical settings

# Conclusion

In this tutorial, we demonstrated how to use the Galaxy Image Learner tool to build a deep learning model for skin lesion classification using the HAM10000 dataset. We followed a structured approach consisting of:

- Uploading image datasets (ZIP files) and metadata (CSV)
- Configuring Image Learner with appropriate hyperparameters for transfer learning
- Training a deep learning classifier using the caformer_s18_384 model architecture
- Evaluating the model's performance using comprehensive metrics (accuracy, ROC-AUC, F1-score, Cohen's Kappa)
- Interpreting results through visualizations (ROC curves, confusion matrix, training history)

Throughout the process, we showcased how Image Learner simplifies deep learning workflows, making them accessible to researchers without extensive coding expertise. The model achieved ~90% accuracy and 0.988 ROC-AUC, demonstrating strong performance for dermoscopic image classification.

By the end of this tutorial, you should have a solid understanding of how to:
- Prepare image datasets for deep learning
- Configure transfer learning models for image classification
- Evaluate and interpret deep learning model performance
- Handle class-imbalanced datasets through augmentation
- Apply deep learning to biomedical image classification tasks

The approaches and insights from this tutorial can be generalized to other image classification tasks in biomedical research and beyond.

