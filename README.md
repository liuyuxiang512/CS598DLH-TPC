# CS598DLH-TPC Project

This is a reproduction project of paper 188 "Temporal Pointwise Convolutional Networks for Length of Stay Prediction in the Intensive Care Unit".

## 1 Citation
```
@inproceedings{rocheteau2021,
author = {Rocheteau, Emma and Li\`{o}, Pietro and Hyland, Stephanie},
title = {Temporal Pointwise Convolutional Networks for Length of Stay Prediction in the Intensive Care Unit},
year = {2021},
isbn = {9781450383592},
publisher = {Association for Computing Machinery},
address = {New York, NY, USA},
url = {https://doi.org/10.1145/3450439.3451860},
doi = {10.1145/3450439.3451860},
booktitle = {Proceedings of the Conference on Health, Inference, and Learning},
pages = {58–68},
numpages = {11},
keywords = {intensive care unit, length of stay, temporal convolution, mortality, patient outcome prediction},
location = {Virtual Event, USA},
series = {CHIL '21}
}
```

## 2 Link to the original repo
We mainly reused the existing codes in the authors' [repo](https://github.com/EmmaRocheteau/TPC-LoS-prediction).

## 3 Environment preparation

### 3.1 Python setup
Make sure to install python 3.6 for this project. Install all dependencies by 

	pip install -r requirements.txt

