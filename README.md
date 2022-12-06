The main objective is retrieve video output from yolov5 detection on docker while the video is still being analized, allowing us to see detections in real time and save the frames.

What is working:

  -Option to save all frames or just detected frames from video input, using the flags --save-frame or save-detected-frame when executing detect.py;

  -Docker image with yolov5 + functs to save frames + gst installed (guhzantut/yolov5_alpr:latest).


Those frames can be streamed via gst from inside the yolo docker using the following commands:

==============================================================PREPARING YOLO IMAGE==========================================================
Simply:

    docker run --ipc=host --gpus all -it guhzantut/yolov5_live_gst:latest
 
 Or 

Start ultralytics yolov5 docker:

    sudo docker run --ipc=host --gpus all -it ultralytics/yolov5:latest

Instaling gst on docker(pull my docker image: guhzantut/yolov5_live_gst:latest and skip this part):

    DEBIAN_FRONTEND=noninteractive apt-get install -y libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev libgstreamer-plugins-bad1.0-dev gstreamer1.0-plugins-base gstreamer1.0-plugins-good gstreamer1.0-plugins-bad gstreamer1.0-plugins-ugly gstreamer1.0-libav gstreamer1.0-doc gstreamer1.0-tools gstreamer1.0-x gstreamer1.0-alsa gstreamer1.0-gl gstreamer1.0-gtk3

    pkg-config --cflags --libs gstreamer-1.0
 
 ================================SETTING UP ENVIROMENT TO RECEIVE GST STREAM VIA DOCKER================================================================
 
  Exporting the display to vcxsrv for windows (Just tested in Windows 11  with vcxsrv but may work on ubuntu with x11 too).

    export DISPLAY=<display IP>:<display port> (usually <host ip>:0)
    
==================================================RUN THE DETECTION====================================================================================

    python detect.py --source rtsp://<rtspServerIP>:<rtspServerPort>/stream --save-frame

  Wait for yolo to generete some frames before starting the stream( gst's frame rate higher than yolo's processing capacity will eventually  make the stream run out of frames and closing).

=============================================================START STREAMING====================================================================

  At docker's bash start streaming from frames's folder:

    gst-launch-1.0 -e multifilesrc location="/usr/src/app/runs/detect/exp/frames/video%d.png" index=2 caps="image/png,framerate=10/1,width=640,height=480" ! pngdec ! autovideosink sync=false

  *"autovideosink" will display the stream in the DISPLAY, but you may use tdp or udp with gstream or even the gst-rtsp to get the images over network.

WIP:

Build docker image with everything installed and configured; - Done

Make it work with tcp, udp and rtsp;

Start stream directly from source code using opencv images with gst, no more need to save each frame;

Make detect.py accepts gst stream as an input, since they are really faster.


