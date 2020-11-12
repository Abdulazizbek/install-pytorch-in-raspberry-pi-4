# Install-pytorch-in-raspberry-pi-4
Install Ubuntu on Raspberry Pi 4, Create Pytorch environment on Raspberry pi 4

## Downloading and Installing Ubuntu 20.04 LTS for Raspberry Pi

Visit the official website of [Ubuntu](https://ubuntu.com/) and go to the Download section.

Highly recommend to follow [this great guideline](https://linuxhint.com/install-ubuntu-desktop-20-04-lts-on-raspberry-pi-4/) for instalation process, when you reach  `internet connection` and `ubuntu-desktop` parts follow these links (some Possible troubleshooting also considered for easiness):

* Connecting Raspberry Pi 4 to the Internet: [link](https://itsfoss.com/connect-wifi-terminal-ubuntu/)

* Instead of `ubuntu-desktop`, I suggest to install `lubuntu-desktop` in terms of lightweightness:
  * `sudo apt install ubuntu-desktop` ==> `sudo apt install lubuntu-desktop`
  
## Check Python and is packages working properly

By default `python` version is `3.8` and need to install `pip`, some related packages and check is working properly

After some effort and the help of blogposts, I could compile the version of PyTorch, which I desired.

First, compile and install [sleef](https://sleef.org/compile.xhtml). You should also add the following line to your .bashrc (or equivalent):
* `export LD_LIBRARY_PATH='$LD_LIBRARY_PATH:/usr/local/lib'`

and ensure that sleef is working properly, however for preventing ‘cannot open shared object file’ error, run:
* `ldconfig`
* `sudo /sbin/ldconfig -v`
* `apt search libsleef`

## Procedure for Installing `PyTorch` and `PyTorch Vision` on Raspberry Pi 4B

#### 1. First, update the Raspbian OS: 
  * `sudo apt-get update`
  * `sudo apt-get dist-upgrade`

#### 2. Install dependencies:
  * `sudo apt install libopenblas-dev libblas-dev m4 cmake cython python3-dev python3-yaml python3-setuptools`
  * `sudo apt-get install libavutil-dev libavcodec-dev libavformat-dev libswscale-dev`
  
#### 3. You can install `PyTorch` and `PyTorch Vision` using the pre-compiled Python wheel files for saving your time for compiling:
  * download wheel files
  * `sudo pip3 install torch-1.6.0a0+b31f58d-cp38-cp38-linux_aarch64.whl`
  * `sudo pip3 install torchvision-0.7.0a0+78ed10c-cp38-cp38-linux_aarch64.whl`
  
#### 4. Or, if you would like to get the different version by building PyTorch from the source yourself, which may take 6~10 hours (otherwise skip this step):
  
  * For preventing OOM memory error, add swap memory:
    * `sudo dphys-swapfile swapoff`
    * `sudo fallocate -l 2G /swapfile`
    * `sudo chmod 600 /swapfile`
    * `sudo mkswap /swapfile`
    * `suso swapon /swapfile`
    * `sudo swapon /swapfile`
    * `sudo nano /etc/fstab`
    * `sudo swapon --show`
    * `sudo free -h`
  
  * and for preventing `blocked for more than 120 seconds.` stack, run:
    * `sudo sysctl -w vm.dirty_ratio=10`
    * `sudo sysctl -w vm.dirty_background_ratio=5`    
    > Then commit your changes:
      * `sudo sysctl -p`

## Compile Pytorch and Torchvision

  #### 1. Download source code and compile
  * `mkdir pytorch_installation`
  * `cd pytorch_installation`
  * `git clone --recursive https://github.com/pytorch/pytorch`
  * `cd pytorch`
  * `git checkout v1.6.0` # choose the version
  * `git submodule sync` 
  * `git submodule update --init --recursive`
  * `git submodule update --remote third_party/protobuf`
  
  #### 2. Edit CMakeLists.txt to set two flags:
  * `option(USE_PYTORCH_QNNPACK "Use ATen/QNNPACK (quantized 8-bit operators)" OFF)` # By dedault was `ON`
  * `option(USE_SYSTEM_SLEEF "Use system-provided sleef." ON)` # By default was `OFF`
  
  #### 3. Compile Pytorch
  * `export BUILD_TEST=0`
  * `export USE_SYSTEM_NCCL=0`
  * `export USE_CUDA=0`
  * `export USE_MKLDNN=0`
  * `export USE_NNPACK=0`
  * `export USE_QNNPACK=0`
  * `export USE_NUMPY=1`
  * `export USE_DISTRIBUTED=0`
  * `export NO_CUDA=1`
  * `export NO_DISTRIBUTED=1`
  * `export NO_MKLDNN=1`
  * `export NO_NNPACK=1`
  * `export NO_QNNPACK=1`
  * `export ONNX_ML=1`
  * `python3 setup.py build`
  * `sudo -E python3 setup.py install`
  * `python setup.py bdist_wheel` # build whl file. The wheel will be in the `dist/` folder.
  
  #### 4. Compile Torchvision
  * `cd ..`
  * `mkdir pytorch_vision && cd pytorch_vision`
  * `git clone --recursive https://github.com/pytorch/vision`
  * `cd vision`
  * `sudo -E python3 setup.py install`
  * `python setup.py bdist_wheel` # build whl file. The wheel will be in the `dist/` folder.

#### 5. Check the installation by running a simple test and confirming that Python outputs a tensor
* `cd`
* `python3`
* `from __future__ import print_function`
* `import torch`
* `a = torch.rand(5,3)`
* `print (a)`