### 3.2 Database setup
- You must have access to [eICU database](https://physionet.org/content/eicu-crd/2.0/) and [MIMIC-IV database](https://physionet.org/content/mimiciv/0.4/)

- You must also have the tool `postgres` installed to access the database.

### 3.3 Pre-processing
#### 3.3.1 eICU
Follow the **eICU** part of the **Pre-processing Instructions** section of the [original repo's](https://github.com/EmmaRocheteau/TPC-LoS-prediction) readme file. This will likely take you more than 15 hours to finish.

#### 3.3.2 MIMIC-IV
The initial steps of the original repo's MIMIC guide was confusing, you should follow this guide first.

Follow the installation guide on this [repo](https://github.com/EmmaRocheteau/MIMIC-IV-Postgres) to set up the project. Notice that the `dbname` should be the same as the one you use later, here the default value is `mimic4`, which is different than the original guide.

Now, start from the 3rd step of the **MIMIC-IV** part of the **Pre-processing Instructions** section of the [original repo](https://github.com/EmmaRocheteau/TPC-LoS-prediction), follow the whole guide until the script

	\i MIMIC_preprocessing/create_all_tables.sql

Before running this command, you need to include the necessary extension by executing

	CREATE EXTENSION tablefunc;

Then follow the rest of the original guide to finish preprocessing.

Now we need to solve the batch loading bug. You can use a data loader to drop the last batch with a parameter, but in this project we decided to drop it manually. Navigate to the MIMIC generated data folder, try dropping the last records of generated csv files by

	sed -i '$ d' {target file name}.csv

The whole pre processing task will will likely take you 0.5 to 2 days depending on your hardware and will require at least 100 GB of free disk space. 

## 4 Running the models
### 4.1 Training
To train the proposed TPC model in default setting:
```
python3 -m models.run_tpc
```

To train the baseline LSTM (CW LSTM) in default setting:
```
python3 -m models.run_lstm (-channelwise)
```

To train the baseline Transformer in default setting:
```
python3 -m models.run_transformer
```

Then you can get logs of training and validation results.

To reproduce the main experiments, we also tried different setting with "--dataset MIMIC" for all models; "--loss mse", "--task mortality", "--task multitask", "--model_type pointwise_only", "--model_type temp_only", "--model_type temp_only -share_weights", "-no_skip_connections" for the proposed TPC model.

### 4.2 Evaluation
The best hyper-parameters are provided for part of the settings. We have changed the codes so that when running the test cases, the models choose the best hyper-parameter combination automatically.

To set the mode from training to testing, we just need to apply "--mode test". For example, when testing the TPC with the best hyper-parameters.
```
python3 -m models.run_tpc --mode test
```

We also tried all the aforementioned settings in the testing mode. All the commands we have run can be found in [*running_tpc.ipynb*](https://github.com/liuyuxiang512/CS598DLH-TPC/blob/main/running_tpc.ipynb), [*running_lstm.ipynb*](https://github.com/liuyuxiang512/CS598DLH-TPC/blob/main/running_tpc.ipynb), and [*running_transformer.ipynb*](https://github.com/liuyuxiang512/CS598DLH-TPC/blob/main/running_transformer.ipynb).

### 4.3 Results
Following the original paper, we consider six metrics: MAD, MAPE, MSE, MSLE, R<sup>2</sup>, and Kappa. For the first four, lower is better. For the last two, higher is better.

#### 4.3.1 Claim 1: Comparisons with Baselines on eICU and MIMIC-IV
##### eICU
Model | MAD | MAPE | MSE | MSLE | R<sup>2</sup> | Kappa
--- | --- | --- | --- | --- | --- | ---
Mean | 3.21 | 395.7 | 29.5 | 2.87 | 0 | 0
Median | 2.76 | 184.4 | 32.6 | 2.15 | -0.11 | 0
APACHE-IV | 2.54 | 182.1 | 16.6 | 1.1 | -0.01 | 0.2
LSTM | 2.63 | 126.63 | 30.77 | 1.45 | 0.12 | 0.32
Transformer | 2.57 | 121.89 | 30.2 | 1.41 | 0.13 | 0.34
TPC | 1.43 | 35.2 | 18.61 | 0.31 | 0.46 | 0.77
##### MIMIC-IV
Model | MAD | MAPE | MSE | MSLE | R<sup>2</sup> | Kappa
--- | --- | --- | --- | --- | --- | ---
Mean | 5.24 | 474.9 | 77.7 | 2.8 | 0 | 0
Median | 4.6 | 216.8 | 86.8 | 2.09 | -0.12 | 0
LSTM | 3.67 | 106.17 | 66.07 | 1.26 | 0.15 | 0.43
CW LSTM | 3.68 | 108.79 | 66.52 | 1.22 | 0.14 | 0.43
Transformer | 3.63 | 115.92 | 63.93 | 1.21 | 0.18 | 0.44
TPC | 2.23 | 30.43 | 41.28 | 0.18 | 0.47 | 0.85

#### 4.3.2 Claim 2: Comparison of Two Loss Functions
Model | MAD | MAPE | MSE | MSLE | R<sup>2</sup> | Kappa
--- | --- | --- | --- | --- | --- | ---
TPC(MSLE) | 1.43 | 35.2 | 18.61 | 0.31 | 0.46 | 0.77
TPC(MSE) | 2.04 | 119.28 | 19.34 | 1.68 | 0.44 | 0.62

#### 4.3.3 Claim 3 & 4: Comparison of LoS / Mortality in Singl-task and Multi-task Settings
![results of claim 3 & 4](https://github.com/liuyuxiang512/CS598DLH-TPC/blob/main/claim34_results.PNG)

#### 4.3.4 Ablation studies:
Model         | MAD  | MAPE   | MSE   | MSLE | R<sup>2</sup> | Kappa 
--- | --- | --- | --- | --- | --- | ---
TPC           | 1.43 | 35.2   | 18.61 | 0.31 | 0.46  | 0.77  
Point. only   | 3.28 | 172.24 | 43.7  | 1.59 | -0.24 | 0.45  
Temp. only    | 1.57 | 47.68  | 18.89 | 0.44 | 0.46  | 0.74  
WS            | 2.51 | 128.58 | 28.44 | 1.36 | 0.18  | 0.38  
TPC (no skip) | 1.63 | 51.51  | 19.68 | 0.51 | 0.43  | 0.72  
