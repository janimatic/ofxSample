sample openfx plugin
--------------------

based on bmd Developper/OFX folder
* GainPlugin didn't work

replaced with https://github.com/AcademySoftwareFoundation/openfx/blob/main/Examples/Basic/basic.cpp

* cuda support is turned off (opencl on)

* opencl + openfx libs included

Openfx files, coming from bmd, might be outdated. But it seem to use the official latest 1.4, slightly modified.
Copying files from https://github.com/AcademySoftwareFoundation/openfx gave me build errors
on ofxBinary loadLibrary under windows , that didn't occur with bmd provided files.

* added crossplatform CMake files (tested on windows / vs2019 x64 only)

* use the standard envvar OFX_PLUGIN_PATH to deploy the binary, if defined (but that's not finished, see below)

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

* On linux (and apple?)
the opencl libs should be installed with :
```
sudo apt install ocl-icd-opencl-dev
```

#### 2. Download the XClip source code
You can download source code archives from [GitHub](https://www.github.com/janimatic/XClip) or use `git` to clone the repository.
```
> git clone https://www.github.com/janimatic/XClip
Cloning into 'xclip'...
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

![cmake linux workflow](https://github.com/janimatic/xclip/actions/workflows/cmake-linux-platform.yml/badge.svg)
![cmake windows workflow](https://github.com/janimatic/xclip/actions/workflows/cmake-windows-platform.yml/badge.svg)
