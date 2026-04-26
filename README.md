# EEG Motor Imagery Preprocessing with MNE
*[中文版说明请点击这里](README_zh.md)*
This project demonstrates an EEG motor imagery preprocessing pipeline using **MNE-Python** and the public **EEG Motor Movement/Imagery Dataset v1.0.0** from PhysioNet.

The goal of this project is to process raw EDF EEG files into clean, epoch-based EEG data that can be used for downstream machine learning classification.

This repository focuses on **EEG signal preprocessing**, not classification accuracy.

---

## Project Overview

The notebook processes EEG motor imagery data for the task:

```text
imagined both fists vs imagined both feet
```

The final preprocessed outputs are:

```python
raw_clean_filt
epochs
X
y
```

Where:

- `raw_clean_filt` is the preprocessed continuous EEG signal
- `epochs` contains event-locked EEG trials
- `X` is the EEG data array for downstream machine learning
- `y` is the label array

---

## Dataset Source

This project uses the following public dataset:

**EEG Motor Movement/Imagery Dataset v1.0.0**

- Source: PhysioNet
- Dataset page: https://www.physionet.org/content/eegmmidb/1.0.0/
- Published: September 9, 2009
- Version: 1.0.0
- DOI: https://doi.org/10.13026/C28G6P
- Data license: Open Data Commons Attribution License v1.0

The dataset contains 64-channel EEG recordings from subjects performing motor and motor imagery tasks. The EEG files are provided in EDF+ format and include annotation channels.

---

## Included Raw Data

For reproducibility, this repository includes only the three EDF files used in this demo:

```text
data/eegbci_manual/S001/S001R06.edf
data/eegbci_manual/S001/S001R10.edf
data/eegbci_manual/S001/S001R14.edf
```

These files are a small subset of the full EEG Motor Movement/Imagery Dataset.

The full PhysioNet dataset is **not** included in this repository.

---

## Task Used in This Demo

This project uses Subject 1 and the following imagined movement runs:

```text
S001R06.edf
S001R10.edf
S001R14.edf
```

In the EEGBCI / PhysioNet experimental protocol:

```text
Runs 6, 10, 14 = Task 4
Task 4 = imagine opening and closing both fists or both feet
T1 = imagined both fists
T2 = imagined both feet
```

Therefore, the target task prepared by this preprocessing pipeline is:

```text
imagined both fists vs imagined both feet
```

---

## Preprocessing Pipeline

The notebook performs the following steps:

1. Load local EDF EEG files
2. Standardize EEGBCI channel names
3. Set the `standard_1005` electrode montage
4. Preserve a copy of the original Raw object to avoid irreversible changes
5. Visualize the raw EEG signal
6. Apply average reference
7. Optionally inspect and remove artifacts using ICA
8. Apply 7–30 Hz band-pass filtering
9. Extract T1 / T2 events from annotations
10. Create epochs from continuous EEG
11. Check epoch data shape and label distribution
12. Save key preprocessing visualizations

---

## Why `standard_1005`?

The EEGBCI dataset contains 64 EEG channels, including intermediate channel names such as:

```text
FC1, FC2, C1, C2, CP1, CP2
```

These channels are better covered by the denser `standard_1005` montage than by the more basic `standard_1020` montage.

---

## Average Reference

EEG signals are reference-dependent.

This project applies average reference using:

```python
raw_ref.set_eeg_reference(ref_channels="average", projection=False)
```

This operation is performed on a copied object, so the original raw data remains unchanged.

---

## Optional ICA Artifact Inspection

ICA is included as an optional artifact inspection step.

The notebook supports manual ICA component selection using MNE's interactive ICA browser.

If no clear artifact component is found, the pipeline skips ICA removal and continues with the average-referenced data.

Important note:

```python
ICA_EXCLUDE = [0, 2]
```

The numbers in `ICA_EXCLUDE` refer to ICA component indices, not EEG channel names.

This is incorrect:

```python
ICA_EXCLUDE = ["Fp1", "Fpz"]
```

---

## Filtering

A 7–30 Hz band-pass filter is applied because motor imagery EEG is commonly associated with:

```text
mu rhythm: 8–13 Hz
beta rhythm: 13–30 Hz
```

