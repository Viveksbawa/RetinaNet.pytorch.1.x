# Surgeon Action Detection for endoscopic images/videos
### This is a baseline model developed for SARAS-ESAD 2020 challenge. To download the dataset and participate in the challenge, please register at [SARAS-ESAD website](https://saras-esad.grand-challenge.org).

The code is adopated from [RetinaNet implementation in pytorch.1.x](https://github.com/gurkirt/RetinaNet.pytorch.1.x).

## Features of this baseline

- Data preparation instructions for SARAS-ESAD 2020 challenge
- Dataloader for SARAS-ESAD dataset
- Pytorch1.X implementation
- Feature pyramid network (FPN) archtecture with different ResNet backbones
- Three types of loss functions i.e OHEM Loss, Focal Loss, and YOLO Loss on top of FPN

## Introduction

Here, we implement basic data-handling tools for [SARAS-ESAD](https://saras-esad.grand-challenge.org/Dataset/) dataset with FPN training process. We implement a pure pytorch code for train FPN with [Focal-Loss](https://arxiv.org/pdf/1708.02002.pdf) or [OHEM/multi-box-loss](https://arxiv.org/pdf/1512.02325.pdf) paper. Aim of this repository try different loss functions and make a fair comparison in terms of performance on SARAR-ESAD dataset.

We hope this will help kick start more teams to get upto the speed and allow the time for more invoative solutions. We want to eleminate the pain of building data handling and training process from scratch. Our final aim is to get this repository the level of [realtime-action-detection](https://github.com/gurkirt/realtime-action-detection).

At the moment we support latest pytorch and ubuntu with Anaconda distribution of python. Tested on a single machine with 2/4/8 GPUs.

You can found out about architecture and loss function on parent reportory, i.e. [RetinaNet implementation in pytorch.1.x](https://github.com/gurkirt/RetinaNet.pytorch.1.x).

ResNet is used as a backbone network (a) to build the pyramid features (b). 
Each classification (c) and regression (d) subnet is made of 4 convolutional layers and finally a convolutional layer to predict the class scores and bounding box coordinated respectively.

Similar to the original paper, we freeze the batch normalisation layers of ResNet based backbone networks. Also, few initial layers are also frozen, see `fbn` flag in training arguments. 

## Loss functions 
- OHEM with multi-box loss function: We use multi-box loss function with online hard example mining (OHEM), similar to [SSD](https://arxiv.org/pdf/1512.02325.pdf). A huge thanks to Max DeGroot, Ellis Brown for [Pytorch implementation](https://github.com/amdegroot/ssd.pytorch) of SSD and loss function.

- Focal loss: Same as in the original paper we use sigmoid focal loss, see [RetnaNet](https://arxiv.org/pdf/1708.02002.pdf). We use pure pytorch implementation of it.

- Yolo Loss: Multi-part loss function from [YOLO](https://pjreddie.com/darknet/yolo/) is also implemented here.

## Installation
You will need the following to run this code successfully
- Anaconda python
- Pytorch latest
- Visulisation 
  - if you want to visulise set tensorboard flag equal to true while training
  - [TensorboadX](https://github.com/lanpa/tensorboardX)
  - Tensorflow for tensorboad


### Datasets and other downloads
- Please visit [SARAS-ESAD](https://saras-esad.grand-challenge.org) website to download the dataset for surgeon action detection. 
- Extract all the sets (train and val) from zip files and put them under single directory. Provide the path of that directory as data_root in train file. Data prepocessing and feeding pipeline is in [detectionDatasets.py](https://github.com/Viveksbawa/SARAS-ESAD-baseline/blob/master/data/detectionDatasets.py) file.
- rename the data dirctory `esad`. 
- Your directory will look like
  - esad
    - train
      - files
      - ..
    - val
      - files
      - ..

- Now your dataset is ready, that is time to download imagenet pretrained weights for ResNet backbone models. 
- Weights are initialised with imagenet pretrained models, specify the path of pre-saved models, `model_dir` in `train.py`. Download them from [torchvision models](https://github.com/pytorch/vision/blob/master/torchvision/models/resnet.py). After you have download weights, please rename then aprropiratley under `model_dir` e.g. resnet50 resen101 etc. from This is a requirement of training process. 

## TRAINING

Once you have pre-processed the dataset, then you are ready to train your networks.
We must have following arguments set correctly:
- `data_root` is base path upto `esad` dirotory e.g. `\home\gurkirt\`
- `save_root` is base path where you want to store the checkpoints, training logs, tensoboard logs etc.
- `model_dir` is path where ResNet backbone model weights are stored

To train run the following command. 

```
python train.py --loss_type=mbox --data_root=\home\gurkirt\ --tensoboard=true
```

It will use all the visible GPUs. 
You can append `CUDA_VISIBLE_DEVICES=<gpuids-comma-separated>` at the beginning of the above command to mask certain GPUs. We used 2 GPU machine to run our experiments.

Please check the arguments in `train.py` to adjust the training process to your liking.

### Some usefull flags
- 

## Evaluation
Model is evaluated and saved after each `1000` iterations. 

mAP@0.25 is computed after every `500` iterations and at the end. You can chnage to your liking by specify it in `train.py` arguments.

You can evaluate and save the results in json file using `evaluate.py`. It follow the same aruments `train.py`.
By default it evaluate using the model store at `max_iters`, but you can chnage it any other snapshot/checkpoint.

### AT THE MOMENT it under devlopment
### TODO
- reorgnise validate function to accpt multiple iou thresholds
- write submission file


```
python evaluate.py --loss_type=focal
```


## Results
Here are the results on `esad` dataset.

Loss   |depth | width | height | AP_25    | AP_50   | AP_75    | AP_MEAN  |   
|----- |----- |:----: |:------:| :------: | :------:| :------: | :------: |
| Focal| 50   |  1024 | 576    | ----     | ----    | ----     | ----     |  
| OHEM | 50   |  1024 | 576    | ----     | ----    | ---- | **.9** |

## Details
- Input image size is `600x1024`.
- Batch size is set to `16`, the learning rate of `0.01`.
- Weights for initial layers are frozen see `freezeupto` flag in `train.py`

