# OpenCV-Cuda-Using-JNI-wrapper-in-IntelliJ  
This repository contains instructions to run OpenCV Cuda using JNI wrapper in IntelliJ  

# Setup IntelliJ environment for OpenCV with pre-built libraries of CUDA:  

**1. Install Nvidia Driver for your system:**    

a. Follow pre-installation at https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html   
b. Follow instructions available at the link: https://towardsdatascience.com/opencv-cuda-aws-ec2-no-more-tears-60af2b751c46 to install Nvidia Driver and setup required pre-requisite libraries for OpenCV files. thions:

Errors and Solutions:   
1. The following signatures couldn't be verified because the public key is not available: NO_PUBKEY FEEA9169307EA071 NO_PUBKEY 8B57C5C2836F4BEB   
> curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key --keyring /usr/share/keyrings/cloud.google.gpg add -   

2. Which Nvidia Driver to install:   
> ubuntu-drivers devices   
> sudo apt-get install nvidia-driver-***   

c. To generate the .so file and .jar file required for implementing OpenCV in JAVA add a few flags in the CMake command and thus, the final Cmake looks like: 
> cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_C_COMPILER=/usr/bin/gcc-7 -D CMAKE_INSTALL_PREFIX=/usr/local -D INSTALL_PYTHON_EXAMPLES=ON -D INSTALL_C_EXAMPLES=OFF -D **BUILD_SHARED_LIBRARY**=OFF -D BUILD_TESTS=OFF -D JAVA_AWT_INCLUDE_PATH=/usr/lib/jvm/java-8-openjdk-amd64/include -D JAVA_AWT_LIBRARY=/usr/lib/jvm/java-8-openjdk-amd64/include/jawt.h -D JAVA_INCLUDE_PATH=/usr/lib/jvm/java-8-openjdk-amd64/include -D JAVA_INCLUDE_PATH2=/usr/lib/jvm/java-8-openjdk-amd64/include/linux -D JAVA_JVM_LIBRARY=/usr/lib/jvm/java-8-openjdk-amd64/include/jni.h -D ANT_EXECUTABLE=/usr/bin/ant -D WITH_TBB=ON -D WITH_CUDA=ON -D BUILD_opencv_cudacodec=OFF -D ENABLE_FAST_MATH=1 -D BUILD_PERF_TESTS=OFF -D CUDA_FAST_MATH=1 -D WITH_CUBLAS=1 -D WITH_V4L=ON -D WITH_QT=OFF -D WITH_OPENGL=ON -D WITH_GSTREAMER=ON -D OPENCV_GENERATE_PKGCONFIG=ON -D OPENCV_PC_FILE_NAME=opencv.pc -D OPENCV_ENABLE_NONFREE=ON -D OPENCV_PYTHON3_INSTALL_PATH=\~/.virtualenvs/opencv_cuda/lib/python3.6/site-packages -D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib/modules -D PYTHON_EXECUTABLE=\~/.virtualenvs/opencv_cuda/bin/python -D BUILD_EXAMPLES=ON -D WITH_CUDNN=ON -D OPENCV_DNN_CUDA=ON -D **CUDA_ARCH_BIN**=3.5 .. 

Error Build output check failed:   
> Regex: 'command[- ]line option .* is valid for .* but not for C\+\+'   
Try chaning the gcc-Version at _-D CMAKE_C_COMPILER=/usr/bin/gcc-7_    
Or else, try by removing _-D WITH_CUDNN=ON -D OPENCV_DNN_CUDA=ON_   


d. Follow these instructions for rest installation:  
  
> make -j8   
> sudo make install  
> sudo /bin/bash -c 'echo "/usr/local/lib" >> /etc/ld.so.conf.d/opencv.conf'  
> sudo ldconfig  
> sudo cp -r ~/.virtualenvs/opencv_cuda/lib/python3.6/site-packages/cv2 /usr/local/lib/python3.6/dist-packages  
> sudo nano /usr/local/lib/python3.6/dist-packages/cv2/config-3.6.py and add:   
>> PYTHON_EXTENSIONS_PATHS = [os.path.join('/usr/local/lib/python3.8/dist-packages/cv2', 'python-3.8')] + PYTHON_EXTENSIONS_PATHS    

_To check installation and version of the NVCC compiler:_  
`nvcc -V`  
_To check installation and version of the OpenCV in your system:_  
`python3    
import cv2    
cv2.__version__`    

d. .so file can be found in opencv/build/lib & .jar file can be found in opencv/build/bin  
e. Copy paste both these files in the library directory of your IntelliJ project. If you don't have one for your project, it is always advisable to create one  

# Writing a JNI wrapper for running C++:
Creating a JNI wrapper involves 3 steps:  

**1. Generation of .h file for the wrapper:**  

Write a java class in your wrapper package with native libraries and instantiate various methods you want.   
*For an example check: *  

Then, follow the steps for .h file generation:  

   a. File->Settings->External Tools  
   b. Click the + button for the "Edit Tool" dialog  
   c. The following are the form name/value pairs I used:  
> Name: javah  
> Group: Java  
> Description: Java Native Interface C Header and Stub File Generator  
> Options: Check All  
> Show in: Check All  
> Tool Settings...  
> Program: {Path to javah: Open a terminal and type: locate bin/javah} e.g._/usr/lib/jvm/java-8-openjdk-amd64/bin/javah_  
> Parameters: -jni -v -d $FileDir$ $FileClass$            
> Working directory: $SourcepathEntry$  
> Click OK, Click OK 
 
   d. Navigate to your java class with the native method    
   e. With the java class being shown in the editor, go to Tools->Java->javah  

*Notice the .h file has been generated in the same directory as your class.*  

**2. Write the .cpp file for JNI wrapper with OpenCV Algorithm implementation:**  

Define your various methods defined earlier in .cpp file for the wrapper and copy your c++ OpenCV implemenatation in the .cpp file.  
_Check an example .cpp file at:_  

**3. Generation of .so file:**  

   a. Open Terminal and run:  

> g++ -Wl,--enable-new-dtags -fPIC   
> -o {path for o/p file directory that contains the library for the IntelliJ environment}/{name of o/p file e.g_.home/wrapper/lib**wrapper**.so_}   
> -lc -shared   
> -I{path to jni.h e.g._/usr/lib/jvm/java-8-openjdk-amd64/include_}   
> -I{path to jni_md.h e.g._/usr/lib/jvm/java-8-openjdk-amd64/include/linux_}   
> {name of your cpp file e.g._com_morphle_wrapper_Cuda_libraries_Cudawrapper.cpp_}   
> \`pkg-config opencv --cflags --libs\`   

An example of the command to run:  
_g++ -Wl,--enable-new-dtags -fPIC -o /etc/libs/libCudaWrapperImpl.so -lc -shared -I/usr/lib/jvm/java-8-openjdk-amd64/include -I/usr/lib/jvm/java-8-openjdk-amd64/include/linux Cudawrapper.cpp \`pkg-config opencv --cflags --libs\`_  

Note the .so filenmae.    
Here, .so filename is e.g._lib**wrapper**.so_. The text marked in bold will be used in java class which is to be created in the next section.

# Try and Test by running an OpenCV program:    

1. Open IntelliJ and open your project.   
2. Create a new java file and create a constructor    
3. Add a line to load the just created .so library file: _`System.loadLibrary("wrapper");`_   
4. Proceed forward to use the various methods already defined in the .h and .cpp file earlier  

_Check an example .java file at:_  
