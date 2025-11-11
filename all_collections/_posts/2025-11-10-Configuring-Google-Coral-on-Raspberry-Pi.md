This tutorial demonstrates how to use a Google Coral USB Accelerator alongside a Raspberry Pi Zero 2W equipped with a Raspberry Pi Camera to perform edge-based image classification inference.

Due to the Google Coral device dropping support for the latest development environments and the newest Raspbian OS images lacking support for Python 3.9 (the latest version compatible with Coral), setting up the device on modern systems can be difficult.
Instead of compiling dependencies manually and dealing with mismatched versions, we’ll use an archival Raspbian image that still supports Python 3.9, simplifying the configuration process.

**System Compatibility and Notes**

Tested on: Google Coral USB Accelerator (plastic case version)
→ Other Coral variants may also work.

Tested with: Raspberry Pi Zero 2W (64-bit ARM, 4-core)
→ Should work on other Raspberry Pi models as well.

Note: This setup uses a legacy OS version that may not receive security patches or updates.

**Step 1 — Prepare the Raspberry Pi OS Image**

**1.1 Download the OS Image**

Download the Raspberry Pi OS Full (64-bit, Bullseye) image that supports Python 3.9:

```
https://downloads.raspberrypi.org/raspios_full_arm64/images/raspios_full_arm64-2023-05-03/2023-05-03-raspios-bullseye-arm64-full.img.xz
```

**1.2 Flash the SD Card**
Use the Raspberry Pi Imager to flash your SD card:
Select the downloaded image.
Apply custom settings (username, password, Wi-Fi configuration, etc.).
Flash and verify.

**1.3 Boot and Verify Python Version**

Insert the SD card, boot your Raspberry Pi, and verify Python:

```
python --version
```
Ensure the output shows Python 3.9.x.

**Step 2 — Install Edge TPU Runtime**

The official Coral setup guide recommends installing the Edge TPU runtime from the Debian repository, but since we’re using an older OS, we’ll manually install a compatible version.

Run the following commands:
```
wget https://github.com/google-coral/libedgetpu/releases/download/release-grouper/edgetpu_runtime_20221024.zip
unzip edgetpu_runtime_20221024.zip
cd edgetpu_runtime/
sudo ./install.sh
```
When prompted, select “N” to disable the maximum frequency option.

**Step 3 — Install Python Dependencies**

Install the necessary Python packages, including the pycoral library, OpenCV, and a compatible NumPy version:
```
python3 -m pip install --extra-index-url https://google-coral.github.io/py-repo/ pycoral~=2.0
pip install opencv-python
pip install numpy==1.26.4
```

**Step 4 — Testing the Setup**

At this stage:
Your Coral USB Accelerator should be connected.
Your Raspberry Pi Camera should be detected by default by the system.

**4.1 Download Model and Labels**
Download a pre-trained MobileNet SSD model and corresponding COCO labels:

```
wget https://github.com/google-coral/test_data/blob/104342d2d3480b3e66203073dac24f4e2dbb4c41/ssd_mobilenet_v2_coco_quant_postprocess_edgetpu.tflite
wget https://github.com/google-coral/test_data/blob/104342d2d3480b3e66203073dac24f4e2dbb4c41/coco_labels.txt
```

**4.2 Create detect.py**
Create a Python file named detect.py and paste the following code:
```
import cv2
import os

from pycoral.adapters.common import input_size
from pycoral.adapters.detect import get_objects
from pycoral.utils.dataset import read_label_file
from pycoral.utils.edgetpu import make_interpreter, run_inference
from picamera2 import Picamera2

def main():
    cap = Picamera2()

    interpreter = make_interpreter('ssd_mobilenet_v2_coco_quant_postprocess_edgetpu.tflite')
    interpreter.allocate_tensors()
    labels = read_label_file('coco_labels.txt')
    inference_size = input_size(interpreter)

    cap.configure(cap.create_video_configuration(main={"format": 'XRGB8888', "size": (800, 600)}))
    cap.start()

    while True:
        print("\nNew frame results:")
        frame = cap.capture_array()
        cv2_im = frame
        cv2_im_rgb = cv2.cvtColor(cv2_im, cv2.COLOR_BGR2RGB)
        cv2_im_rgb = cv2.resize(cv2_im_rgb, inference_size)
        run_inference(interpreter, cv2_im_rgb.tobytes())
        objs = get_objects(interpreter, 0.1)
        for obj in objs:
            print(f'{obj.score * 100:.2f}% {labels.get(obj.id, obj.id)}')

    cap.release()

if __name__ == '__main__':
    main()
```

**4.3 Run the Script**
Execute the detection script:
```
python detect.py
```

**Conclusion**

You’ve successfully configured a Google Coral USB Accelerator with a Raspberry Pi Zero 2W to perform edge-based object detection using a legacy version of Raspberry Pi OS that supports Python 3.9.

This setup enables you to perform real-time inference directly on-device, leveraging Coral’s Edge TPU performance even on lightweight Raspberry Pi hardware.