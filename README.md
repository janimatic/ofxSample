sample openfx plugin
--------------------

based on bmd Developper/OFX folder
* GainPlugin didn't work

replaced with https://github.com/AcademySoftwareFoundation/openfx/blob/main/Examples/Basic/basic.cpp

* cuda support is turned off (opencl on)

* opencl + openfx libs included
(openfx files, coming from bmd, might be outdated. But it seem to the official latest 1.4, slightly modified.
Copying files from https://github.com/AcademySoftwareFoundation/openfx gave me build errors
on ofxBinary loadLibrary under windows , that didn't occur with bmd provided files)

* added crossplatform CMake files (tested on windows / vs2019 x64 only)

* use the standard envvar OFX_PLUGIN_PATH to deploy the binary, if defined (but that's not finished, see below)

* copy the lib with ofx extention in the bundle directory

* copy the bundle directory in OFX_PLUGIN_PATH if defined else in standard ofx plugin location
That is not recommended to do an install during the build process, but auto-install so handy while developping c++ plugins...
