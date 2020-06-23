
#   :pushpin:  Building openCV 4 for linux!
###   :books: Used resources :
* [Youtube video](https://www.youtube.com/watch?v=DRH-EaIhOlc&list=PLS1lqxOwNjOaEFHEhbU_5uUZwrUquKTwZ)
* [Jupyter notebook on github](https://github.com/cesco345/StemApks/blob/master/TutorialsNotebook%20(1).ipynb)
* [linuxize.com](https://linuxize.com/post/how-to-install-opencv-on-ubuntu-18-04/)


**1**.  first download **openCV** library form their website :
[opencv.org](https://opencv.org/opencv-4-3-0/)
 you can also use this direct [download link](https://github.com/opencv/opencv/archive/4.3.0.zip) from **Github**
 ```bash
git clone https://github.com/opencv/opencv.git
```
**2**. unzip downloaded library and open terminal on it.

---
`another way from terminal : `
```bash
cd ~
wget -O opencv.zip https://github.com/Itseez/opencv/archive/4.3.0.zip
unzip opencv.zip
cd opencv-4.3.0
```
**2'**. `extra` download contrib modules
```bash
git clone https://github.com/opencv/opencv_contrib.git
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
`OR :`
in one command type :
```bash
sudo apt install build-essential cmake git pkg-config libgtk-3-dev \
    libavcodec-dev libavformat-dev libswscale-dev libv4l-dev \
    libxvidcore-dev libx264-dev libjpeg-dev libpng-dev libtiff-dev \
    gfortran openexr libatlas-base-dev python3-dev python3-numpy \
    libtbb2 libtbb-dev libdc1394-22-dev
```
---
**4**. now you need to build the library itself :
go where you installed the library folder `opencv`
```bash
cd opencv
mkdir build
cd build
cmake -D CMAKE_BUILD_TYPE=RELEASE \
    -D CMAKE_INSTALL_PREFIX=/usr/local \
    -D INSTALL_C_EXAMPLES=ON \
    -D INSTALL_PYTHON_EXAMPLES=ON \
    -D OPENCV_GENERATE_PKGCONFIG=ON \
    -D OPENCV_EXTRA_MODULES_PATH=~/opencv_build/opencv_contrib/modules \
    -D BUILD_EXAMPLES=ON ..
```
> N.B : 
>

 - `OPENCV_EXTRA_MODULES_PATH`

  =  path where you installed the opencv contrib repo
- `OPENCV_GENERATE_PKGCONFIG`

 =**ON** to let the system identify opencv version by adding it to pkg-config

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
**5**. make library
```bash
make -j4
```
Modify the `-j` flag according to your processor. If you do not know the number of cores your processor, you can find it by typing `nproc`.
:pushpin: this step takes the most time of them all

---
**6**. install it
```bash
sudo make install
```
---
in case of errors you might need the rest of steps
**7**.   edit some configuration files..
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
4. At the end of the file .. just append these magic lines :
>PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/local/lib/pkgconfig

>export PKG_CONFIG_PATH

---
**8**. check open cv version
```bash
pkg-config --modversion opencv4
```
```
output :

4.3.0
```
---
**9**. now try to compile a file using open cv with this line :
```bash
g++ csrt.cpp -o tracker `pkg-config --cflags --libs opencv4`
```
`pkg-config --cflags --libs opencv4` part is very important for the code to run don't remove it and it is essential to write it between two back ticks (\`)  
-> *you will find it above tab button and it is not a single quotation*

and watch out that it is `opencv4` not `opencv` to avoid errors like this :
> Package opencv was not found in the pkg-config search path.
Perhaps you should add the directory containing `opencv.pc'
to the PKG_CONFIG_PATH environment variable
No package 'opencv' found

or this :
>fatal error: opencv2/core/utility.hpp: No such file or directory
