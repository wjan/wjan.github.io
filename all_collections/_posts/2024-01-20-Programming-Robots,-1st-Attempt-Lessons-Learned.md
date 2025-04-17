**Motivation**

Recently I've been doing more DIY home automation projects and liked the idea of applying the passion for creating software into real world devices.
The decision to give it a try and start tinkering with **robotics**, although scary, seemed the natural path to take on for further exploration of the potential passion, especially in the age of AI taking over almost anything these days which would be the obvious addition to potentially build something useful, fun and insightful. 

**Place to start**

Since I don't have any practical background in electronics I decided to order pre-build ESP32Cam WI-FI based RC car that would be a proof of concept platform and potentially a base to even move beyond that.

Several factors I've considered into buying this specific WI-FI RC car:
* ESP32Cam chip - has WI-FI, built in camera, easy to modify and flash the RC car firmware
* Web-based software ready to use with potential to use as API
* Ease of assembly - boils down into plugging in the battery and attaching the ESP32Cam chip into the custom RC car base plate
* Price - around 60 USD
* Aesthetics - looks quite decent compared to a lot of other options that seem more DIY like

Given that I decided to order following RC car directly from China:

** link **
** photo **

**What I've received**

After the unboxing the product makes a very positive impression. It's built well, it has custom PCB base plate that was specifically designed for this RC car and compared to others there are no wires and cables dangling everywhere around it. Well, until you start modifying it of course ;)

After a closer look I've quickly discovered there are some downsides to this cheap IoT RC car (or basically - ANY cheap IoT RC car):
* The quality of the camera image is very low even at the highest settings. It's both due to limitation of the sensor as well as the processing power of ESP32 chip
* The camera is prone to blur or jitter when IoT RC car is in move or rotating. That leads to problems for the further object detection algorithms 
* The built in ESP32Cam PCB WI-FI camera is weak 
* The built in ESP32Cam PCB WI-FI antenna has weak signal but (supposedly) you can attach an external antenna to improve it
* Seems the IoT RC car firmware is sloppy, it doesn't always work as expected and it's just a Chinese spinoff of Pilot Hobbies software (link) which in order to flash it yourself you have to tinker with a lot (they don't provide the modified Pilot Software code, you have to work with the original one and adjust it for the IoT RC car)

**What I did**

After turning on the IoT RC car exposes an open WI-FI network (named Scout32, exactly as it's original version from Pilot Code) that you can connect into with your laptop or phone. Once connected you can access the 192.168.4.1 host that serves a Pilot Hobbies website showing you the video stream of the ESP32Cam camera and gives you some buttons to control the IoT RC car.

Having that I've quickly extracted those HTTP call details that I was able to launch directly from the terminal using curl or Python.

**curl example link**

With little bit more work and ChatGPT help I was also able to retrieve the binary video stream from the ESP32Cam camera.

Given that I already had an access to both input (possibility to control) and the output (ability to see) of the IoT RC car/robot I was able to start applying the actual AI on top of it.

Due to my previous object detection experiences (link) I decided to use Yolo to process the image in order for the robot to understand the surroundings better. That way I could plan to create a simple AI based program for the basic seek-and-destroy algorithm.
Since ESP32Cam is very limited chip that can barely process video streaming at highest settings I needed to use my laptop as a platform for all the complex object detection processing. End of all, in the first phase of the experiments the POC will be run on the laptop which will basically retrieve video stream from the robot, process its frames in order to detect objects and then send commands back to the robot in order to move them into the desired direction.

**What I've learned**
...

