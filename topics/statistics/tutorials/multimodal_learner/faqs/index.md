# Frequently Asked Questions

## What is multimodal learning?

Multimodal learning is a machine learning approach that integrates information from multiple data sources or modalities (e.g., images, text, tabular data) to make predictions. By combining complementary information from different modalities, multimodal models can often achieve better performance than single-modality approaches.

## Why use multimodal learning for cancer prediction?

Cancer is a complex disease with multiple contributing factors. Different data types capture different aspects:
- Clinical/pathological data: Standard risk factors and biomarkers
- Medical images: Visual patterns in tissue structure and cellular composition
- Text data: Medical history and comorbidities encoded in diagnostic codes

Integrating these modalities provides a more comprehensive view of the patient's condition.

## What if I have missing data in some modalities?

The Multimodal Learner tool is designed to handle missing data. The model can make predictions using available modalities even when some data is missing. However, performance may be reduced when key modalities are absent.

## How do I choose the fusion strategy?

- **Early fusion**: Combines features from all modalities at the input level. Good when modalities are highly complementary.
- **Late fusion**: Trains separate models for each modality and combines predictions at the decision level. More robust to missing data and modality-specific noise.
- **Hybrid fusion**: Combines aspects of both early and late fusion.

For most applications, late fusion is a good starting point.

## Can I use my own dataset?

Yes! The Multimodal Learner tool can be used with any dataset that includes:
- A CSV file with tabular features, target labels, and dataset split information
- Optional: A ZIP file containing images
- Optional: Text data columns in the CSV file

Follow the same format as the HANCOCK dataset for best results.

## How long does training take?

Training time depends on:
- Dataset size (number of samples)
- Number of modalities
- Model complexity
- Number of epochs
- Available computational resources

For the HANCOCK dataset (~500 training samples, 30 epochs), training typically takes 15-30 minutes on standard Galaxy servers.

## How do I interpret the results?

The Multimodal Learner generates an HTML report with:
- **ROC-AUC curves**: Show discrimination ability (higher is better, >0.9 is excellent)
- **Confusion matrix**: Shows correct and incorrect predictions
- **Accuracy, Precision, Recall, F1-score**: Standard classification metrics
- **Training curves**: Show model convergence over epochs

Focus on ROC-AUC and F1-score for overall performance assessment.

## Can I use the trained model for new predictions?

Yes! The Multimodal Learner outputs a trained model file that can be used for predictions on new patients without retraining. Use the "Multimodal Learner Predict" tool with the saved model.

