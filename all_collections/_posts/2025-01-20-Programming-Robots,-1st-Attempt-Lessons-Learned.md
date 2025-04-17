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

Given that I decided to order [following RC car directly from China](https://pl.aliexpress.com/item/1005006722055337.html):

![IoT RC car from China](https://raw.githubusercontent.com/wjan/wjan.github.io/main/img/rccar.jpeg)

**What I've received**

After the unboxing the product makes a very positive impression. It's built well, it has custom PCB base plate that was specifically designed for this RC car and compared to others there are no wires and cables dangling everywhere around it. Well, until you start modifying it of course ;)

After a closer look I've quickly discovered there are some downsides to this cheap IoT RC car (or basically - ANY cheap IoT RC car):
* The quality of the camera image is very low even at the highest settings. It's both due to limitation of the sensor as well as the processing power of ESP32 chip
* The camera is prone to blur or jitter when IoT RC car is in move or rotating. That leads to problems for the further object detection algorithms 
* The built in ESP32Cam PCB WI-FI camera quality is poor 
* The built in ESP32Cam PCB WI-FI antenna has weak signal but (supposedly) you can attach an external antenna to improve it
* Seems the IoT RC car firmware is sloppy, it doesn't always work as expected and it's just a Chinese spinoff of [Pilot Hobbies software](https://pilothobbies.com/) which in order to flash it yourself you have to tinker with a lot (they don't provide the modified Pilot Software code, you have to work with the original one and adjust it for the IoT RC car)

**What I did**

After turning on the IoT RC car exposes an open WI-FI network (named Scout32, exactly as it's [original version from Pilot Code](https://www.pilothobbies.com/product/scout32/)) that you can connect into with your laptop or phone. Once connected you can access the 192.168.4.1 host that serves a Pilot Hobbies website showing you the video stream of the ESP32Cam camera and gives you some buttons to control the IoT RC car.

Having that I've quickly extracted those HTTP call details that I was able to launch directly from the terminal using curl or Python.

**curl example link**

With little bit more work and ChatGPT help I was also able to retrieve the binary video stream from the ESP32Cam camera.

Given that I already had an access to both input (possibility to control) and the output (ability to see) of the IoT RC car/robot I was able to start applying the actual AI on top of it.

Due to [my previous object detection experiences](https://wjan.github.io/posts/Tesla-like-2D-Environment-Mapping-Using-YOLOv5-and-MiDaS/) I decided to use Yolo to process the image in order for the robot to understand the surroundings better. That way I could plan to create a simple AI based program for the basic seek-and-destroy algorithm.
Since ESP32Cam is very limited chip that can barely process video streaming at highest settings I needed to use my laptop as a platform for all the complex object detection processing. End of all, in the first phase of the experiments the POC will be run on the laptop which will basically retrieve video stream from the robot, process its frames in order to detect objects and then send commands back to the robot in order to move them into the desired direction.

**What I've learned**
* As mentioned before - the ESP32Cam camera is of very poor quality. Using that with even a decent object detection model will not give you good results. Due to jitter, blur or just poor lightning the object detection model will not return correct prediction making the algorithm really unstable as many frames of the input video stream will not be resolved correctly thus the algorithm will not make good (or any) decision for the robot to take. 
    * One way to overcome it is to move the robot sequentially (in other words - by taking little steps) only to grab the video frame while it's stopped - that way we can make sure that the video frame will be still and having the best possible quality. There's a downside here of course which is that such algorithm is really slow and it takes quite longer time to plan next move. Also due to this blur and jitter disruptions it's quite impossible to achieve fluid movement of the robot.
    * Another way to overcome that would be to use another camera of better quality but it will not be suitable for ESP32Cam chip as whatever is shipped with it is almost as good as it gets. If I want to have a better camera on board then I should think of hooking up some other chip on top of the robot and there might be some options, such as Raspberry Pi with camera combination, as long as the car still has some energy left to lift and power it up. This seems to be preferred way as I at the end of the POC I won't be having the robot algorithm running externally on the laptop and I want the IoT RC car to be as autonomous as possible.
* As also mentioned before - the software based on Pilot Hobbies Scout32 that is burned into the ESP32Cam chip have some bugs and it's sloppy. You can [get open source version of Scout32](https://www.pilothobbies.com/wp-content/uploads/2022/05/Scout32-ScoutXL_2.2.zip) and modify using Arduino IDE which is an improvement, also the default access point mode being on the ESP32Cam by default can be switched into the client mode so the robot can connect into your WI-FI router directly instead you connecting into the robot.
* Robot development seems to be more more healthy than general programming ;) You'll have to walk away from your laptop and chase or move the robot if it goes to far or collides with the objects in the room.

**What's next**
* This robot needs a better camera. What I need is better resolution, less jitter or blur. 
* This robot needs a autonomous brain - possibly a Raspberry Pi and preferably one that can handle some kind of object detection algorithm that processes faster than 1 frame per second.
* End of all - this robot needs to move fluidly and semi-intelligently while avoiding bumping into obstacles.

**Only that much and that much**