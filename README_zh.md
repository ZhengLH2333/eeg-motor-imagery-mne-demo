# 使用 MNE 进行运动想象 EEG 预处理

本项目展示了一个基于 **MNE-Python** 的运动想象 EEG 信号预处理流程。

项目使用 PhysioNet 的公开数据集 **EEG Motor Movement/Imagery Dataset v1.0.0**，目标是将原始 EDF 格式 EEG 文件处理成可以用于后续机器学习分类的 epochs 数据。

---

## 项目目标

本项目重点是 **EEG 信号预处理**，不是追求分类准确率。

当前 Notebook 完成从原始 EEG 文件到机器学习可用数据的准备流程，最终得到：

```python
raw_clean_filt
epochs
X
y
```

其中：

- `raw_clean_filt`：预处理后的连续 EEG 信号
- `epochs`：按照事件切分好的 EEG trials
- `X`：后续模型可使用的 EEG 数据矩阵
- `y`：对应的分类标签

后续可以继续在此基础上添加 CSP + LDA、SVM、深度学习模型等分类方法。

---

## 数据集来源

本项目使用公开数据集：

**EEG Motor Movement/Imagery Dataset v1.0.0**

数据来源：

- PhysioNet
- 数据集页面：https://www.physionet.org/content/eegmmidb/1.0.0/
- 发布时间：2009 年 9 月 9 日
- 版本：1.0.0
- DOI：https://doi.org/10.13026/C28G6P
- 数据许可证：Open Data Commons Attribution License v1.0

该数据集包含 64 通道 EEG 记录，受试者完成了一系列真实运动和运动想象任务。数据以 EDF+ 格式提供，并包含 annotation channel。

---

## 本项目包含的数据

为了保证本项目可以直接复现，本仓库只包含本 demo 使用的 3 个 EDF 文件：

```text
data/eegbci_manual/S001/S001R06.edf
data/eegbci_manual/S001/S001R10.edf
data/eegbci_manual/S001/S001R14.edf
```

这些文件只是完整 EEGMMIDB 数据集中的一个很小子集。

本仓库没有包含完整的 EEG Motor Movement/Imagery Dataset。

---

## 当前使用的实验任务

本项目使用 Subject 1 的三个 imagined movement runs：

```text
S001R06.edf
S001R10.edf
S001R14.edf
```

在 EEGBCI / PhysioNet 的实验设计中：

```text
Runs 6, 10, 14 = Task 4
Task 4 = imagine opening and closing both fists or both feet
T1 = imagined both fists
T2 = imagined both feet
```

因此，本项目当前处理的目标任务是：

```text
imagined both fists vs imagined both feet
```

---

## 预处理流程

Notebook 中完成了以下步骤：

1. 读取本地 EDF EEG 文件
2. 标准化 EEGBCI 通道名称
3. 设置 `standard_1005` 电极位置
4. 保存原始 Raw 副本，避免不可逆操作影响原始数据
5. 可视化原始 EEG 信号
6. 进行平均参考
7. 使用 ICA 作为可选伪迹检查步骤
8. 对数据进行 7–30 Hz 带通滤波
9. 从 annotations 中提取 T1 / T2 事件
10. 将连续 EEG 切分为 epochs
11. 检查 epochs 数据形状和标签分布
12. 保存关键预处理可视化结果

---

## 为什么使用 `standard_1005`？

EEGBCI 数据集中包含 64 个 EEG 通道，其中包括一些中间位置的通道名，例如：

```text
FC1, FC2, C1, C2, CP1, CP2
```

这些通道更适合使用更密集的 `standard_1005` 电极模板，而不是较基础的 `standard_1020` 模板。

---

## 平均参考

EEG 信号是相对电位，必须有参考点。

本项目使用平均参考：

```python
raw_ref.set_eeg_reference(ref_channels="average", projection=False)
```

这一步在副本 `raw_ref` 上完成，不会修改原始数据副本 `raw_original`。

---

## ICA 伪迹检查

ICA 是本项目中的可选步骤。

本 Notebook 支持使用 MNE 的交互式 ICA 浏览器，通过鼠标点击选择需要排除的 ICA 成分。

如果没有发现明显伪迹成分，则不删除任何 ICA 成分，直接使用平均参考后的数据继续后续处理。

注意：

```python
ICA_EXCLUDE = [0, 2]
```

这里的 `0` 和 `2` 表示 ICA 成分编号，不是 EEG 通道名。

不要写成：

```python
ICA_EXCLUDE = ["Fp1", "Fpz"]
```

---

## 滤波

