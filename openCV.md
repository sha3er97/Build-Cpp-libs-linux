
#   :pushpin:  Building openCV 4 for linux!
###   :books: Used resources :
* [Youtube video](https://www.youtube.com/watch?v=DRH-EaIhOlc&list=PLS1lqxOwNjOaEFHEhbU_5uUZwrUquKTwZ)
* [Jupyter notebook on github](https://github.com/cesco345/StemApks/blob/master/TutorialsNotebook%20(1).ipynb)


**1**.  first download **openCV** library form their website :
[opencv.org](https://opencv.org/opencv-4-3-0/)
 you can also use this direct [download link](https://github.com/opencv/opencv/archive/4.3.0.zip) from **Github**
 
**2**. unzip downloaded library and open terminal on it.

---
`or from terminal : `
```bash
cd ~
wget -O opencv.zip https://github.com/Itseez/opencv/archive/4.3.0.zip
unzip opencv.zip
cd opencv-4.3.0
```
---
**3**.  then make sure you have some prequisites  :

`basic development environment:`
```bash
sudo apt-get install build-essential cmake pkg-config unzip
```
`opencv dependencies:`
```bash
sudo apt-get install libopencv-dev libgtk-3-dev libdc1394-22 libdc1394-22-dev libjpeg-dev  
```
```bash
sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev libxine2-dev
sudo apt-get install libv4l-dev libtbb-dev libfaac-dev libmp3lame-dev libtheora-dev 

sudo apt-get install libvorbis-dev libxvidcore-dev v4l-utils libopencore-amrnb-dev libopencore-amrwb-dev

sudo apt-get install libjpeg8-dev libx264-dev libatlas-base-dev gfortran
```
---
**4**. now you need to build the library itself :
```bash
mkdir build
cd build
cmake -D CMAKE_BUILD_TYPE=RELEASE \
      -D CMAKE_INSTALL_PREFIX=/usr/local \
      -D WITH_V4L=ON ..
```
:arrow_forward:  read :one: for possible errors  
```bash
make
sudo make install
```
:one: possible error in making after wrong previous make
>"FATAL: In-source builds are not allowed.
You should create separate directory for build files."

#### `Solution:`
```bash
rm ../CMakeCache.txt
(cmake again)
```
:arrow_forward: **make sure that you solved the problem before removing CMake cache**

---
**5**. finally you need to edit some configuration files..
1.  Create *opencv.conf* from terminal :
```bash
sudo nano /etc/ld.so.conf.d/opencv.conf
```
2. Paste this line to the file and save
 >**/usr/local/lib**
 
 save configuration from terminal :
```bash
sudo ldconfig
```
3. Edit *bash.bashrc* to add your new configurations
```bash
sudo nano /etc/bash.bashrc
```
5. At the end of the file .. just append these magic lines :
>PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/local/lib/pkgconfig

>export PKG_CONFIG_PATH

---
**6**. now try to compile a file using open cv with this line :
```bash
g++ file.cpp -o out `pkg-config opencv --cflags --libs`
```
`pkg-config opencv --cflags --libs` part is very important for the code to run don't remove it and it is essential to write it between two back ticks (\`)  
-> *you will find it above tab button and it is not a single quotation*

> Written with [StackEdit](https://stackedit.io/).
