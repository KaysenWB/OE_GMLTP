# OE-GMLTP
Ship trajectory prediction in long-term multi-ship scenarios. Implemented by Multi-Graph Convolutional Network (MGSC) and probabilistic sparse self-attention mechanism (Probsparse attention).
This code is for the paper "Graph-driven Multi-vessel Long-term Trajectories Prediction for Route Planning under Complex Waters".

The input is trajectory data with the shape [length, batch, feature], and a corresponding adjacency matrix with the shape [length, nodes, nodes], which can be referred to the following code for data processing: https://github.com/KaysenWB/AIS-Process. Please note that setting the parameter MGSC to "True"

The main execution file is `main_gmltp.py`. The components of several models are located in the `Models` folder. Dataset creation and loading are handled in the `data` directory. Model training, testing, and usage are implemented in the `exp` folder. The `utils` directory contains various utilities, including time encoding and metric calculations.


## Environment Setup

**System Requirements**

- Operating System: Linux (Ubuntu 18.04+ recommended)
- Python 3.8 or higher
- CUDA 11.3+ (for GPU acceleration, optional)

**Dependencies**

- torch==2.8.0
- numpy==2.0.1
- pandas==2.3.3
- math==1.3.0
- pytorch_tcn==1.2.3
- matplotlib==3.7.2
- torch-geometric==2.3.2


## Algorithm Structure

<div align="center">
<img src="https://github.com/KaysenWB/OE-GMLTP/blob/main/Figure01.jpg?raw=true" width="90%" height="90%">
</div>

General overview figure of the paper. It includes data processing, trajectory prediction, and route planning.

Our proposed algorithmic flow for GMLTP on time series prediction. The key point is to perform Q-matrix sparsification before self-attention. We measure the effectiveness of each qi in the attention computation based on the gap (KL scatter) between the data distribution of the dot product pairs of qi over the K matrix and the uniform distribution. The more effective qi vectors are then filtered proportionally to the length of the trajectory input to form a sparse matrix of Q. The sparse computation is ultimately used to reduce the complexity of self-attention and achieve longer-term prediction.

## Results

<div align="center">
<img src="https://github.com/KaysenWB/OE-GMLTP/blob/main/Figure03.jpg?raw=true" width="90%" height="90%">
</div>

Presentation of prediction results, which are based on one month's AIS data for Victoria, Hong Kong.


## <span id="citelink">Citation</span>
If you find this repository useful in your research, please consider citing the following papers:
```
@article{yang2024graph,
  title={Graph-driven multi-vessel long-term trajectories prediction for route planning under complex waters},
  author={Yang, Dong and Yang, Kaisen and Lu, Yuxu and Liang, Maohan and Zhao, Congcong},
  journal={Ocean Engineering},
  volume={313},
  pages={119511},
  year={2024},
  publisher={Elsevier}
}
```

## Contact
If you have any questions, feel free to contact Haoyi Zhou through Email (kaisen.yang@connect.polyu.hk). Pull requests are highly welcomed!


## Acknowledgement
The algorithm in this work references a lot of the following work: https://github.com/zhouhaoyi/Informer2020.git.
Their outstanding contributions are greatly appreciated.
