The main objective is retrieve video output from yolov5 detection on docker while the video is still being analised, allowing us to see detections in real time and save the frames.

For now i have created a function in utils/plots to save all frames from videos, using the flags --save-frame or save-detected-frame when executing detect.py.
E.g.: python detect.py --source rtsp://<rtspServerIP>:<rtspServerPort>/stream --save-frame

Those frames can be streamed via gst(must be installed) from inside the yolo docker using the following commands:

  Instaling gst on docker:

    DEBIAN_FRONTEND=noninteractive apt-get install -y libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev libgstreamer-plugins-bad1.0-dev gstreamer1.0-plugins-base gstreamer1.0-plugins-good gstreamer1.0-plugins-bad gstreamer1.0-plugins-ugly gstreamer1.0-libav gstreamer1.0-doc gstreamer1.0-tools gstreamer1.0-x gstreamer1.0-alsa gstreamer1.0-gl gstreamer1.0-gtk3

    pkg-config --cflags --libs gstreamer-1.0

    export DISPLAY=192.168.0.22:0(Just tested in Windows 11  with vcxsrv)

Wait for yolo to generete some frames before executing, a frame rate higher than yolo's capacit will stop the stream.

 At docker's bash start streaming from png's folder:

  GST_DEBUG=3 gst-launch-1.0 -e multifilesrc location="/usr/src/app/runs/detect/exp21/frames/video%d.png" index=2 caps="image/png,framerate=10/1,width=640,height=480" ! pngdec ! videoconvert  ! videoscale ! video/x-raw,width=680,height=480  ! autovideosink 

"autovideosink" will display the stream in the screen (as docker is running in a windows host vcxsrv is needed in the host and running "export DISPLAY=<hostIP>:0), but you may use tdp or udp with gstream or even the gst-rtsp )

WIP:

Build docker image with everything installed and configured,
Make it work with tcp, udp and rtsp,
Start stream directly from source code using opencv images with gst, no more need to save each frame.


