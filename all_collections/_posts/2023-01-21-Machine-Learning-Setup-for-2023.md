## General requirements
It's important to get your machine learning setup just right. Your system has to be fast and reliable and in many cases the crucial requirement is to have CUDA supported graphics card.

Couple of requirements that needs to be met for machine learning research and development are:
* Fast and reliable general hardware setup
    * Plenty of ram
    * Fast SSD drive
* Nvidia graphics card that could leverage CUDA capabilities thus increase neural networks processing speed
* Right operating system
    * My choice is Linux based OS
    * I prefer Ubuntu due to ease of installation and use, wide support from many software vendors (such as Nvidia)
    * OS version should be chosen carefully, as some older versions don't have the support from vendors **anymore** and newest versions don't have the support **yet**
        * Ubuntu 20.04.5 LTS seems to work just fine, as I've managed to meet all the dependency chain for Nvidia drivers, CUDA and PyTorch (that's especially important to get the GPU support just right)

Given that here's my currenct ML setup.

## Hardware:
* Intel(R) Core(TM) i3-6100 CPU @ 3.70GHz (4 cores)
* 16 GB of RAM
* Nvidia GeForce GT 1030 (low profile version - smaller and less energy consumtion)
* 100 GB SSD partition
* 34" Xiaomi Curved wide screen monitor

## Software:
* Ubuntu 20.04.5 LTS
* Python 3.8
* PyTorch 1.13.1
* CUDA Toolkit 11.7
* Nvidia driver 515.43.04

## Importance of GPU enabled ML
It's not the fastest setup you can build (on contrary, it's probably quite slow), but I've basically took the old PC that was available to me and upgraded the RAM, disk and graphics card. For some initial research it's enoguh, especially that I've managed to enable my GPU for ML training. In example, GPU enabled object detection model training takes 3 times less time for me. Also running object detection models real time (hooked up to a webcam, thus processing many image frames per second) makes the difference, instead of choppy frame rate on CPU I get a full frame rate with GPU.

## Recommended add-ons
Couple of things might be handy while you're on your way to learn machine learning.
* 4GB pendrive (or bigger) for your OS installation image. I keep the image of my Ubuntu 20.04.5 LTS on pendrive so I'm able to reinstall Ubuntu quickly in case of some serious system mess up. That's quite important especially on the beginning if you come into some problems while matching right versions of software dependencies and you want to simply start over again with a fresh system.
* venv for Python. Once I started with ML I had zero idea about Python pip packages and their dependency hell. Simply put, installing some major pip packages will cause reinstalling your existing pip packages with different versions, making your old project stop working. venv is the way to go, as you can isolate your pip packages into different projects.
