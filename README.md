
# Thesis Fiten-Winant improvement of Implicit3D

This is the github page of the master thesis of Fiten-Winant, where we propose an server implentation of the [Implicit3D](https://arxiv.org/pdf/2103.06422) scene understanding.

In this github repo we find two branches. One is the server implementation. This code can be used to deploy the server with pretrained models in it, do response time tests and generate data for 2D detectors. The other one is a repo where the 2D detectors can be trained. We created also an unity program that is our proof of concept this is to big of a file to share so this is on request.




## Setting up environment

Start by installing some dependencies we need:
```
sudo apt install xvfb ninja-build freeglut3-dev libglew-dev meshlab
```

Install [CUDA toolkit](https://developer.nvidia.com/cuda-12-2-0-download-archive). To check if it is installed properly try:

```
nvidia-smi
nvcc -V
```
Both commands should give the same CUDA version. Al code is made with CUDA 12.2 on an ubuntu 22.04.

To set up the conda environment go the root folder and run:

```
conda env create -f environment.yml
conda activate Im3D

```




## Demo

Find the Compute Capability on [cuda-gpus](https://developer.nvidia.com/cuda-gpus) of your GPU. Set this values in 
```
Implicit3D/external/mesh_fusion/libfusiongpu/CMakeLists.txt
```
Then go to 
```
Implicit3D/external/ldif/ldif2mesh/get_cuda_version.py 
```
the method should return major and minor version of you Cuda Toolkit.

Go to the /Implicit3D and run:
```
python project.py build
```
Download the [pretrained](https://stduestceducn-my.sharepoint.com/:u:/g/personal/2015010912010_std_uestc_edu_cn/Efs2Tqlkk_pIhy16ud20m5sBMkbkWJEuspiLjdF4G2jOzA?e=sxnswk)  checkpoint  and unzip it into out/total3d/20110611514267/

To check if everything works fine try to run the demo:
```
CUDA_VISIBLE_DEVICES=0 python main.py out/total3d/20110611514267/out_config.yaml --mode demo --demo_path demo/inputs/1
```

In case of errors please visit [this githubpage](https://github.com/chengzhag/Implicit3DUnderstanding/tree/main)



## Data preparation

In the Implicit3D directory do following (this step is only needed if the pretrained models are not used or when the response time is tested):


1. Follow [Total3DUnderstanding](https://github.com/yinyunie/Total3DUnderstanding) to download the raw data.

2. According to [issue #6](https://github.com/yinyunie/Total3DUnderstanding/issues/6) of [Total3DUnderstanding](https://github.com/yinyunie/Total3DUnderstanding),
there are a few typos in json files of SUNRGBD dataset, which is mostly solved by the json loader.
However, one typo still needs to be fixed by hand.
Please find ```{"name":""propulsion"tool"}``` in ```data/sunrgbd/Dataset/SUNRGBD/kv2/kinect2data/002922_2014-06-26_15-43-16_094959634447_rgbf000089-resize/annotation2Dfinal/index.json``` and remove ```""propulsion```.

3. Process the data by
    ```
    python -m utils.generate_data
    ```

To generate the timing data (being the SUNRGBD dataset but transformed to only pictures and camera intrinstic values matrix) run following command in the root of the project.

```
python generate_time_data.py
```
## Training and Testing

Look at the other repo to find out how to train the 2D detectors and assess them. Train the detector or download the [pretrained model](https://drive.google.com/file/d/1Z_9QHP-3vpYd13l8iTnC33Qqh7XVm-Eo/view?usp=sharing)
Name this pth file "fine_tuned.pth" and save it in the Implicit3D directory.

To assess the average time the Implicit3D takes to do predictions you can run (the 100 can be changed to the amount of samples you take to calculate the average):
```
CUDA_VISIBLE_DEVICES=0 python main.py out/total3d/20110611514267/out_config.yaml --mode demo_with_time --avg_amount 100
```
All the data is afterwards printed and can be used to make custom visualisations an example is given in timing.py. A basic pie chart is already generated.


## Server

Go to Implicit3D/server.py and insert the hostname of the server at the last line then run this command in the Implicit3D directory:
```
python3 server.py out/total3d/20110611514267/out_config.yaml
```
To asses the response time client.py is included. In the file change the ip address where the server is running. This python script can connect to a server and times the response time. In the server.py and in the process_API.py are print command that gives the timing list for certain subprocesses. All this data can be put in timing.py and plots can be generated.
## Camera Intrinsic Value Matrix

To calculate the Camera Intristic Value Matrix of the Meta Quest 3. Run in Matrix Directory:
```
python intrinsic.py
```
