# CS598DLH-TPC Project

This is a reproduction project of paper 188 "Temporal Pointwise Convolutional Networks for Length of Stay Prediction in the Intensive Care Unit".

## Citation
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

## Link to the original repo
We mainly reused the existing codes in the authors' [repo](https://github.com/EmmaRocheteau/TPC-LoS-prediction).

## Environment preparation

### Python setup
Make sure to install python 3.6 for this project. Install all dependencies by 

	pip install -r requirements.txt

### Database setup
- You must have access to [eICU database](https://physionet.org/content/eicu-crd/2.0/) and [MIMIC-IV database](https://physionet.org/content/mimiciv/0.4/)

- You must also have the tool `postgres` installed to access the database.

### Pre-processing
#### eICU
Follow the **eICU** part of the **Pre-processing Instructions** section of the [original repo's](https://github.com/EmmaRocheteau/TPC-LoS-prediction) readme file. This will likely take you more than 15 hours to finish.

#### MIMIC-IV
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

## Running the models
### Training
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

### Evaluation
The best hyper-parameters are provided for part of the settings. We have changed the codes so that when running the test cases, the models choose the best hyper-parameter combination automatically.

To set the mode from training to testing, we just need to apply "--mode test". For example, when testing the TPC with the best hyper-parameters.
```
python3 -m models.run_tpc --mode test
```

We also tried all the aforementioned settings in the testing mode. All the commands we have run can be found in [*running_tpc.ipynb*](https://github.com/liuyuxiang512/CS598DLH-TPC/blob/main/running_tpc.ipynb), [*running_lstm.ipynb*](https://github.com/liuyuxiang512/CS598DLH-TPC/blob/main/running_tpc.ipynb), and [*running_transformer.ipynb*](https://github.com/liuyuxiang512/CS598DLH-TPC/blob/main/running_transformer.ipynb).

### Results
#### Claim 1:
#### Claim 2:
#### Claim 3:
#### Claim 4:
#### Ablation studies:
