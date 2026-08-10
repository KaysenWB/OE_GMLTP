<h1 align="center">GMLTP</h1>

<p align="center">
  <strong>Graph-driven Multi-vessel Long-term Trajectory Prediction for Safer Route Planning</strong>
</p>

<p align="center">
  Official PyTorch implementation of the <em>Ocean Engineering</em> paper<br>
  <a href="https://doi.org/10.1016/j.oceaneng.2024.119511"><em>Graph-driven multi-vessel long-term trajectories prediction for route planning under complex waters</em></a>
</p>

<p align="center">
  <a href="https://doi.org/10.1016/j.oceaneng.2024.119511"><img src="https://img.shields.io/badge/Paper-Ocean%20Engineering-0B6E99" alt="Paper"></a>
  <a href="https://doi.org/10.1016/j.oceaneng.2024.119511"><img src="https://img.shields.io/badge/DOI-10.1016%2Fj.oceaneng.2024.119511-B31B1B" alt="DOI"></a>
  <a href="https://www.python.org/"><img src="https://img.shields.io/badge/Python-3.9-3776AB?logo=python&logoColor=white" alt="Python 3.9"></a>
  <a href="https://pytorch.org/"><img src="https://img.shields.io/badge/PyTorch-implementation-EE4C2C?logo=pytorch&logoColor=white" alt="PyTorch"></a>
  <a href="./License"><img src="https://img.shields.io/badge/License-Apache--2.0-4C1" alt="Apache-2.0 License"></a>
</p>

<p align="center">
  <a href="#highlights">Highlights</a> •
  <a href="#results-at-a-glance">Results</a> •
  <a href="#quick-start">Quick start</a> •
  <a href="#citation">Citation</a>
</p>

---

GMLTP forecasts the long-term trajectories of multiple interacting vessels from processed Automatic Identification System (AIS) data. It combines a **multi-graph spatial convolution (MGSC)** module for fine-grained vessel and waterway interactions with a **ProbSparse self-attention Transformer** for efficient long-range temporal modeling. The resulting forecasts give real-time route planners a longer view of moving obstacles, supporting earlier and safer navigation decisions in complex waters.

> **Key result:** On 50,000 real-world multi-vessel encounter scenarios from Victoria Harbour, GMLTP predicts 6–96 future steps (1–16 minutes at 10-second intervals) and achieves the best results across all four reported metrics from 24 to 96 steps.

