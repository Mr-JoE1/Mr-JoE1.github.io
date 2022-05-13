---
layout: post
title: Cross Compile QT5.15 for Raspberry Pi Zero W 
categories: [Linux]
tags: [embedded, linux, ]
comments: true
---

Qt is your best choice if you want to build a professional GUI for embedded Applications.
So we gonna cross compile open source 'Qt5.15.x' for 'Raspberry pi Zero W'

## Compiling Setup:
  - Host Machine : Ubunut 20.4
  - Target Machine : Raspi Zero W with [2020-02-13-raspbian-buster-lite](https://downloads.raspberrypi.org/raspbian_lite/images/raspbian_lite-2020-02-14/2020-02-13-raspbian-buster-lite.zip)
  - Qtcreator installed on the Host Machine 
  
  
  
## Preparing Rspi-zero-w OS:
  - flash the sd card with the buster img.
  - after flashing is done, power it up with connected keyboard and display
  - you need to enable ssh , connect to your wifi:
      ```cs
            sudo raspi-config
      ```
  - install network file system to remotely share raspi file system with the Host machine 
      ```cs
            sudo apt-get update 
            sudo apt-get install nfs-kernel-server
            # confign nfs server
            # add this line to  /etc/exports
            /    *(rw,nohide,insecure,no_subtree_check,async,no_root_squash)
            service nfs-kernel-server restart
            
      ```
      
   - install all dependencies on raspi zero w 
      ```cs
           apt install build-essential libfontconfig1-dev libdbus-1-dev libfreetype6-dev libicu-dev libinput-dev libxkbcommon-dev libsqlite3-dev libssl-dev libpng-dev libjpeg-dev libglib2.0-dev libgbm-dev libraspberrypi-dev libudev-dev libinput-dev libts-dev libxcb-xinerama0-dev libxcb-xinerama0
            
      ```    
   - now your raspi is read the rest of work is done on the host, just keep raspi powered up and connected to your local network


## Preparing the Host Machine - Ubuntu
  - Remotely mount raspi root file system on the host machine 
       
  ```cs
          sudo apt-get install nfs-common
          # make mounting point for the raspi file system 
          mkdir /zeroQt
          mkdir /zeroQt/sysroot
          # use your raspi ip 
          mount 192.168.x.x:/ /zeroQt/sysroot
  ```
  
   - now check that raspi root fs is mounted correctelly , cd to /zeroQt/sysroot and list files to find the rootfs 
   - download and extract qt5.15 open source
   ```cs
           cd /zeroQt
           wget https://qt-mirror.dannhauer.de/archive/qt/5.15/5.15.0/single/qt-everywhere-src-5.15.0.zip
           # extract the source file in zeroQt directory               
   ```
        
  - Download the cross tool-chain
  - it's important to know that all gcc moderen compilers stoped supporting armV6 which is the core cpu of raspi zero 
    and even if forced the commpiler to build for arm6 it will ignore the flag and compile for arm7 
    so we gonna use custom gcc8 comiler which support both arm6 and arm7 https://github.com/Pro/raspi-toolchain
   
   ```cs
          cd /opt/
          wget https://github.com/Pro/raspi-toolchain/releases/latest/download/raspi-toolchain.tar.gz
          sudo tar xfz raspi-toolchain.tar.gz --strip-components=1 -C /opt             
   ```
   
  - now we are ready , but let's fix some bugs before moving forward !

## Fixing compiler and qt5.15 bugs
  - bug-1: make file use different names for raspi video core libraries, so lets correct the names 
      connect to raspi ssh and dow the following
      
    ```cs
          ssh pi@192.168.x.x
          cd /opt/vc/lib
           sudo cp -PR libbrcmEGL.so libEGL.so
           sudo cp -PR libbrcmOpenVG.so libOpenVG.so
           sudo cp -PR libbrcmGLESv2.so libGLESv2.so
     ```
     
  - bug-2: in Qt sources qtdeclarative/src/3rdparty/masm/wtf/OSAllocatorPosix.cpp, this files uses a variable 'PATH_MAX' 
    (which not present in our custom toolchain ?) so we gonna modifiy this file and hardcoded the value to 4096
        ```cs        
             char buf[4096]; 
        ```
  - Now we are ready to start config and cross compile qt for raspi zero

## Configure Qt5.15 source file 

    ```cs        
         # cd to the extracted source file 
          cd /zeroQt/qt-everywhere-src-5.15.0/
         # run the following configure
         ./configure -release -opensource -confirm-license -opengl es2 -device linux-rasp-pi-g++ -device-option CROSS_COMPILE=/opt/cross-pi-gcc/bin/arm-linux-gnueabihf- -sysroot /zeroQt/sysroot -prefix /usr/local/qt-5.15.0-rpi -make libs  -no-use-gold-linker -skip qtwayland -skip qtlocation -skip qtscript -no-gbm 
    ```
    
  - let's explaint some importnant parameters :
  -  -device linux-rasp-pi-g++  >>  this to select the target device confiuration
  -  CROSS_COMPILE=/opt/cross-pi-gcc/bin/arm-linux-gnueabihf-  >>  the path to your cross compiler on the host 
  -  -sysroot /zeroQt/sysroot >>  the path to your target mounted root file system 
  -  -prefix /usr/local/qt-5.15.0-rpi >> the deployment distnation where the final binarries will be copied to the target 
  -  you can use -skip to disable any qt addons you don't need
  -  check the config command output to make sure all you need is marked as yes before you start commiling 

## compilling and installing the qt15.5 on the target 

    - if the configuration step endded successfully , you are ready to go with make 
    
         ```cs
              make -j4 2>&1 | tee ../make$(date +"%Y-%m-%d_%H-%M").log
         ```
         
    - if you make endded successfully , lets install 
    
         ```cs
               make install 2>&1 | tee ../install$(date +"%Y-%m-%d_%H-%M").log
         ```
         
  - my application does not require desktop appliction and that is very suitable for such a tiny raspi zero 
Install Qtcreator : only needed for development your application and remote deployment 

         ```cs
             sudo apt install qtcreator
         ```
         
  - Congratulations , we are ready to start develop you application in native or remote deployment mode 

## configure QtCreator for remote deployment for remote embedded devices 
check qt documentation for the details :
https://doc.qt.io/qtcreator/creator-deployment-embedded-linux.html



