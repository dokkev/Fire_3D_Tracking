# Thermal Object 3D Tracking

## Demo
Run `roslaunch thermal_object_tracking thermal_object_track.launch`

## Hardware Used
- [FLIR Lepton 2.5 Thermal Camera](https://www.flir.com/products/lepton/)
- [PureThermal 2 - FLIR Lepton Smart I/O Board](https://www.sparkfun.com/products/14670)
- [PureThermal 2 Case](https://www.thingiverse.com/thing:3282890)
- [3D printed Camera Mount](https://github.com/rubberdk/Thermal_Object_3D_Tracking/blob/master/stl/extended_roundcam.stl)
- [Intel Realsense D435](https://www.intelrealsense.com/depth-camera-d435/)

## Software Requirements
- `ROS-noetic`
- `realsense2_ros` packages you can install them [here](https://github.com/IntelRealSense/realsense-ros)
- `pyrealsense2`
- `OpenCV2-Python`


## Introduction
This is a ROS package publishes `tf` called `fire` of a heating element using a thermal camera and a depth camera. Getting the pixel coordinate of the heat source from the thermal imaging, the depth value of the corrspoing pixel from the deph image is calculated. Then the pixel from the depth image is converted into a point. Knowing the transformations from `fire` to `camera_link` and `camera_link` to `world` frame, we can calculate the x, y, and z coordinates of the `fire` relative to the `world` frame

This method does not work unless thermal camera and depth camera are located and aligned properly.
You can take a look at the ROS topic `combined_image` to see how well they are aligned.


## NODE: `thermal_detection`
This node takes care of detecting a highest spot in the camera view and reading its temperature from the thermal camera.
Using USB video class (UVC), it captures thermal imaging data and reads the temperature from the radiometry in the gray scale.
This node was implemented based on [PureThermal UVC Capture Examples](https://github.com/groupgets/purethermal1-uvc-capture)

### SUBSCRIBER & PUBLISHER
Subscriber:
- none

Publisher:
- /object_y (std_msgs/Float32): y-coordinate of the heat source in 2D thermal image 
- /object_x (std_msgs/Float32): x-coordinate of the heat source in 2D thermal image 
- /highest_T_detected (std_msgs/Float32): the highest tempertaure detected
- /thermal_image (sensor_msgs/Image): radiometric image

## NODE: `combined_vision`
This package combines the radiometric image and `camera/aligned_depth_to_color/image_raw` topic which `realsense2_ros` publishes. 

### SUBSCRIBER & PUBLISHER
Subscriber:
- /camera/aligned_depth_to_color/image_raw (sensor_msgs/Image): aligned depth to color image from `realsense`
- /camera/color/image_raw (sensor_msgs/Image): RGB color image from `realsense`

Publsuher:
- /object_y (std_msgs/Float32): y-coordinate of the heat source in 2D thermal image 
- /object_x (std_msgs/Float32): x-coordinate of the heat source in 2D thermal image 
- /highest_T_detected (std_msgs/Float32): the highest tempertaure detected
- /thermal_camera/thermal_image_bgr (sensor_msgs/Image): radiometric image in BGR format
- /thermal_camera/thermal_image_gray (sensor_msgs/Image): radiometric image in gray scale
- /combined_image (sensor_msgs/Image): combined image of radiometic image and aligned depth to color image
- /combined_image2 (sensor_msgs/Image): combined image of radiometic image and RGB color image
- /stacked_image (sensor_msgs/Image): horizontally stacked image of radiometic image and aligned depth to color image

## NODE: `get_depth`
This node reads the aligned depth to color image, gets depth value from the tageted pixel, and converts the pixel to a point

### SUBSCRIBER & PUBLISHER
Subscriber:
- /object_y (std_msgs/Float32): y-coordinate of the heat source in 2D thermal image 
- /object_x (std_msgs/Float32): x-coordinate of the heat source in 2D thermal image 
- /camera/aligned_depth_to_color/image_raw (sensor_msgs/Image): aligned depth to color image from `realsense`
- /camera/aligned_depth_to_color/camera_info (sensor_msgs/CameraInfo): camera info of `realsense`'s aligned depth to color image

Publisher:
- /point_coord (thermal_object_tracking.msg/point_coord):  x, y, and depth data from the point converted from the the pixel

## NODE: `tf_listener`
This node publishes `tf` of the heat source (`fire`) and calculates the x, y, and z coordinates of the `camera_link` and `fire` relative to the `world` frame

### SUBSCRIBER & PUBLISHER
Subscriber:
- /point_coord (thermal_object_tracking.msg/point_coord):  x, y, and depth data from the point converted from the the pixel

Publisher:
- /camera_position (thermal_object_tracking.msg/xyz): xyz-coordinate of the `camera_link` from the `world` frame
- /fire_position (thermal_object_tracking.msg/xyz): xyz-coordinate of the `fire`from the `world` frame







