# NVIDIA Jetson IMX477 HQ RPI V3 camera driver

This driver has been developed by NVIDIA Corporation, RidgeRun Engineering and Leopard Imaging Inc. as a collaborative initiative in order to release the first version of the Sony IMX477 sensor driver for the Raspberry Pi HQ camera.

Supports the following Jetson platforms:
* Jetson Xavier NX
* Jetson Nano

## Repository structure

In this repository you will find the following structure:
```
.
├── li-camera-calibration-files
│   └── camera_overrides.isp
├── patches_nano
│   ├── 4.4.3_l4t32.4.3_rbpv3_imx477_support.patch
│   └── series
├── patches_nx
│   ├── 4.4.3_l4t32.4.3_rbpv3_imx477_support.patch
│   └── series
└── README.md
```
where:

* `4.4.3_l4t32.4.3_rbpv3_imx477_support.patch` is the patch to be applied on the JetPack 4.4 sources in order to add support for the IMX477 camera sensor in the Jetson Xavier NX and Jetson Nano.
* `series` is a file containing the patch name in order to apply it by using the quilt tool.
* `camera_overrides.isp` is a file which contains a set of calibration parameters for the Image Signal Processor of the HQ camera.

## JetPack 4.4 installation instructions

You can download and install the JetPack 4.4 by following the instructions below:

