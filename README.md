This tutorial follows the guide available at the following link:  
https://github.com/Xilinx/Vitis-AI-Copyleft-Model-Zoo.git

### Prerequisites
- Install Ubuntu 20.04.
- Install NVIDIA CUDA (if you want to use a GPU).
- Install Docker from the link below:  
https://docs.docker.com/desktop/install/ubuntu/

### Steps

1. **Clone the latest Vitis AI repository:**
   ```bash
   git clone https://github.com/Xilinx/Vitis-AI
   cd Vitis-AI
   ```

2. **Clone the Copyleft Model Zoo repository and copy the YOLOv7 folder to the Vitis-AI directory:**
   ```bash
   git clone https://github.com/Xilinx/Vitis-AI-Copyleft-Model-Zoo.git
   cp -r Vitis-AI-Copyleft-Model-Zoo/yolov7 Vitis-AI/
   ```

3. **Download the latest Vitis AI Docker container (CPU version):**
   ```bash
   docker pull xilinx/vitis-ai-cpu:latest
   ```

4. **Run the Docker container:**
   ```bash
   ./docker_run.sh xilinx/vitis-ai-cpu:latest
   ```

5. **Activate the Vitis AI environment:**
   ```bash
   conda activate vitis-ai-pytorch
   ```

6. **Install YOLOv7 requirements:**
   ```bash
   !pip install -r yolov7/requirements.txt
   ```

7. **Update `g++`:**
   ```bash
   !sudo add-apt-repository -y ppa:ubuntu-toolchain-r/test
   !sudo apt install -y g++-11
   ```

8. **Get the COCO dataset:**
   ```bash
   %cd yolov7/
   !bash scripts/get_coco.sh
   %cd ../
   ```

   To use a custom dataset, annotate the dataset using LabelImg or other software. Then follow the tutorial linked below to prepare it for training:  
   https://www.youtube.com/watch?v=GRtgLlwxpc4

9. **Download the YOLOv7 training model from GitHub and place it in the `yolov7` folder:**
   ```bash
   cd yolov7 
   wget https://github.com/WongKinYiu/yolov7/releases/download/v0.1/yolov7_training.pt
   ```

10. **Train with quantization-aware training (QAT):**  
   Keep `--device 0` if running on GPU; otherwise, remove it.
   ```bash
   %cd yolov7/
   !python train_qat.py --workers 16 --device 0 --epochs 100 --batch-size 8 --data data/coco.yaml --img 640 640 --cfg cfg/training/yolov7.yaml --weights yolov7_training.pt --name yolov7-qat --hyp data/hyp.scratch.p5_qat.yaml --nndct_convert_sigmoid_to_hsigmoid --nndct_convert_silu_to_hswish 
   %cd ../
   ```

11. **Dump the model:**
   ```bash
   %cd yolov7/
   !python test_nndct.py --data data/coco.yaml --img 640 --batch 1 --conf 0.001 --iou 0.65 --device 0 --weights ../quantized/qat_09.pt --name yolov7_640_val --quant_mode test --nndct_qat --nndct_convert_sigmoid_to_hsigmoid --nndct_convert_silu_to_hswish --dump_model
   %cd ../
   ```