> **Repository scope:** This repository contains the GMLTP prediction model and its training/evaluation pipeline. AIS preprocessing is maintained in [AIS-Process](https://github.com/KaysenWB/AIS-Process); the downstream route-planning experiments are documented in the paper.

<p align="center">
  <img src="./Figure01.jpg" width="100%" alt="Overview of the GMLTP framework from AIS processing to trajectory prediction and route planning">
</p>

<p align="center"><em>GMLTP connects AIS graph construction, long-term multi-vessel trajectory prediction, and real-time route planning.</em></p>

## Highlights

- **Interaction-aware spatial modeling.** MGSC separately learns four complementary graphs: social force, speed/course, virtual channel position, and vessel size.
- **Long-horizon temporal reasoning.** ProbSparse attention reduces the cost of learning global temporal dependencies and supports forecasts up to 96 steps in the reported experiments.
- **Real-world multi-vessel evaluation.** Experiments use one week of AIS observations from Victoria Harbour, Hong Kong, sampled into 50,000 encounter scenarios with a 6:2:2 train/validation/test split.
- **Consistent long-term gains.** GMLTP outperforms LSTM, GRU, BiLSTM, BiGRU, Seq2Seq, TCN, and Social-STGCNN on every reported metric from 24 through 96 prediction steps.
- **Route-planning relevance.** Long-term forecasts are evaluated with artificial potential field (APF), velocity obstacle (VO), and nonlinear model predictive control (NMPC) planners.

## Method

At every observed time step, GMLTP constructs four graphs and processes them with independent graph-convolution branches. Their representations are concatenated and passed to a ProbSparse Transformer encoder–decoder, which predicts the future positions of all vessels jointly.

| Component | What it captures |
| --- | --- |
| Social-force graph | Distance-based interaction between nearby vessels |
| Speed/course graph | Relative motion and potential collision interaction |
| Virtual-channel graph | Deviation from and interaction with the navigable channel |
| Vessel-size graph | Motion similarity associated with vessel dimensions |
| ProbSparse Transformer | Long-term global dependencies with sparse attention computation |

<details>
<summary><strong>View the detailed temporal prediction pipeline</strong></summary>

<p align="center">
  <img src="./Figure02.jpg" width="92%" alt="Detailed GMLTP ProbSparse Transformer encoder-decoder pipeline">
</p>

</details>

## Results at a glance

The table below reproduces the GMLTP values reported in the paper. Lower is better for all metrics; values follow the paper's preprocessing and normalization. Each reported result is averaged over 15 experiments.

| Prediction horizon | Physical horizon | MAE ↓ | MSE ↓ | ADE ↓ | FDE ↓ |
| ---: | ---: | ---: | ---: | ---: | ---: |
| 6 steps | 1 min | 0.0344 | 0.0021 | 0.0539 | 0.0673 |
| 12 steps | 2 min | 0.0470 | 0.0043 | 0.0740 | 0.1083 |
| 24 steps | 4 min | **0.0738** | **0.0121** | **0.1161** | **0.2091** |
| 48 steps | 8 min | **0.1395** | **0.0456** | **0.2199** | **0.4278** |
| 72 steps | 12 min | **0.2025** | **0.1007** | **0.3188** | **0.6264** |
| 96 steps | 16 min | **0.2678** | **0.1814** | **0.4239** | **0.7975** |
| **Average** | — | **0.1275** | **0.0577** | **0.2011** | **0.3727** |

Across all six horizons, GMLTP reduces the average **MAE by 14.8%**, **MSE by 20.9%**, **ADE by 13.8%**, and **FDE by 10.8%** relative to the next-best average score among the compared methods.

<p align="center">
  <img src="./Figure03.jpg" width="100%" alt="GMLTP qualitative multi-vessel trajectory prediction results from 6 to 96 steps">
</p>

<p align="center"><em>Qualitative forecasts in Victoria Harbour from 6 to 96 steps. Red: GMLTP prediction; black: ground truth.</em></p>

## Quick start

### 1. Clone the repository

```bash
git clone https://github.com/KaysenWB/OE_GMLTP.git
cd OE_GMLTP
```

### 2. Create an environment

The experiments in the paper were run with Python 3.9 and an NVIDIA GPU. CUDA is recommended for full training; the code automatically falls back to CPU when CUDA is unavailable.

```bash
conda create -n gmltp python=3.9 -y
conda activate gmltp

# Install the PyTorch build matching your CUDA version first:
# https://pytorch.org/get-started/locally/
pip install torch

# Remaining runtime dependencies
pip install "numpy<2" pandas scipy matplotlib torch-geometric==2.3.2
```

For platform-specific PyTorch Geometric wheels, see the [official installation guide](https://pytorch-geometric.readthedocs.io/en/latest/install/installation.html).

### 3. Prepare the AIS graph sequences

The processed AIS dataset is not distributed in this repository. Use [KaysenWB/AIS-Process](https://github.com/KaysenWB/AIS-Process) as the reference preprocessing pipeline, then place the processed pickle file at:

```text
data/AIS_processed_mgsc.cpkl
```

Each sample consumed by `Dataset_ship` contains:

```text
[trajectory, A_social_force, A_speed_course, A_channel, A_vessel_size]
```

- `trajectory`: `[time, vessels, features]`
- each adjacency sequence: `[time, vessels, vessels]`

The public loader standardizes each encounter independently and creates the 6:2:2 train/validation/test split.

### 4. Train and evaluate

The following command runs the six prediction horizons used in the paper: 6, 12, 24, 48, 72, and 96 steps.

```bash
python main_gmltp.py \
  --root_path ./data/ \
  --data_path AIS_processed_mgsc.cpkl \
  --gpu 0 \
  --itr 6 \
  --train_epochs 30
```

For a short pipeline check, use `--itr 1 --train_epochs 1`. Add `--do_predict True` to export prediction arrays after testing.

### 5. Find the outputs

```text
checkpoints/<setting>/checkpoint.pth   # best validation checkpoint
results/<setting>/metrics.npy         # [MAE, MSE, ADE, FDE]
results/<setting>/*prediction.npy      # optional prediction exports
```

## Repository structure

```text
OE_GMLTP/
├── main_gmltp.py          # experiment entry point and horizon configuration
├── models/
│   ├── model.py           # MGSC + ProbSparse Transformer model
│   ├── attn.py            # full and ProbSparse attention
│   ├── encoder.py         # Transformer encoder and distillation layers
│   ├── decoder.py         # Transformer decoder
│   └── embed.py           # value, positional, and temporal embeddings
├── data/
│   └── data_loader.py     # AIS graph-sequence loading and preprocessing
├── exp/
│   ├── exp_basic.py       # device and model setup
│   └── exp_gmltp.py       # training, validation, testing, and prediction
├── utils/                 # metrics, masks, time features, and training helpers
├── Figure01–03.jpg        # method and result figures
└── CITATION.cff           # machine-readable citation metadata
```

## Paper

**Dong Yang, Kaisen Yang, Yuxu Lu, Maohan Liang, and Congcong Zhao.** “Graph-driven multi-vessel long-term trajectories prediction for route planning under complex waters.” *Ocean Engineering*, 313:119511, 2024. [https://doi.org/10.1016/j.oceaneng.2024.119511](https://doi.org/10.1016/j.oceaneng.2024.119511)

## Citation

If you find this repository useful in your research, please consider citing the following paper:

```bibtex
@article{yang2024graph,
  title     = {Graph-driven multi-vessel long-term trajectories prediction for route planning under complex waters},
  author    = {Yang, Dong and Yang, Kaisen and Lu, Yuxu and Liang, Maohan and Zhao, Congcong},
  journal   = {Ocean Engineering},
  volume    = {313},
  pages     = {119511},
  year      = {2024},
  publisher = {Elsevier},
  doi       = {10.1016/j.oceaneng.2024.119511}
}
```

GitHub-compatible citation metadata are also available in [`CITATION.cff`](./CITATION.cff).

## Contact

If you have any questions or are interested in research collaboration, please contact **Kaisen Yang** at [kaisen.yang@connect.polyu.hk](mailto:kaisen.yang@connect.polyu.hk).

## Acknowledgement

The temporal modeling components in this work build on ideas and code from [Informer2020](https://github.com/zhouhaoyi/Informer2020). We sincerely thank the authors for their outstanding contribution to long-sequence time-series forecasting.

## License

This project is released under the [Apache License 2.0](./License).
