---
layout: page
title: Linux Image
permalink: /linuximage/
---
# Installing Ubuntu 22.04
Use the Raspberry Pi Imager to install Ubuntu Server 22.04 LTS (64-BIT)

Follow the instructions from the [Raspberry Pi Documentation](https://www.raspberrypi.com/documentation/computers/linux_kernel.html#cross-compiling-the-kernel) to cross-compile a newer version of their linux kernel [rpi-6.1.y](https://github.com/raspberrypi/linux). 
To make sure you are compiling this specific version, swap out their **Get the Kernel Sources** command with 
    
    git clone --depth=1 https://github.com/raspberrypi/linux --branch rpi-6.1.y

*Note this process is time consuming. 
I used a virtual machine running Ubuntu 22.04 and it took roughly an hour to compile. However, this is still far far far faster than attempting to compile on the Pi itself.* 

Once the cross-compilation is complete, insert your SD card into your Pi and boot it up. 
Be sure to run 

    sudo apt update 
    
and then

    sudo apt upgrade 

You will encounter a notification every time you run apt that your kernel is not the expected version and you should reboot. 
This is expected because we are using the rpi-6.1 kernel and Ubuntu 22.04 expects rpi-5.15.
To get rid of this error, you can remove the application that checks to see if you need to restart your system after a new installation. 
However, please do this with caution and reboot your Pi whenever you install applications or services that may impact system operations as apt will no longer run this check for you. 

    sudo apt-get purge needrestart 

# Installing a Desktop Environment
After testing both the full Ubuntu desktop as well as lightweight versions xubuntu and lubuntu, I recommend installing xubuntu as it boots without issue and provides a much faster experience than the full desktop. 
I was unable to get lubuntu to boot into the desktop environment, but [this blog](https://waldorf.waveform.org.uk/2020/ubuntu-desktops-on-the-pi.html) that Ubuntu cites highly recommends lubuntu, so do with that what you will. 

For xubuntu, I also tested both the full desktop and the core. I found the core version did everything I needed, and this way I was able to customize add on applications to save space. 
sudo apt-get install xubuntu-core^







Getting Pi Camera 3 to work on Linux

sudo apt install net-tools

**libcamera**

    sudo apt install python3-dev -y
    sudo apt install meson ninja-build pkg-config -y
    sudo apt install clang g++ -y
    sudo apt install libyaml-dev python3-yaml python3-ply python3-jinja2 -y
    sudo apt install libgnutls28-dev libssl-dev openssl -y
    sudo apt install libdw-dev libunwind-dev -y
    sudo apt install libudev-dev -y
    sudo apt install python3-sphinx doxygen graphviz texlive-latex-extra -y
    sudo apt install libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev -y
    sudo apt install libevent-dev -y
    sudo apt install libdrm-dev libdrm-dev libjpeg-dev libsdl2-dev -y
    sudo apt install qtbase5-dev libqt5core5a libqt5gui5 libqt5widgets5 qttools5-dev-tools libtiff-dev -y
sudo apt install liblttng-ust-dev python3-jinja2 lttng-tools -y
sudo apt-get install libgtest-dev -y

git clone https://github.com/raspberrypi/libcamera.git
cd libcamera
meson setup build -Dpycamera=enabled -Dv4l2=true
ninja -C build install



**libcamera-apps**
sudo apt install -y libepoxy-dev libjpeg-dev libtiff5-dev
sudo apt install -y libavcodec-dev libavdevice-dev libavformat-dev libswresample-dev
sudo apt install -y libboost-dev
sudo apt install -y cmake libboost-program-options-dev libexif-dev
sudo apt install -y libpng-dev
sudo apt install -y python3-pyqt5 python3-prctl libatlas-base-dev ffmpeg python3-pip python3-opengl
sudo pip3 install pyyaml ply

git clone https://github.com/raspberrypi/libcamera-apps.git
cd libcamera-apps
mkdir build
cd build
cmake .. -DENABLE_DRM=1 -DENABLE_X11=1 -DENABLE_QT=1 -DENABLE_OPENCV=0 -DENABLE_TFLITE=0
make -j4
sudo make install
sudo ldconfig

**kmsxx**
sudo apt install libfmt-dev libev-dev -y
git clone https://github.com/tomba/kmsxx.git
cd kmsxx 
git submodule update --init
meson build
ninja -C build


**picamera2**
pip3 install numpy --upgrade
pip3 install picamera2[gui]


export PYTHONPATH="${PYTHONPATH}:/usr/local/lib/aarch64-linux-gnu/python3.10/site-packages"

Go into /home/$USER/.local/lib/python3.10/site-packages/picamera2 and rename utils.py to utils_new.py
Change line 16 of request.py to `from .utils_new import convert_from_libcamera_type`

In line 25 of picamera2.py remove DrmPreview - I also removed from .previews _init_.py as well

In line 22 of picamera2.py change `import picamera2.utils as utils` to `import picamera2.utils_new as utils`

            preview_table = {Preview.NULL: NullPreview,
                             Preview.DRM: DrmPreview,
                             Preview.QT: QtPreview,
                             Preview.QTGL: QtGlPreview}
Remove DRM from above table

Eliminate display autodetect starting in line 504 and set preveiw = Preview.QT

opencv
Follow these instruction explicitly: https://qengineering.eu/install-opencv-4.5-on-raspberry-64-os.html 

tensorflow 
python3 -m pip install tensorflow


Other
sudo apt install v4l-utils
v4l2-ctl --list-devices
sudo apt install i2c-tools
sudo apt install rpi.gpio-common

**Screensaver Issue**

    sudo apt install XScreensaver
    sudo apt purge xfce4-screensaver
