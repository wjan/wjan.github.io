In this article you'll learn how to install PyTorch on Ubuntu 20 with NVIDIA GPU support using CUDA.

Software stack:
* Ubuntu 20.04.5 LTS
* Python 3.8 with pip
* PyTorch 1.13.1
* CUDA Toolkit 11.7
* NVIDIA driver 515.43.04

## Installation steps

### 1. Install PyTorch 

```
pip3 install torch torchvision torchaudio
```

### 2. Install CUDA Toolkit with NVIDIA driver

**Requirement: disable Noveau driver.** Default Ubuntu installation might contain unsupported Noveau driver installed by default. In order for CUDA installer to succeed you must first disable the Noveau driver:

```
sudo bash -c "echo blacklist nouveau > /etc/modprobe.d/blacklist-nvidia-nouveau.conf"
sudo bash -c "echo options nouveau modeset=0 >> /etc/modprobe.d/blacklist-nvidia-nouveau.conf"
sudo update-initramfs -u
sudo reboot
```

Once the Noveau driver is disabled the following package should be downloaded and ran:

```
wget https://developer.download.nvidia.com/compute/cuda/11.7.0/local_installers/cuda_11.7.0_515.43.04_linux.run
sudo sh cuda_11.7.0_515.43.04_linux.run
```

### 3. Verify installation

To verify the installation simply run Python from command line and paste following lines:

```
import torch
torch.cuda.is_available()
torch.cuda.device_count()
torch.cuda.current_device()
torch.cuda.device(0)
torch.cuda.get_device_name(0)
```

In case of successfull installation your graphics card name would be printed after executing last command. In my case that is:

> 'NVIDIA GeForce GT 1030'

### 4. Further ML model specific configuration

If you made to this step then you've successfully set up GPU in your CUDA environment. It doesn't neccesarilly mean that the ML models you'll run will have the GPU computing enabled by default though - configuring your ML models for the runtime will most probably require pointing to the specific computational unit (i.e. CPU or GPU). Providing this configuration value in your code may be necessary to fully leverage GPU in your environment and such configuration value is specific to certain ML framework and ML model you'll be using. Although further model GPU configuration is out of scope of this article it is worth mentioning and is just one thing to keep it in mind once you get into training and running specific models.
