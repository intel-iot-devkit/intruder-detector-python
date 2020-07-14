# Intruder Detector

| Details               |                   |
| --------------------- | ----------------- |
| Target OS:            | Ubuntu* 18.04 LTS |
| Programming Language: | Python* 3.5       |
| Time to Complete:     | 45 min            |

![intruder-detector](./docs/images/intruder.png)

**Figure 1:** An application capable of detecting any number of objects from a video input.

## What it does
This reference implementation detect the objects in a designated area. It gives the number of objects in the frame, total count and also record the alerts of the objects present in the frame. The application is capable of processing the inputs from multiple cameras and video files.

## Requirements

### Hardware

- 6th to 8th generation Intel® Core™ processors with Iris® Pro graphics or Intel® HD Graphics

### Software

- [Ubuntu\* 18.04 LTS](http://releases.ubuntu.com/18.04/)<br>
  NOTE: Use kernel versions 4.14+ with this software.<br> 
  Determine the kernel version with the below uname command. 

  ```
   uname -a
  ```

- Intel® Distribution of OpenVINO™ toolkit 2020 R3 release

## How It works

The application uses the Inference Engine included in the Intel® Distribution of OpenVINO™ toolkit. A trained neural network detects objects within a designated area by displaying a green bounding box over them and registers them in a logging system.

![arch_diagram](./docs/images/architectural-diagram.png)

**Figure 2:**  Architectural Diagram

## Setup

### Get the code

Steps to clone the reference implementation: (intruder-detector)
```
sudo apt-get update && sudo apt-get install git
git clone https://github.com/intel-iot-devkit/intruder-detector-python.git
```
### Install Intel® Distribution of OpenVINO™ toolkit

Refer to [Install Intel® Distribution of OpenVINO™ toolkit for Linux*](https://software.intel.com/en-us/articles/OpenVINO-Install-Linux) to learn how to install and configure the toolkit.

Install the OpenCL™ Runtime Package to run inference on the GPU, as shown in the instructions below. It is not mandatory for CPU inference.

### Other dependencies
#### FFmpeg* 
FFmpeg is a free and open-source project capable of recording, converting and streaming digital audio and video in various formats. It can be used to do most of our multimedia tasks quickly and easily say, audio compression, audio/video format conversion, extract images from a video and a lot more.

### Which model to use

The application uses the [person-vehicle-bike-detection-crossroad-0078](https://docs.openvinotoolkit.org/2020.3/_models_intel_person_vehicle_bike_detection_crossroad_0078_description_person_vehicle_bike_detection_crossroad_0078.html) Intel® model, that can be accessed using the **model downloader**. The **model downloader** downloads the __.xml__ and __.bin__ files that will be used by the application.

The application also works with any object-detection model, provided it has the same input and output format of the SSD model.
The model can be any object detection model:
- Downloaded using the **model downloader**, provided by Intel® Distribution of OpenVINO™ toolkit.

- Built by the user.<br>

To install the dependencies of the RI and to download the **person-vehicle-bike-detection-crossroad-0078** Intel® model, run the following command:

  ```
  cd <path_to_the_intruder-detector-python_directory>
  ./setup.sh
  ```
The model will be downloaded inside the following directory:

    /opt/intel/openvino/deployment_tools/open_model_zoo/tools/downloader/intel/person-vehicle-bike-detection-crossroad-0078/

### The Labels File

In order to work, this application requires a _labels_ file associated with the model being used for detection.  

All detection models work with integer labels, not string labels (e.g., For the **person-vehicle-bike-detection-crossroad-0078** model, the number 1 represents the class "person"). Each model must have a _labels_ file, which associates an integer, the label the algorithm detects, with a string denoting the human-readable label.

The _labels_ file is a text file containing all the classes/labels that the model can recognize, in the order that it was trained to recognize them (one class per line).

For the **person-vehicle-bike-detection-crossroad-0078** model, find the class file _labels.txt_ in the resources folder.

### The Config File

The _resources/config.json_ contains the path to the videos that will be used by the application, followed by the labels to be detected on those videos. All labels (intruders) defined will be detected on all videos.   

The _config.json_ file is of the form name/value pair, `video: <path/to/video>` and `label: <label>`   
The labels used in the _config.json_ file must coincide with the labels from the _labels.txt_ file.

Example of the _config.json_ file:

```
{

    "inputs": [
	    {
            "video": ["videos/video1.mp4","videos/video2.avi"],
            "label": [ "person", "bicycle"]
        }
    ]
}
```

The application can use any number of videos for detection, but the more videos the application uses in parallel, the more the frame rate of each video scales down. This can be solved by adding more computation power to the machine on which the application is running.

### Which Input video to use

The application works with any input video. Find sample videos for object detection [here](https://github.com/intel-iot-devkit/sample-videos/).  

For first-use, we recommend using the [person-bicycle-car-detection]( https://github.com/intel-iot-devkit/sample-videos/blob/master/person-bicycle-car-detection.mp4) video.The video is automatically downloaded to the `resources/` folder.
For example: <br>
The config.json would be:

```
{

    "inputs": [
	    {
            "video": "sample-videos/person-bicycle-car-detection.mp4",
            "label": [ "person", "bicycle", "car"]
        }
    ]
}
```
To use any other video, specify the path in config.json file

### Using the Camera instead of video

Replace the path/to/video in the _resources/config.json_  file with the camera ID, where the ID is taken from the video device (the number X in /dev/videoX).   

For example:

```
{

    "inputs": [
	    {
            "video": "0",
            "label": [ "person", "bicycle", "car"]
        }
    ]
}
```

On Ubuntu, list all available video devices with the following command:

```
ls /dev/video*
```
## Setup the environment
You must configure the environment to use the Intel® Distribution of OpenVINO™ toolkit one time per session by running the following command:

    source /opt/intel/openvino/bin/setupvars.sh 
    
__Note__: This command needs to be executed only once in the terminal where the application will be executed. If the terminal is closed, the command needs to be executed again.
    
### Run the Application

Change the current directory to the git-cloned application code location on your system:
```
cd <path_to_the_intruder-detector-python_directory>/application
```

To see a list of the various options:

```
python3 intruder_detector.py -h
```

 To run the application with the needed models:

```
python3 intruder_detector.py -lb ../resources/labels.txt -m /opt/intel/openvino/deployment_tools/open_model_zoo/tools/downloader/intel/person-vehicle-bike-detection-crossroad-0078/FP32/person-vehicle-bike-detection-crossroad-0078.xml
```
To run the application on sync mode, use **-f sync** as command line argument. By default, the application runs on async mode.

### Run on Different Hardware

A user can specify a target device to run on by using the device command-line argument `-d` followed by one of the values `CPU`, `GPU`,`MYRIAD` or `HDDL`.<br>
To run with multiple devices use -d MULTI:device1,device2. For example: `-d MULTI:CPU,GPU,MYRIAD`

#### Run on the CPU

Although the application runs on the CPU by default, this can also be explicitly specified through the `-d CPU` command-line argument:

```
python3 intruder_detector.py -lb ../resources/labels.txt -m /opt/intel/openvino/deployment_tools/open_model_zoo/tools/downloader/intel/person-vehicle-bike-detection-crossroad-0078/FP32/person-vehicle-bike-detection-crossroad-0078.xml -d CPU
```

#### Run on the Integrated GPU

* To run on the integrated Intel® GPU with floating point precision 32 (FP32), use the `-d GPU` command-line argument:

```
python3 intruder_detector.py -lb ../resources/labels.txt -m /opt/intel/openvino/deployment_tools/open_model_zoo/tools/downloader/intel/person-vehicle-bike-detection-crossroad-0078/FP32/person-vehicle-bike-detection-crossroad-0078.xml -d GPU
```
**FP32**: FP32 is single-precision floating-point arithmetic uses 32 bits to represent numbers. 8 bits for the magnitude and 23 bits for the precision. For more information, [click here](https://en.wikipedia.org/wiki/Single-precision_floating-point_format)<br>

* To run on the integrated Intel® GPU with floating point precision 16 (FP16):

```
python3 intruder_detector.py -lb ../resources/labels.txt -m /opt/intel/openvino/deployment_tools/open_model_zoo/tools/downloader/intel/person-vehicle-bike-detection-crossroad-0078/FP16/person-vehicle-bike-detection-crossroad-0078.xml -d GPU
```
**FP16**: FP16 is half-precision floating-point arithmetic uses 16 bits. 5 bits for the magnitude and 10 bits for the precision. For more information, [click here](https://en.wikipedia.org/wiki/Half-precision_floating-point_format)



#### Run on the Intel® Neural Compute Stick

To run on the Intel® Neural Compute Stick, use the ```-d MYRIAD``` command-line argument:

```
python3 intruder_detector.py -lb ../resources/labels.txt -m /opt/intel/openvino/deployment_tools/open_model_zoo/tools/downloader/intel/person-vehicle-bike-detection-crossroad-0078/FP16/person-vehicle-bike-detection-crossroad-0078.xml -d MYRIAD
```

**Note:** The Intel® Neural Compute Stick can only run FP16 models. The model that is passed to the application, through the `-m <path_to_model>` command-line argument, must be of data type FP16.

#### Run on the Intel® Movidius™ VPU
To run on the Intel® Movidius™ VPU, use the `-d HDDL` command-line argument:
```
python3 intruder_detector.py -lb ../resources/labels.txt -m /opt/intel/openvino/deployment_tools/open_model_zoo/tools/downloader/intel/person-vehicle-bike-detection-crossroad-0078/FP16/person-vehicle-bike-detection-crossroad-0078.xml -d HDDL
```
**Note:** The Intel® Movidius™ VPU can only run FP16 models. The model that is passed to the application, through the `-m <path_to_model>` command-line argument, must be of data type FP16.


#### Input Video Loop

By default, the application reads the input videos only once, and ends when the videos end.

The reference implementation provides an option to loop the video so that the input videos and application run continuously.

To loop the sample video, run the application with the `-lp true` command-line argument:

```
python3 intruder_detector.py -d CPU -lb ../resources/labels.txt -m /opt/intel/openvino/deployment_tools/open_model_zoo/tools/downloader/intel/person-vehicle-bike-detection-crossroad-0078/FP32/person-vehicle-bike-detection-crossroad-0078.xml -lp true
```

This looping does not affect live camera streams, as camera video streams are continuous and do not end.

## Use the Browser UI

The default application uses a simple user interface created with OpenCV. A web based UI, with more features is also provided with this application.<br>
From the working directory, run the application with `-ui true` command line argument. For example:
```
python3 intruder_detector.py -lb ../resources/labels.txt  -m /opt/intel/openvino/deployment_tools/open_model_zoo/tools/downloader/intel/person-vehicle-bike-detection-crossroad-0078/FP32/person-vehicle-bike-detection-crossroad-0078.xml -d CPU -ui true
```
Follow the readme provided [here](./UI) to run the web based UI. <br>
__Note:__ The browser UI does not support when the application is run using the option to loop the video.