In the notebook:

```python
raw_clean_filt.filter(
    l_freq=7.0,
    h_freq=30.0
)
```

This is already a band-pass filter and does not need to be split into separate high-pass and low-pass filtering steps.

---

## Epoching

Epochs are created based on T1 / T2 events.

The epoch time window is:

```text
0.5 s to 2.5 s
```

This means that each trial uses EEG data from 0.5 seconds to 2.5 seconds after the cue onset.

This window avoids the immediate cue onset period and focuses more on the motor imagery interval.

---

## Project Structure

```text
eeg-motor-imagery-mne-demo/
├── README.md
├── README_zh.md
├── environment.yml
├── .gitignore
├── LICENSE
├── notebooks/
│   └── motor_imagery_demo_v2.ipynb
├── data/
│   └── eegbci_manual/
│       └── S001/
│           ├── S001R06.edf
│           ├── S001R10.edf
│           └── S001R14.edf
├── results/
│   ├── 01_original_raw_eeg.png
│   ├── 02_after_average_reference.png
│   ├── 03_reference_comparison_cz.png
│   ├── 07_after_7_30hz_filter.png
│   ├── 08_filter_comparison_cz.png
│   ├── 09_filter_psd_comparison.png
│   ├── 10_events_distribution.png
│   └── 11_epochs_image_cz.png
└── docs/
    └── notes.md
```

---

## Environment Setup

This project recommends using **Miniforge** and installing dependencies from **conda-forge**.

It is not recommended to install the dependencies into the system Python or a global Python environment. A separate conda environment is recommended to avoid package conflicts.

### 1. Install Miniforge

Install Miniforge from:

```text
https://github.com/conda-forge/miniforge
```

After installation, open **Miniforge Prompt**.

### 2. Create the Project Environment

From the project root directory, run:

```bash
conda env create -f environment.yml
```

This command creates a conda environment named:

```text
mne_eeg
```

### 3. Activate the Environment

```bash
conda activate mne_eeg
```

### 4. Register the Jupyter Kernel

To make this environment available inside Jupyter Notebook, run:

```bash
python -m ipykernel install --user --name mne_eeg --display-name "Python (mne_eeg)"
```

### 5. Start JupyterLab

From the project root directory, run:

```bash
jupyter lab
```

Then open:

```text
notebooks/motor_imagery_demo_v2.ipynb
```

In the notebook interface, select the kernel:

```text
Python (mne_eeg)
```

Then run all cells from top to bottom.

### 6. Why Use `environment.yml`?

This project uses `environment.yml` as the recommended dependency file.

The project uses MNE interactive visualization tools such as:

```text
mne-qt-browser
pyqt
```

These packages are usually more stable when installed from `conda-forge`.

---

## Example Outputs

The notebook saves preprocessing visualizations to the `results/` folder.

Example outputs include:

- Raw EEG sample
- Average reference comparison
- Optional ICA effect check
- 7–30 Hz filtering comparison
- PSD comparison
- Event distribution
- Epochs image

---

## Citation

When using this dataset, please cite the original BCI2000 publication and PhysioNet.

Original publication:

Schalk, G., McFarland, D.J., Hinterberger, T., Birbaumer, N., & Wolpaw, J.R. (2004).  
BCI2000: A General-Purpose Brain-Computer Interface (BCI) System.  
IEEE Transactions on Biomedical Engineering, 51(6), 1034–1043.

PhysioNet citation:

Goldberger, A., Amaral, L., Glass, L., Hausdorff, J., Ivanov, P. C., Mark, R., ... & Stanley, H. E. (2000).  
PhysioBank, PhysioToolkit, and PhysioNet: Components of a new research resource for complex physiologic signals.  
Circulation [Online], 101(23), e215–e220. RRID:SCR_007345.

---

## License

This repository contains:

1. Project code and documentation
2. Generated result figures
3. Three EDF files from the PhysioNet EEG Motor Movement/Imagery Dataset

License information:

- Project code: MIT License
- Included EDF files: Open Data Commons Attribution License v1.0, as specified by PhysioNet

Please follow PhysioNet's terms when using the dataset and keep the dataset source and citation information.