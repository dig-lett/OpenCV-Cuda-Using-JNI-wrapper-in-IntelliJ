# OpenCV-Cuda-Using-JNI-wrapper-in-IntelliJ  
This repository contains instructions to run OpenCV Cuda using JNI wrapper in IntelliJ  

# Setup IntelliJ environment for OpenCV with pre-built libraries of CUDA:  

1. Install Nvidia Driver for your system:  

2. Build OpenCV with CUDA and generate .so and .jar files:  


# Writing a JNI wrapper for running C++:
Creating a JNI wrapper involves 3 steps:  

**`1. Generation of .h file for the wrapper:`**  

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

**`2. Write the .cpp file for JNI wrapper with OpenCV Algorithm implementation:`**  

Define your various methods defined earlier in .cpp file for the wrapper and copy your c++ OpenCV implemenatation in the .cpp file.  
_Check an example .cpp file at:_  

**`3. Generation of .so file:`**  

   a. Open Terminal and run:  

> g++ -Wl,--enable-new-dtags -fPIC   
> -o {path for o/p file directory that contains the library for the IntelliJ environment}/{name of o/p file e.g.home/wrapper/lib**wrapper**.so }   
> -lc -shared   
> -I{path to jni.h e.g._/usr/lib/jvm/java-8-openjdk-amd64/include_}   
> -I{path to jni_md.h e.g._/usr/lib/jvm/java-8-openjdk-amd64/include/linux_}   
> {name of your cpp file e.g._com_morphle_wrapper_Cuda_libraries_Cudawrapper.cpp_}   
> \`pkg-config opencv --cflags --libs\`  

An example of the command to run:  
_g++ -Wl,--enable-new-dtags -fPIC -o /etc/libs/libCudaWrapperImpl.so -lc -shared -I/usr/lib/jvm/java-8-openjdk-amd64/include -I/usr/lib/jvm/java-8-openjdk-amd64/include/linux Cudawrapper.cpp \`pkg-config opencv --cflags --libs\`_  

# Try and Test by running an OpenCV program:  

