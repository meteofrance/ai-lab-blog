---
layout: post
title: "Comparing neural network architectures with Py4cast & Titan"
date: 2024-10-21
categories: [Weather Forecasting, Machine Learning, Python]
author:
- Frank Guibert
- Léa Berthomier
---

# Experimental reports

We present here a report on our experiments to build a weather forecasting model with Deep Learning.

## Choice of neural network architecture

Our first task was to train and compare several neural network architectures and assess their differences.

Our goal in this first experiment campaign was to compare the performance of different neural network architectures under the same conditions.

### Dataset: TITAN

- Source **AROME Analyses**
- **Resolution**: 2.5km
- **Historical Data**: 2021-2023 (training: 2021-2022, testing: 2023)
- **Time Step**: 1 Hour
- **21 Weather Parameters**: Input and output of the model
  - **5 Surface Variables**: Temperature, humidity, wind (u & v), and precipitation
  - **4 Variables at 4 Vertical Levels**:
    - 850, 700, 500, 250 hPa
    - T, U, V, Z
- **4 Forcing Fields**: Model inputs
  - Cosine and sine of the time of day
  - Cosine and sine of the day of the year
- **Training Samples**: ~16,000 pairs (t0, t+1)

**Note**: Precipitation is the only parameter that is not an analysis. It is an AROME forecast made every hour, predicting the cumulative precipitation in mm for the next hour. In the future, we aim to use higher quality expertized radar data.

### Training Methodology

- **Training strategy**: Trained to make 1-hour forecasts, in scaled steps: y_pred = x + f(x) * step_diff_std + step_diff_mean
- **Cost Function**: Weighted Mean Squared Error (MSE)
- **Models**: [UNetR++](https://github.com/meteofrance/py4cast/blob/6fd15e707aefb5747bfb9bc13719d16923e022ac/py4cast/models/vision/unetrpp.py#L580), [SwinUNetR](https://github.com/meteofrance/py4cast/blob/6fd15e707aefb5747bfb9bc13719d16923e022ac/py4cast/models/vision/transformers.py#L370), [Hilam](https://github.com/meteofrance/py4cast/blob/6fd15e707aefb5747bfb9bc13719d16923e022ac/py4cast/models/nlam/models.py#L730)
- **Learning Rate Scheduler**: TODO


### Hardware

- **Training**: Conducted on 1 node of 4 Nvidia V100 32GB GPUs
- **Training Duration**: 2 to 15 days depending on the models
- **Inference Time**: Less than one minute on CPU for +12h forecasts


### Results

Both in metrics and forecast case studies, the UNetR++ model, modified and optimized, proved to be the best among the tested models.

The HiLAM model, on the other hand, was costly to train, with lower scores and lower resolution forecasts.

Here we present some forecast animations of these two models, compared to AROME and AROME analyses (our "ground truth" here) on a case study for 5 surface parameters.

On 2023/06/18 at 12h UTC, a warm and unstable southwesterly flow favored the development of strong thunderstorms over parts of France on June 18. The north of the country was swept by wind gusts between 90 and 110 km/h. Some supercells formed over the center and then the southwest of the country, sometimes generating heavy hail.

**INSERT GIFS**

**Tableau : temps training, commentaires (config + lien vers fichier), prévisions lisses, loss finale sur jeu de test, RAM utilisée, batch size**

**Ajouter citation dans readme principal**

| Model  | Configuration | Training Time | Batch Size  | Used RAM | Final loss on test set | Comments |
| :---:   | :---: | :---: | :---: | :---: | :---: | :---: |
| HiLAM | details + config file | 5 days | Ground |  Cumulated Rainfall on next 1h |
| HiLAM | 128 details + config file |8 days | 10m |  U, V |
| SwinUNetR | ARPEGE  | EURAT01 (0.1°) | 2m |  T, HU |
| UNetR++ | ARPEGE | EURAT01 (0.1°) | 24 Isobaric levels |  Z, T, U, V, HU |
| UNetR++ | ARPEGE | EURAT01 (0.1°) | Sea |  P |
| UNetR++ | AROME | EURW1S100 (1.3 km) | 10m |  U, V |


## Analysis & Perspectives

- **Initial experiments** show that it is possible to train a neural network model to provide forecasts at the scale of France.
- The obtained models provide consistent forecasts for the studied cases but become extravagant beyond a 12-hour lead time.
- These initial experiments highlight the importance of not being confined to assumptions derived from physical models and exploring all the possibilities offered by neural networks. We see here that interesting forecasts can already be obtained with a model trained on few parameters and vertical levels, with little historical data. Similarly, Graph Neural Networks, which were thought to be suitable for weather forecasting, are costly and (in our case) less effective than Vision Transformers.
- The **py4cast development framework** proves useful for quickly conducting experiment campaigns and testing new ideas.
- **Future priorities** include:
  - Testing other architectures: GraphCast, Pangu, others (in collaboration with Eviden)
  - Expanding the Titan dataset: temporal depth, other parameters...
  - Testing the influence of the number of time steps in input and output, adding a large-scale forcing model in input, testing variability between two trainings...
  - Testing the influence of the duration of a time step (like Panguweather) on the quality of forecasts depending on the lead time (24H, 48H, ...)
  - Conducting optimization work on models and training strategies to remain as resource-efficient as possible