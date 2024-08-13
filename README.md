This tutorial is made following below tutorial:
https://github.com/Xilinx/Vitis-AI-Copyleft-Model-Zoo.git


To build the environment the Ubuntu 20.04 needed to be installed 
Install NVIDIA CUDA (if want to use gpu)

Install docker from below link:
https://docs.docker.com/desktop/install/ubuntu/


Clone latest Vitis ai 
```
git clone https://github.com/Xilinx/Vitis-AI
cd Vitis-AI

```

Clone below link and copy the yolov folder to Vitis-AI direcotory
```
git clone https://github.com/Xilinx/Vitis-AI-Copyleft-Model-Zoo.git

cp -r Vitis-AI-Copyleft-Model-Zoo/yolov7 Vitis-AI_directrory
```


Download the latest Vitis AI Docker with the following command. This container runs on CPU.

```
docker pull xilinx/vitis-ai-cpu:latest  
```

To run the docker, use command:
```
./docker_run.sh xilinx/vitis-ai-cpu:latest
```



Activate environment 

```
conda activate vitis-ai-pytorch

```

Install yolov7 requrements

```
!pip install -r yolov7/requirements.txt

```

Update g++
```
!sudo add-apt-repository -y ppa:ubuntu-toolchain-r/test
!sudo apt install -y g++-11
```

Get coco dataset 
```
%cd yolov7/
!bash scripts/get_coco.sh
%cd ../
```

Download yolov7_trainign model from github and place in yolov7 folder
```
cd yolov7 
wget https://github.com/WongKinYiu/yolov7/releases/download/v0.1/yolov7_training.pt
```

Train qant aware training 
keep --device 0 if you are running on GPU else remove 

```
%cd yolov7/
!python train_qat.py --workers 16 --device 0 --epochs 100 --batch-size 8 --data data/coco.yaml --img 640 640 --cfg cfg/training/yolov7.yaml --weights yolov7_training.pt --name yolov7-qat --hyp data/hyp.scratch.p5_qat.yaml --nndct_convert_sigmoid_to_hsigmoid --nndct_convert_silu_to_hswishv 
%cd ../
```

Dump model: 

```
%cd yolov7/
!python test_nndct.py --data data/coco.yaml --img 640 --batch 1 --conf 0.001 --iou 0.65 --device 0 --weights ../quantized/qat_09.pt --name yolov7_640_val --quant_mode test --nndct_qat --nndct_convert_sigmoid_to_hsigmoid --nndct_convert_silu_to_hswish --dump_model
%cd ../

```








