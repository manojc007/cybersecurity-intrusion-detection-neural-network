# Cybersecurity Intrusion Detection using Deep Learning

## Problem Statement

Detecting suspicious processes from system call data using deep learning. This project builds a binary classifier that distinguishes between normal and suspicious process activity based on process metadata features, enabling automated threat detection in real-time environments.

## Dataset

The dataset consists of labeled system call records split into three sets:

| Split | Samples | Description |
|-------|---------|-------------|
| Train | ~763,000 | Used for model training |
| Test | ~175,000 | Used for final evaluation |
| Validation | ~191,000 | Used for hyperparameter tuning and early stopping |

### Features (7)

| Feature | Description |
|---------|-------------|
| `processId` | Process identifier |
| `threadId` | Thread identifier |
| `parentProcessId` | Parent process identifier |
| `userId` | User identifier |
| `mountNamespace` | Mount namespace identifier |
| `argsNum` | Number of arguments |
| `returnValue` | System call return value |

### Target

- `sus_label`: Binary label (0 = Normal, 1 = Suspicious)

### Known Limitation: Train/Test Distribution Mismatch

The training and test sets have drastically different class distributions:

| Split | Normal | Suspicious |
|-------|--------|------------|
| Train | ~99.8% | ~0.2% |
| Test | ~9.3% | ~90.7% |
| Validation | ~99.7% | ~0.3% |

This mismatch means the model trains on overwhelmingly normal data but is evaluated on mostly suspicious data. This is documented transparently and addressed through weighted loss functions in the improved model.

## Methodology

1. **Preprocessing:** StandardScaler fitted on training data, applied to all splits
2. **Architecture:** PyTorch neural network (7 → 16 → 8 → 1) with ReLU activations
3. **Original Model:** BCELoss + SGD optimizer, 10 epochs
4. **Improved Model (NetV2):** BCEWithLogitsLoss with pos_weight for class imbalance, Adam optimizer, early stopping with patience of 5, up to 50 epochs

## Results

### Original Model

| Metric | Score |
|--------|-------|
| Accuracy | 0.95 |
| Precision | 1.00 |
| Recall | 0.94 |
| F1 Score | 0.97 |

### Improved Model (NetV2)

The improved model addresses class imbalance through weighted loss and uses numerically stable BCEWithLogitsLoss with early stopping to prevent overfitting. See the notebook for full evaluation metrics including ROC-AUC and Precision-Recall curves.

## Cybersecurity Relevance

- **Real-time threat detection:** The lightweight neural network architecture enables fast inference on streaming process data
- **Reducing false alert fatigue:** Balancing precision and recall helps security analysts focus on genuine threats rather than being overwhelmed by false positives
- **SIEM integration:** The model's binary output can feed into Security Information and Event Management systems as an additional signal layer

## How to Run

1. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

2. Ensure data files are in the project directory:
   - `labelled_train.csv`
   - `labelled_test.csv`
   - `labelled_validation.csv`

3. Open and run the notebook:
   ```bash
   jupyter notebook cyber.ipynb
   ```

4. Execute cells sequentially from top to bottom.

## Future Improvements

- **Address data split issue:** Resample or re-split data to ensure consistent class distributions across train/test/validation sets
- **LSTM for sequential system calls:** Capture temporal patterns in process behavior by modeling system call sequences
- **Real-time inference pipeline:** Build a streaming pipeline that classifies processes as they execute
- **Feature engineering:** Incorporate additional system call attributes, process trees, and temporal features
- **Ensemble methods:** Combine neural network predictions with tree-based models for improved robustness
