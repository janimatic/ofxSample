sample openfx plugin
--------------------

based on Blackmagic Resolve inlcuded examples found in Developper/OFX folder

* GainPlugin with cuda disabled

* To use the OpenCL in Resolve, force to use OpenCL instead of Cuda in the preferences :
Memory and GPU>GPU configuration>GPU processing mode > OpenCL

* cuda support is turned off (opencl on)

* openfx libs included

* opencl libs for windows included (see below for linux)

Openfx files, coming from bmd, seem to use the official latest 1.4, slightly modified.
Copying files from https://github.com/AcademySoftwareFoundation/openfx gave me build errors
on ofxBinary loadLibrary under windows , that didn't occur with bmd provided files.

* added crossplatform CMake files (github actions for windows and linux only, I couldn't make the macos 12 workflow to build)

* Check AUTO_DEPLOY in cmake if you find handy to auto install the plugin on build like I do...

The standard envvar OFX_PLUGIN_PATH will be used to deploy the binary, if defined,
otherwise the standard openfx plugin location (which might have permissions restrictions that makes deployment fails)

* copy the lib with ofx extention in the bundle directory

* copy the bundle directory in OFX_PLUGIN_PATH if defined else in standard ofx plugin location
That is not recommended to do an install during the build process, but auto-install so handy while developping c++ plugins...

Building
--------
#### 1. Install prerequisites
- Required:
    - C++ compiler:
        - gcc
        - Xcode
        - Microsoft Visual Studio (tested on vs2019)
    - CMake https://cmake.org/download/

* On windows
No action needed, the OpenCL 2.0 libs are provided

* On linux
the opencl libs should be installed with :
```
sudo apt install ocl-icd-opencl-dev
```

* On mac
I tried to install the opencl libs with :
```
brew install ocl-icd
```
but i still get a build error
```
make[2]: *** No rule to make target `/usr/lib/x86_64-linux-gnu/libOpenCL.so', needed by `libofxSample.dylib'.  Stop.
```

#### 2. Download the ofxSample  source code
You can download source code archives from [GitHub](https://www.github.com/janimatic/ofxSample) or use `git` to clone the repository.
```
> git clone https://www.github.com/janimatic/ofxSample
Cloning into 'ofxSample'...
```

#### 3. Run cmake
    
    Here is the procedure with CMake-gui under windows :

    Browse source : /ofxSample
    Browse build : /ofxSample/bin (That is the new folder, where the vs solution and projects will be generated)
    Configure
    You can choose the default generator (vs2019 in my case) and just clic finish
    Check AUTO_DEPLOY if you find handy to auto install the plugin on build like I do...
    Configure
    Generate
    Open the ofxSample solution (ofxSample.sln in my case) found in /ofxSample/bin
    Switch to Release mode
    Build all the projects

Github worflows
---------------

<img src="https://github.com/janimatic/ofxSample/actions/workflows/cmake-linux-platform.yml/badge.svg?branch=main&kill_cache=1" />
<img src="https://github.com/janimatic/ofxSample/actions/workflows/cmake-windows-platform.yml/badge.svg?branch=main&kill_cache=1" />
<img src="https://github.com/janimatic/ofxSample/actions/workflows/cmake-mac-platform.yml/badge.svg?branch=main&kill_cache=1" />


I couldn't make the macos 12 workflow to build so i disabled it... please contribute if you can...