* [JetPack 4.4 download and installation instructions](https://developer.ridgerun.com/wiki/index.php?title=Raspberry_Pi_HQ_camera_IMX477_Linux_driver_for_Jetson#Download_the_JetPack_4.4)

## Driver Installation instructions

There are two options to install the driver:

### OPTION A: Installing the kernel and dtb debians (Recommended)

This is the easiest and fastest way to install the driver. In order to install the debian packages you just need to perform the following instructions:

* [Installing the kernel and dtb debians](https://developer.ridgerun.com/wiki/index.php?title=Raspberry_Pi_HQ_camera_IMX477_Linux_driver_for_Jetson#Installing_the_Driver_-_Option_A:_Debian_Packages_.28Recommended.29)

#### Jetson Nano

NOTE: check that your kernel version is the same as those in the driver name (4.9.10). Installation of the drivers bricked the device with the latest Jetson Nano ISO image (Jetpack 4.4.2) but worked using the ISO imae with Jetpack 4.4

```sh
# Download the packages
wget https://github.com/Westray-AI/NVIDIA-Jetson-IMX477-RPIV3/releases/download/nano-deb-4.9.140/nvidia-l4t-kernel-dtbs_4.9.140-tegra-32.4.3-20200625213809_arm64.deb
wget https://github.com/Westray-AI/NVIDIA-Jetson-IMX477-RPIV3/releases/download/nano-deb-4.9.140/nvidia-l4t-kernel_4.9.140-tegra-32.4.3-20200625213809_arm64.deb

# Install the debian packages in your board:
sudo apt-get install --reinstall ./nvidia-l4t-kernel_4.9.140-tegra-32.4.3-20200625213809_arm64.deb
sudo apt-get install --reinstall ./nvidia-l4t-kernel-dtbs_4.9.140-tegra-32.4.3-20200625213809_arm64.deb

# Enable the dtb changes by modifying the /boot/extlinux/extlinux.conf file:
echo "FDT /boot/dtb/tegra210-p3448-0000-p3449-0000-b00.dtb" | sudo tee -a /boot/extlinux/extlinux.conf

# Reboot your board:
sudo reboot

```

### OPTION B: Applying the patches on the sources

In order to apply the patch on the JetPack 4.4 sources with Xavier NX and Nano support, you must perform the following instructions:
 
* [JetPack 4.4 sources download instructions](https://developer.ridgerun.com/wiki/index.php?title=Raspberry_Pi_HQ_camera_IMX477_Linux_driver_for_Jetson#Download_the_JetPack_4.4_sources)
* [Patch instructions](https://developer.ridgerun.com/wiki/index.php?title=Raspberry_Pi_HQ_camera_IMX477_Linux_driver_for_Jetson#Patch_instructions)
* [Kernel build instructions](https://developer.ridgerun.com/wiki/index.php?title=Raspberry_Pi_HQ_camera_IMX477_Linux_driver_for_Jetson#Kernel_build_instructions)
* [Flash the Jetson](https://developer.ridgerun.com/wiki/index.php?title=Raspberry_Pi_HQ_camera_IMX477_Linux_driver_for_Jetson#Flash_the_Jetson)

## ISP camera overrides installation

Visit the [Custom ISP Configuration page](https://developer.ridgerun.com/wiki/index.php?title=JetsonTX2/HowTo%27s/NVIDIA_Jetson_ISP_Control#Custom_ISP_Configuration) in order to learn how to use the __camera_overrides.isp__ file to configure the ISP.

## Supported features

### Resolutions and framerates

* 1920x1080 @ 60fps
* 4032x3040 @ 30fps

### Controls

* Gain
* Exposure
* Framerate
* Group Hold

## Example pipelines

### Display

* 1920x1080

```
SENSOR_ID=0 # 0 for CAM0 and 1 for CAM1 ports
FRAMERATE=60 # Framerate can go from 2 to 60 for 1920x1080 mode
gst-launch-1.0 nvarguscamerasrc sensor-id=$SENSOR_ID ! "video/x-raw(memory:NVMM),width=1920,height=1080,framerate=$FRAMERATE/1" ! nvvidconv ! nvoverlaysink
```

* 4032x3040

```
SENSOR_ID=0 # 0 for CAM0 and 1 for CAM1 ports
FRAMERATE=30 # Framerate can go from 2 to 30 for 4032x3040 mode
gst-launch-1.0 nvarguscamerasrc sensor-id=$SENSOR_ID ! "video/x-raw(memory:NVMM),width=4032,height=3040,framerate=$FRAMERATE/1" ! nvvidconv ! "video/x-raw(memory:NVMM),width=1920,height=1080,framerate=$FRAMERATE/1" ! nvoverlaysink
```

### MP4 Recording

* 1920x1080

```
SENSOR_ID=0 # 0 for CAM0 and 1 for CAM1 ports
FRAMERATE=60 # Framerate can go from 2 to 60 for 1920x1080 mode
gst-launch-1.0 -e nvarguscamerasrc sensor-id=$SENSOR_ID ! "video/x-raw(memory:NVMM),width=1920,height=1080,framerate=$FRAMERATE/1" ! nvv4l2h264enc ! h264parse ! mp4mux ! filesink location=rpi_v3_imx477_cam$SENSOR_ID.mp4
```

* 4032x3040

```
SENSOR_ID=0 # 0 for CAM0 and 1 for CAM1 ports
FRAMERATE=30 # Framerate can go from 2 to 30 for 4032x3040 mode
gst-launch-1.0 -e nvarguscamerasrc sensor-id=$SENSOR_ID ! "video/x-raw(memory:NVMM),width=4032,height=3040,framerate=$FRAMERATE/1" ! nvv4l2h264enc ! h264parse ! mp4mux ! filesink location=rpi_v3_imx477_cam$SENSOR_ID.mp4
```

### JPEG snapshots

* 1920x1080

```
SENSOR_ID=0 # 0 for CAM0 and 1 for CAM1 ports
FRAMERATE=60 # Framerate can go from 2 to 60 for 1920x1080 mode
NUMBER_OF_SNAPSHOTS=20
gst-launch-1.0 -e nvarguscamerasrc num-buffers=$NUMBER_OF_SNAPSHOTS sensor-id=$SENSOR_ID ! "video/x-raw(memory:NVMM),width=1920,height=1080,framerate=$FRAMERATE/1" ! nvjpegenc ! multifilesink location=%03d_rpi_v3_imx477_cam$SENSOR_ID.jpeg
```

* 4032x3040

```
SENSOR_ID=0 # 0 for CAM0 and 1 for CAM1 ports
FRAMERATE=30 # Framerate can go from 2 to 30 for 4032x3040 mode
NUMBER_OF_SNAPSHOTS=20
gst-launch-1.0 -e nvarguscamerasrc num-buffers=$NUMBER_OF_SNAPSHOTS sensor-id=$SENSOR_ID ! "video/x-raw(memory:NVMM),width=4032,height=3040,framerate=$FRAMERATE/1" ! nvjpegenc ! multifilesink location=%03d_rpi_v3_imx477_cam$SENSOR_ID.jpeg
```





