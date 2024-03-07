sample openfx plugin
--------------------

based on bmd Developper/OFX folder
* GainPlugin didn't work

replaced with https://github.com/AcademySoftwareFoundation/openfx/blob/main/Examples/Basic/basic.cpp

* cuda support is turned off (opencl on)

* opencl + openfx libs included (openfx files might be outdated ! comming from bmd)

* added crossplatform CMake files (tested on windows / vs2019 x64 only)

* use the standard envvar OFX_PLUGIN_PATH to deploy the binary, if defined (but that's not finished, see below)

TODO
----

* copy the lib with ofx extention
* copy in the bundle directory
* copy the bundle directory in OFX_PLUGIN_PATH if defined
