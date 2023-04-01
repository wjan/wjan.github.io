In this article I'll walk you through my first experiment related to environment object mapping using object detection and mono depth ML algorithms. Inspired with Tesla autopilot and Full Self Driving suites surroundings real time 3D visualization I'll try to recreate similar  TODOs

For the sake of this experiment I'll use two prebaked PyTorch models, both available on PyTorch hub:
* Yolo (TODO link, version) - responsible for handling basic object detection, pretrained model can recognize (TODO) object classes
* MiDaS: https://pytorch.org/hub/intelisl_midas_v2/ - responsible for monocular dept estimation, in oder words this ML model can estimate the distance from the camera to the object. The beauty of this model is that it works quite well despite of the fact it uses only single image (thus 2D vision instead of 3D vision).

In this experiment I'll try to combine both ML model in order to create 2D spacial map of objects found in an image or video. Some custom Python code will be written to tie the process together and visualize the 2D map of objects.

The high level algorithm will consist of following steps:
1. Retrieve an image (i.e. a single video frame)
2. Process the image with monodepth MiDaS model. It will return a heat map with an estimation of depth for every pixel.
3. Process the image with Yolo object detection model. It will return a list of found objects with their coordinates.
4. Match every object found using Yolo with a heatmap.  Object coordinates will be used to get the MiDaS depth.
5. Show the image on a screen using 2D visualization graphics library.

## Screenshots
![Yolo](https://raw.githubusercontent.com/wjan/wjan.github.io/main/img/yolo.png)
![MiDaS](https://raw.githubusercontent.com/wjan/wjan.github.io/main/img/midas.png)
![Yolo + MiDaS](https://raw.githubusercontent.com/wjan/wjan.github.io/main/img/molo.png)


Problems 
Object flickering




Conclusions:
1. Due to the nature of self driving environment solution for this problem might have some possible simplifications but also complication i.e.:
* Self driving simplification: car drivers have to navigate on a 2D plane instead of 3D space. Thus problem with height and Z axis position estimation rarely occurs. In this POC we have to deal with an environment with Z axis, let's say there is a glass on the top of the table and those object have to be correctly mapped one on top of another. Such positioning rarely occurs for the objects relevant for driving - cars, pedestrians and obstacles are always placed on top of one and only surface.
* Self driving comlication: the object from where the mapping takes place (which is basically a set of camera) is either stopped or moving. Most of the times the camera will be moving with the car so the mapped environment will be changing constantly and those environment changes should be presented gracefully with some animation. Solving the animation will most probably involve maintaning a memory of recognized objects, so mechanism will keep track of unique objects around.
3. Camera image quality, lightning and recognized object flickering (possibly due to image compression) were challenging to provide a good quality of 2D spatial map. I.e. coordinates of frames for recognized objects may vary depending on a frame. The location of frame in time might be flickering (or jumping) so objects may seem as they are chaning position even when not moving. 

Road to take from here:
1. 3D visualization
2. Object size visualization