本项目使用 7–30 Hz 带通滤波。

原因是运动想象 EEG 通常关注：

```text
mu rhythm: 8–13 Hz
beta rhythm: 13–30 Hz
```

代码中：

```python
raw_clean_filt.filter(
    l_freq=7.0,
    h_freq=30.0
)
```

本身就是带通滤波，不需要拆成高通和低通两步。

---

## Epoch 切分

本项目根据 T1 / T2 事件切分 epochs。

时间窗口为：

```text
0.5 s 到 2.5 s
```

即每个 trial 取 cue 出现后 0.5 到 2.5 秒的 EEG 信号。

这样可以避开 cue 刚出现的瞬间，更聚焦于运动想象阶段。

---

## 项目结构

```text
eeg-motor-imagery-mne-demo/
├── README.md
├── requirements.txt
├── .gitignore
├── LICENSE
├── notebooks/
│   └── motor_imagery_preprocessing.ipynb
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

## 安装依赖

## 环境安装

本项目推荐使用 **Miniforge**，并通过 **conda-forge** 创建 Python 环境。

不建议直接在系统 Python 或全局环境中安装依赖。  
推荐为本项目创建一个独立环境，避免包版本冲突。

### 1. 安装 Miniforge

请先安装 Miniforge：

https://github.com/conda-forge/miniforge

安装完成后，打开 **Miniforge Prompt**。

### 2. 创建项目环境

在项目根目录下运行：

```bash
conda env create -f environment.yml
```

该命令会根据 `environment.yml` 自动创建名为 `mne_eeg` 的环境。

### 3. 激活环境

```bash
conda activate mne_eeg
```

### 4. 注册 Jupyter Kernel

为了在 Jupyter Notebook 中选择该环境作为运行内核，请运行：

```bash
python -m ipykernel install --user --name mne_eeg --display-name "Python (mne_eeg)"
```

### 5. 启动 JupyterLab

在项目根目录运行：

```bash
jupyter lab
```

然后打开：

```text
notebooks/motor_imagery_preprocessing.ipynb
```

在 Notebook 右上角选择 kernel：

```text
Python (mne_eeg)
```

然后从上到下运行所有 cell。

### 6. 环境文件说明

本项目使用 `environment.yml` 管理依赖，而不是优先使用 `pip install -r requirements.txt`。

原因是本项目使用了 MNE 的交互式图形窗口，例如：

```text
mne-qt-browser
pyqt
```

这些包通过 `conda-forge` 安装通常更稳定。

### 7. 如果环境创建失败

可以先更新 conda：

```bash
conda update -n base conda -y
```

然后重新运行：

```bash
conda env create -f environment.yml
```

如果环境已经存在，可以先删除旧环境：

```bash
conda env remove -n mne_eeg
```

再重新创建：

```bash
conda env create -f environment.yml
```

---

## 运行方式

打开 Notebook：

```text
notebooks/motor_imagery_preprocessing.ipynb
```

然后从上到下运行所有 cell。

---

## 输出结果

Notebook 会在 `results/` 文件夹中保存预处理过程中的可视化结果，例如：

- 原始 EEG 波形
- 平均参考前后对比图
- ICA 前后检查图
- 7–30 Hz 滤波前后对比图
- PSD 频谱对比图
- 事件分布图
- Epochs image

---

## 引用说明

使用该数据集时，请引用原始 BCI2000 论文和 PhysioNet。

原始论文：

Schalk, G., McFarland, D.J., Hinterberger, T., Birbaumer, N., & Wolpaw, J.R. (2004).  
BCI2000: A General-Purpose Brain-Computer Interface (BCI) System.  
IEEE Transactions on Biomedical Engineering, 51(6), 1034–1043.

PhysioNet 引用：

Goldberger, A., Amaral, L., Glass, L., Hausdorff, J., Ivanov, P. C., Mark, R., ... & Stanley, H. E. (2000).  
PhysioBank, PhysioToolkit, and PhysioNet: Components of a new research resource for complex physiologic signals.  
Circulation [Online], 101(23), e215–e220. RRID:SCR_007345.

---

## 许可证

本仓库包含：

1. 本项目代码和文档
2. 生成的结果图片
3. 来自 PhysioNet 的 3 个 EDF 原始数据文件

许可证说明：

- 本项目代码：MIT License
- 本项目包含的 EDF 文件：遵循 PhysioNet 数据集页面指定的 Open Data Commons Attribution License v1.0

请在使用数据时遵守 PhysioNet 的数据使用条款，并保留数据来源和引用信息。