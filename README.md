# Human Activity Recognition with a Hidden Markov Model

This project recognizes human activities from mobile inertial-measurement-unit
(IMU) data using a continuous Gaussian Hidden Markov Model (HMM). The
repository contains accelerometer and gyroscope recordings for four activity
classes:

- **Still**
- **Standing**
- **Walking**
- **Jumping**

The complete analysis is implemented in
[`notebook/human_activity_hmm.ipynb`](./notebook/human_activity_hmm.ipynb).
The notebook explores and synchronizes the sensor streams, extracts
window-level features, selects the most discriminative features, estimates HMM
parameters from the labelled training data, and evaluates Viterbi-decoded
predictions on held-out recordings.

## Repository structure

```text
.
├── dataset/
│   ├── train/                 # 50 labelled recording sessions
│   └── test/                  # 8 held-out recording sessions
└── notebook/
│   └── human_activity_hmm.ipynb
├── report/
│   ├── Formative 2_HMM_Hamed.pdf
├── requirements.txt
```

Each recording session is a directory whose name identifies the split and
activity, for example `train_walking_01-2026-10-28_08-18-49/`. A session
contains:

```text
Accelerometer.csv
Gyroscope.csv
Metadata.csv
```

The sensor files contain timestamp, elapsed-time, and three-axis (`x`, `y`,
`z`) measurements. The metadata file records information such as the device,
recording time, sensor sampling rate, and platform.

## Requirements

- Python 3.9 or newer
- Jupyter Notebook or JupyterLab
- The Python packages listed in
  [`requirements.txt`](./requirements.txt)

The notebook also works in Google Colab. No separate HMM package is required:
the Gaussian HMM used by the project is implemented in the notebook with
NumPy.

## Local setup

1. Clone the repository and enter the project directory:

   ```bash
   git clone https://github.com/hd77alu/human_sensor_activity_hmm.git
   cd human_sensor_activity_hmm
   ```

2. Create and activate a virtual environment:

   **Windows PowerShell**

   ```powershell
   python -m venv .venv
   .\.venv\Scripts\Activate.ps1
   ```

   **macOS/Linux**

   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   ```

3. Install the dependencies:

   ```bash
   python -m pip install --upgrade pip
   python -m pip install -r requirements.txt
   ```

4. Start Jupyter from the repository root:

   ```bash
   jupyter notebook
   ```

5. Open
   [`notebook/human_activity_hmm.ipynb`](./notebook/human_activity_hmm.ipynb)
   and run the cells from top to bottom.

### Dataset paths

When running locally, set the notebook's dataset paths to the directories in
this repository before running the data-loading cells:

```python
train_folder = "dataset/train"
test_folder = "dataset/test"
```

The notebook also contains the Google Drive paths used for the original Colab
workflow. Use those paths only when the dataset has been copied to Google
Drive.

## Google Colab

Open the notebook directly in Colab using the
[Open in Colab](https://colab.research.google.com/github/hd77alu/human_sensor_activity_hmm/blob/main/notebook/human_activity_hmm.ipynb)
link at the top of the notebook. If the dataset is stored in Google Drive:

1. Upload or copy the `dataset` directory to Drive.
2. Run the Drive-mount cell and authorize access.
3. Set `train_folder` and `test_folder` to the corresponding Drive paths.
4. Run the notebook cells in order.

## Analysis workflow

The notebook follows this pipeline:

1. **Load and label data** from the training and test session directories.
   Activity labels are inferred from the session directory names.
2. **Align sensor streams** using their timestamps so accelerometer and
   gyroscope observations describe the same moments.
3. **Explore the data** with sensor-magnitude distributions, time-series
   visualizations, and joint accelerometer/gyroscope plots.
4. **Extract features** from two-second windows with 50% overlap. The feature
   set includes statistical and frequency-domain summaries of the sensor
   signals, producing 45 candidate features.
5. **Select features** using ANOVA F-scores and standardize the selected
   features with z-score normalization.
6. **Estimate HMM parameters** for the four activity states. Emissions are
   modelled with diagonal-covariance multivariate Gaussian distributions, and
   transition probabilities capture temporal persistence between activities.
7. **Decode and evaluate** the test sequences with the Viterbi algorithm using
   accuracy, a classification summary, and a confusion matrix.

The HMM class also includes a Baum-Welch/forward-backward implementation for
future extensions, but this notebook's primary experiment initializes model
parameters from the supervised labels and uses Viterbi decoding for evaluation.

## Results

The evaluation results below summarize the model's performance for each
activity state on the test data:

| State (Activity) | Number of Samples | Sensitivity (Recall) | Specificity | Overall Accuracy |
|---|---:|---:|---:|---:|
| Jumping | 10 | 1.0000 | 1.0000 | 1.0000 |
| Standing | 11 | 1.0000 | 1.0000 | 1.0000 |
| Still | 9 | 1.0000 | 1.0000 | 1.0000 |
| Walking | 9 | 1.0000 | 1.0000 | 1.0000 |

All four activity classes achieved perfect sensitivity, specificity, and
overall accuracy in this evaluation.

## Reproducibility notes

- Run the notebook from the repository root so relative dataset paths resolve
  correctly.
- Execute cells in order because later cells use data frames, selected
  features, and model parameters created earlier.
- Keep the `dataset/train` and `dataset/test` split unchanged when comparing
  results with the notebook's original evaluation.
- The repository includes the raw sensor recordings; avoid renaming session
  directories because their names provide the activity labels.

## Limitations

This is an experimental notebook implementation rather than a packaged
command-line application. The model is trained and evaluated within the
notebook, and the current labels are derived from recording-directory names.
Performance may change for different devices, sampling rates, users, sensor
placements, or activities not represented in the dataset.
