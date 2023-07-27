---
description: Deploy YOLOv8 on NVIDIA Jetson using TensorRT
title: Deploy YOLOv8 on NVIDIA Jetson using TensorRT
keywords:
  - Edge
  - reComputer Application
image: https://files.seeedstudio.com/wiki/wiki-platform/S-tempor.png
slug: /YOLOv8-TRT-Jetson
last_update:
  date: 07/17/2023
  author: Lakshantha
---

# Deploy YOLOv8 on NVIDIA Jetson using TensorRT

This wiki guide explains how to deploy a YOLOv8 model into NVIDIA Jetson Platform and perform inference using TensorRT. Here we use TensorRT to maximize the inference performance on the Jetson platform.

Different computer vision tasks will be introduced here such as:

- Object Detection
- Image Segmentation
- Image Classification
- Pose Estimation
- Object Tracking

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/YOLOV8-TRT/8.gif
" style={{width:1000, height:'auto'}}/></div>

## Prerequisites

- Ubuntu Host PC (native or VM using VMware Workstation Player)
- [reComputer Jetson](https://www.seeedstudio.com/reComputer-J4012-p-5586.html) or any other NVIDIA Jetson device running JetPack 5.1.1 or higher

:::note
This wiki has been tested and verified on a [reComputer J4012](https://www.seeedstudio.com/reComputer-J4012-p-5586.html) powered by NVIDIA Jetson orin NX 16GB module 
:::

## Flash JetPack to Jetson

Now you need to make sure that the Jetson device is flashed with a [JetPack](https://developer.nvidia.com/embedded/jetpack) system including SDK components such as CUDA, TensorRT, cuDNN and more. You can either use NVIDIA SDK Manager or command-line to flash JetPack to the device.

For Seeed Jetson-powered devices flashing guides, please refer to the below links:
- [reComputer J1010 | J101](https://wiki.seeedstudio.com/reComputer_J1010_J101_Flash_Jetpack)
- [reComputer J2021 | J202](https://wiki.seeedstudio.com/reComputer_J2021_J202_Flash_Jetpack)
- [reComputer J1020 | A206](https://wiki.seeedstudio.com/reComputer_J1020_A206_Flash_JetPack)
- [reComputer J4012 | J401](https://wiki.seeedstudio.com/reComputer_J4012_Flash_Jetpack)
- [A203 Carrier Board](https://wiki.seeedstudio.com/reComputer_A203_Flash_System)
- [A205 Carrier Board](https://wiki.seeedstudio.com/reComputer_A205_Flash_System)
- [Jetson Xavier AGX H01 Kit](https://wiki.seeedstudio.com/Jetson_Xavier_AGX_H01_Driver_Installation)
- [Jetson AGX Orin 32GB H01 Kit](https://wiki.seeedstudio.com/Jetson_AGX_Orin_32GB_H01_Flash_Jetpack)

:::note
Make sure to Flash JetPack version 5.1.1 because that is the version we have verified for this wiki 
:::

## Install Ultralytics Package

- **Step 1.** Access the terminal of Jetson device, install pip and upgrade it

```sh
sudo apt update
sudo apt install -y python3-pip -y
pip3 install --upgrade pip
```

- **Step 2.**  Install Ultralytics package

```sh
pip3 install ultralytics
```

- **Step 3.**  Upgrade numpy version to latest

```sh
pip3 install numpy -U
```

- **Step 4.** Reboot the device

```sh
sudo reboot
```

## Uninstall Torch and Torchvision

The above ultralytics installation will install Torch and Torchvision. However, these 2 packages installed via pip are not compatible to run on Jetson platform wwhich is based on **ARM aarch64 architecture**. Therefore we need to manually install pre-built PyTorch pip wheel and compile/ install Torchvision from source.

```sh
pip3 uninstall torch torchvision
```

## Install PyTorch and Torchvision

Visit [this page](https://forums.developer.nvidia.com/t/pytorch-for-jetson) to access all the PyTorch and Torchvision links.

Here are some of the versions supported by JetPack 5.0 and above.

**PyTorch v2.0.0**

Supported by JetPack 5.1 (L4T R35.2.1) / JetPack 5.1.1 (L4T R35.3.1) with Python 3.8

**file_name:** torch-2.0.0+nv23.05-cp38-cp38-linux_aarch64.whl
**URL:** https://nvidia.box.com/shared/static/i8pukc49h3lhak4kkn67tg9j4goqm0m7.whl

**PyTorch v1.13.0**

Supported by JetPack 5.0 (L4T R34.1) / JetPack 5.0.2 (L4T R35.1) / JetPack 5.1 (L4T R35.2.1) / JetPack 5.1.1 (L4T R35.3.1) with Python 3.8

**file_name:** torch-1.13.0a0+d0d6b1f2.nv22.10-cp38-cp38-linux_aarch64.whl
**URL:** https://developer.download.nvidia.com/compute/redist/jp/v502/pytorch/torch-1.13.0a0+d0d6b1f2.nv22.10-cp38-cp38-linux_aarch64.whl

- **Step 1.** Install torch according to your JetPack version in the following format
pip3 
```sh
wget <URL> -O <file_name>
pip3 install <file_name>
```

For example, here we are running **JP5.1.1** and therefore we choose **PyTorch v2.0.0**

```sh
sudo apt-get install -y libopenblas-base libopenmpi-dev
wget https://nvidia.box.com/shared/static/i8pukc49h3lhak4kkn67tg9j4goqm0m7.whl -O torch-2.0.0+nv23.05-cp38-cp38-linux_aarch64.whl
pip3 install torch-2.0.0+nv23.05-cp38-cp38-linux_aarch64.whl
```

- **Step 2.** Install torchvision depending on the version of PyTorch that you have installed. For example, we chose PyTorch v2.0.0, which means, we need to choose Torchvision v0.15.2

```sh
sudo apt install -y libjpeg-dev zlib1g-dev
git clone https://github.com/pytorch/vision torchvision
cd torchvision
git checkout v0.15.2
python3 setup.py install --user
```

Here is a list of the corresponding torchvision version that you need to install according to the PyTorch version:

- PyTorch v2.0.0 - torchvision v0.15
- PyTorch v1.13.0 - torchvision v0.14

If you want a more detailed list, please check [this link](https://github.com/pytorch/vision).

## Use Pre-trained models

The fastest way to get started with YOLOv8 is to use pre-trained models provided by YOLOv8. However, these are PyTorch models and therefore will only utilize the CPU when inferencing on the Jetson. If you want the best performance of these models on the Jetson while running on the GPU, you can export the PyTorch models to TensorRT by following this section of the wiki.

detection - helmet
classification - panda
segmentation - traffic
pose - exercise
tracking - people


### Object Detection

YOLOv8 offers 5 pre-trained PyTorch model weights for object detection, trained on COCO dataset at input image size 640x640. You can find them below

<table>
  <thead>
    <tr>
      <th>Model</th>
      <th>size<br />(pixels)</th>
      <th>mAPval<br />50-95</th>
      <th>Speed<br />CPU ONNX<br />(ms)</th>
      <th>Speed<br />A100 TensorRT<br />(ms)</th>
      <th>params<br />(M)</th>
      <th>FLOPs<br />(B)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><a href="https://github.com/ultralytics/assets/releases/download/v0.0.0/yolov8n.pt" target="_blank" rel="noopener noreferrer">YOLOv8n</a></td>
      <td>640</td>
      <td>37.3</td>
      <td>80.4</td>
      <td>0.99</td>
      <td>3.2</td>
      <td>8.7</td>
    </tr>
    <tr>
      <td><a href="https://github.com/ultralytics/assets/releases/download/v0.0.0/yolov8s.pt" target="_blank" rel="noopener noreferrer">YOLOv8s</a></td>
      <td>640</td>
      <td>44.9</td>
      <td>128.4</td>
      <td>1.20</td>
      <td>11.2</td>
      <td>28.6</td>
    </tr>
    <tr>
      <td><a href="https://github.com/ultralytics/assets/releases/download/v0.0.0/yolov8m.pt" target="_blank" rel="noopener noreferrer">YOLOv8m</a></td>
      <td>640</td>
      <td>50.2</td>
      <td>234.7</td>
      <td>1.83</td>
      <td>25.9</td>
      <td>78.9</td>
    </tr>
    <tr>
      <td><a href="https://github.com/ultralytics/assets/releases/download/v0.0.0/yolov8l.pt" target="_blank" rel="noopener noreferrer">YOLOv8l</a></td>
      <td>640</td>
      <td>52.9</td>
      <td>375.2</td>
      <td>2.39</td>
      <td>43.7</td>
      <td>165.2</td>
    </tr>
    <tr>
      <td><a href="https://github.com/ultralytics/assets/releases/download/v0.0.0/yolov8x.pt" target="_blank" rel="noopener noreferrer">YOLOv8x</a></td>
      <td>640</td>
      <td>53.9</td>
      <td>479.1</td>
      <td>3.53</td>
      <td>68.2</td>
      <td>257.8</td>
    </tr>
  </tbody>
</table>


Reference: https://docs.ultralytics.com/tasks/detect

You can choose and download your desired model from the above table and execute the below command to run inference on an image

```sh
yolo detect predict model=yolov8n.pt source='https://ultralytics.com/images/bus.jpg' show=True
```

Here for model, you can change to either yolov8s.pt, yolov8m.pt, yolov8l.pt, yolov8x.pt and it will download the relavant pre-trained model

You can also connect a webcam and execute the below command 

```sh
yolo detect predict model=yolov8n.pt source='0' show=True
```

:::note
If you face any errors when executing the above commands, try adding "device=0" at the end of the command
:::

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/YOLOV8-TRT/2.gif
" style={{width:1000, height:'auto'}}/></div>

### Image Classification

YOLOv8 offers 5 pre-trained PyTorch model weights for image classification, trained on ImageNet at input image size 224x224. You can find them below

<table>
  <thead>
    <tr>
      <th>Model</th>
      <th>size<br />(pixels)</th>
      <th>acc<br />top1</th>
      <th>acc<br />top5<br /></th>
      <th>Speed<br />CPU ONNX<br />(ms)<br /></th>
      <th>Speed<br />A100 TensorRT<br />(ms)<br /><br /></th>
      <th>params<br />(M)<br /></th>
      <th>FLOPs<br />(B) at 640</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>YOLOv8n-cls</td>
      <td>224</td>
      <td>66.6</td>
      <td>87.0</td>
      <td>12.9</td>
      <td>0.31</td>
      <td>2.7</td>
      <td>4.3</td>
    </tr>
    <tr>
      <td>YOLOv8s-cls</td>
      <td>224</td>
      <td>72.3</td>
      <td>91.1</td>
      <td>23.4</td>
      <td>0.35</td>
      <td>6.4</td>
      <td>13.5</td>
    </tr>
    <tr>
      <td>YOLOv8m-cls</td>
      <td>224</td>
      <td>76.4</td>
      <td>93.2</td>
      <td>85.4</td>
      <td>0.62</td>
      <td>17.0</td>
      <td>42.7</td>
    </tr>
    <tr>
      <td>YOLOv8l-cls</td>
      <td>224</td>
      <td>78.0</td>
      <td>94.1</td>
      <td>163.0</td>
      <td>0.87</td>
      <td>37.5</td>
      <td>99.7</td>
    </tr>
    <tr>
      <td> YOLOv8x-cls</td>
      <td>224</td>
      <td>78.4</td>
      <td>94.3</td>
      <td>232.0</td>
      <td>1.01</td>
      <td>57.4</td>
      <td>154.8</td>
    </tr>
  </tbody>
</table>

Reference: https://docs.ultralytics.com/tasks/classify

You can choose your desired model and execute the below command to run inference on an image

```sh
yolo classify predict model=yolov8n-cls.pt source='https://ultralytics.com/images/bus.jpg' show=True
```

Here for model, you can change to either yolov8s-cls.pt, yolov8m-cls.pt, yolov8l-cls.pt, yolov8x-cls.pt and it will download the relavant pre-trained model

You can also connect a webcam and execute the below command 

```sh
yolo classify predict model=yolov8n-cls.pt source='0' show=True
```

:::note
If you face any errors when executing the above commands, try adding "device=0" at the end of the command
:::

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/YOLOV8-TRT/5.gif
" style={{width:1000, height:'auto'}}/></div>

### Image Segmentation

YOLOv8 offers 5 pre-trained PyTorch model weights for image segmentation, trained on COCO dataset at input image size 640x640. You can find them below

<table>
  <thead>
    <tr>
      <th>Model</th>
      <th>size<br />(pixels)</th>
      <th>mAPbox<br />50-95</th>
      <th>mAPmask<br />50-95</th>
      <th>Speed<br />CPU ONNX<br />(ms)</th>
      <th>Speed<br />A100 TensorRT<br />(ms)</th>
      <th>params<br />(M)</th>
      <th>FLOPs<br />(B)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><a href="https://github.com/ultralytics/assets/releases/download/v0.0.0/yolov8n-seg.pt">YOLOv8n-seg</a></td>
      <td>640</td>
      <td>36.7</td>
      <td>30.5</td>
      <td>96.1</td>
      <td>1.21</td>
      <td>3.4</td>
      <td>12.6</td>
    </tr>
    <tr>
      <td><a href="https://github.com/ultralytics/assets/releases/download/v0.0.0/yolov8s-seg.pt">YOLOv8s-seg</a></td>
      <td>640</td>
      <td>44.6</td>
      <td>36.8</td>
      <td>155.7</td>
      <td>1.47</td>
      <td>11.8</td>
      <td>42.6</td>
    </tr>
    <tr>
      <td><a href="https://github.com/ultralytics/assets/releases/download/v0.0.0/yolov8m-seg.pt">YOLOv8m-seg</a></td>
      <td>640</td>
      <td>49.9</td>
      <td>40.8</td>
      <td>317.0</td>
      <td>2.18</td>
      <td>27.3</td>
      <td>110.2</td>
    </tr>
    <tr>
      <td><a href="https://github.com/ultralytics/assets/releases/download/v0.0.0/yolov8l-seg.pt">YOLOv8l-seg</a></td>
      <td>640</td>
      <td>52.3</td>
      <td>42.6</td>
      <td>572.4</td>
      <td>2.79</td>
      <td>46.0</td>
      <td>220.5</td>
    </tr>
    <tr>
      <td><a href="https://github.com/ultralytics/assets/releases/download/v0.0.0/yolov8x-seg.pt">YOLOv8x-seg</a></td>
      <td>640</td>
      <td>53.4</td>
      <td>43.4</td>
      <td>712.1</td>
      <td>4.02</td>
      <td>71.8</td>
      <td>344.1</td>
    </tr>
  </tbody>
</table>

Reference: https://docs.ultralytics.com/tasks/segment

You can choose your desired model and execute the below command to run inference on an image

```sh
yolo segment predict model=yolov8n-seg.pt source='https://ultralytics.com/images/bus.jpg' show=True
```

Here for model, you can change to either yolov8s-seg.pt, yolov8m-seg.pt, yolov8l-seg.pt, yolov8x-seg.pt and it will download the relavant pre-trained model

You can also connect a webcam and execute the below command 

```sh
yolo segment predict model=yolov8n-seg.pt source='0' show=True
```

:::note
If you face any errors when executing the above commands, try adding "device=0" at the end of the command
:::

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/YOLOV8-TRT/3.gif
" style={{width:1000, height:'auto'}}/></div>

### Pose Estimation

YOLOv8 offers 6 pre-trained PyTorch model weights for pose estimation, trained on COCO keypoints dataset at input image size 640x640. You can find them below

<table>
  <thead>
    <tr>
      <th>Model</th>
      <th>size<br />(pixels)</th>
      <th>mAPpose<br />50-95</th>
      <th>mAPpose<br />50</th>
      <th>Speed<br />CPU ONNX<br />(ms)</th>
      <th>Speed<br />A100 TensorRT<br />(ms)</th>
      <th>params<br />(M)</th>
      <th>FLOPs<br />(B)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><a href="https://github.com/ultralytics/assets/releases/download/v0.0.0/yolov8n-pose.pt">YOLOv8n-pose</a></td>
      <td>640</td>
      <td>50.4</td>
      <td>80.1</td>
      <td>131.8</td>
      <td>1.18</td>
      <td>3.3</td>
      <td>9.2</td>
    </tr>
    <tr>
      <td><a href="https://github.com/ultralytics/assets/releases/download/v0.0.0/yolov8s-pose.pt">YOLOv8s-pose</a></td>
      <td>640</td>
      <td>60.0</td>
      <td>86.2</td>
      <td>233.2</td>
      <td>1.42</td>
      <td>11.6</td>
      <td>30.2</td>
    </tr>
    <tr>
      <td><a href="https://github.com/ultralytics/assets/releases/download/v0.0.0/yolov8m-pose.pt">YOLOv8m-pose</a></td>
      <td>640</td>
      <td>65.0</td>
      <td>88.8</td>
      <td>456.3</td>
      <td>2.00</td>
      <td>26.4</td>
      <td>81.0</td>
    </tr>
    <tr>
      <td><a href="https://github.com/ultralytics/assets/releases/download/v0.0.0/yolov8l-pose.pt">YOLOv8l-pose</a></td>
      <td>640</td>
      <td>67.6</td>
      <td>90.0</td>
      <td>784.5</td>
      <td>2.59</td>
      <td>44.4</td>
      <td>168.6</td>
    </tr>
    <tr>
      <td><a href="https://github.com/ultralytics/assets/releases/download/v0.0.0/yolov8x-pose.pt">YOLOv8x-pose</a></td>
      <td>640</td>
      <td>69.2</td>
      <td>90.2</td>
      <td>1607.1</td>
      <td>3.73</td>
      <td>69.4</td>
      <td>263.2</td>
    </tr>
    <tr>
      <td><a href="https://github.com/ultralytics/assets/releases/download/v0.0.0/yolov8x-pose-p6.pt">YOLOv8x-pose-p6</a></td>
      <td>1280</td>
      <td>71.6</td>
      <td>91.2</td>
      <td>4088.7</td>
      <td>10.04</td>
      <td>99.1</td>
      <td>1066.4</td>
    </tr>
  </tbody>
</table>

Reference: https://docs.ultralytics.com/tasks/pose

You can choose your desired model and execute the below command to run inference on an image

```sh
yolo pose predict model=yolov8n-pose.pt source='https://ultralytics.com/images/bus.jpg'
```

Here for model, you can change to either yolov8s-pose.pt, yolov8m-pose.pt, yolov8l-pose.pt, yolov8x-pose.pt, yolov8x-pose-p6 and it will download the relavant pre-trained model

You can also connect a webcam and execute the below command 

```sh
yolo pose predict model=yolov8n-pose.pt source='0'
```

:::note
If you face any errors when executing the above commands, try adding "device=0" at the end of the command
:::

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/YOLOV8-TRT/4.gif
" style={{width:1000, height:'auto'}}/></div>

### Object Tracking

Object tracking is a task that involves identifying the location and class of objects, then assigning a unique ID to that detection in video streams.

Basically the output of object tracking is the same as object detection with an added object ID.

Reference: https://docs.ultralytics.com/modes/track

You can choose your desired model based on object detection/ image segmentation and execute the below command to run inference on an video

```sh
yolo track model=yolov8n.pt source="https://youtu.be/Zgi9g1ksQHc"
```

Here for model, you can change to either yolov8n.pt, yolov8s.pt, yolov8m.pt, yolov8l.pt, yolov8x.pt, yolov8n-seg.pt, yolov8s-seg.pt, yolov8m-seg.pt, yolov8l-seg.pt, yolov8x-seg.pt, and it will download the relavant pre-trained model

You can also connect a webcam and execute the below command 

```sh
yolo track model=yolov8n.pt source="0"
```

:::note
If you face any errors when executing the above commands, try adding "device=0" at the end of the command
:::

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/YOLOV8-TRT/6.gif
" style={{width:1000, height:'auto'}}/></div>

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/YOLOV8-TRT/7.gif
" style={{width:1000, height:'auto'}}/></div>

## Use TensorRT to Improve Inference Speed

As we mentioned before, if you want to improve the inference speed on the Jetson running YOLOv8 models, you first need to convert the original PyTorch models to TensorRT models. 

Follow the steps below to convert YOLOv8 PyTorch models to TensorRT models.

:::note
This works for all four computer vision tasks that we have mentioned before
:::

- **Step 1.** Install ONNX which is a requirement

```sh
pip3 install onnx
```

- **Step 2.** Downgrade to lower version of Numpy to fix an error

```sh
pip3 install numpy==1.20.3
```

- **Step 3.** Execute the export command by specifying the model path

```sh
yolo export model=<path_to_pt_file> format=engine device=0
```

For example:

```sh
yolo export model=yolov8n.pt format=engine device=0
```

:::note
If you, encouter an error about cmake, you can ignore it. Please be patient until the TensorRT export is finished. It might take a few minutes
:::

After the TensorRT model file (.engine) is created, you will see the output as follows

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/YOLOV8-TRT/1.jpg
" style={{width:800, height:'auto'}}/></div>

- **Step 4.** If you want to pass additional arguments, you can do so by following the below table

<table>
<thead>
  <tr>
    <th>Key</th>
    <th>Value</th>
    <th>Description</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td>imgsz</td>
    <td>640</td>
    <td>Image size as scalar or (h, w) list, i.e. (640, 480)</td>
  </tr>
  <tr>
    <td>half</td>
    <td>False</td>
    <td>FP16 quantization</td>
  </tr>
  <tr>
    <td>dynamic</td>
    <td>False</td>
    <td>Dynamic axes</td>
  </tr>
  <tr>
    <td>simplify</td>
    <td>False</td>
    <td>Simplify model</td>
  </tr>
  <tr>
    <td>workspace</td>
    <td>4</td>
    <td>Workspace size (GB)</td>
  </tr>
</tbody>
</table>

For example, if you want to convert your PyTorch model into a TensorRT model in FP16 quantization, execute as

```sh
yolo export model=yolov8n.pt format=engine half=True device=0
```

Once the model is exported successfully, you can directly replace this model with **model=** argument inside **predict** command of **yolo** when running all 4 tasks of detection, classification, segmentation, pose estimation.

For example, with object detection:

```sh
yolo detect predict model=yolov8n.engine source='0' show=True
```

## Bonus Demo: Exercise Detector and Counter with YOLOv8

We have built a pose estimation demo application for exercise detection and counting with YOLOv8 using YOLOv8-Pose model. You can check the project [here](https://github.com/yuyoujiang/Exercise-Counter-with-YOLOv8-on-NVIDIA-Jetson) to learn more about this demo and deploy on your own Jetson device!

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/YOLOV8-TRT/9.gif
" style={{width:1000, height:'auto'}}/></div>

## Resources

- [YOLOv8 documentation](https://docs.ultralytics.com)
- [Roboflow documentation](https://docs.roboflow.com)
- [TensorRT documentation](https://docs.nvidia.com/deeplearning/tensorrt/developer-guide/index.html)

## Tech Support & Product Discussion

Thank you for choosing our products! We are here to provide you with different support to ensure that your experience with our products is as smooth as possible. We offer several communication channels to cater to different preferences and needs.

<div class="button_tech_support_container">
<a href="https://forum.seeedstudio.com/" class="button_forum"></a> 
<a href="https://www.seeedstudio.com/contacts" class="button_email"></a>
</div>

<div class="button_tech_support_container">
<a href="https://discord.gg/eWkprNDMU7" class="button_discord"></a> 
<a href="https://github.com/Seeed-Studio/wiki-documents/discussions/69" class="button_discussion"></a>
</div>