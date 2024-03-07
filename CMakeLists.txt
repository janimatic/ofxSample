cmake_minimum_required(VERSION 3.10)
project(ofxSample VERSION 1.0)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
configure_file(ofxSampleConfig.h.in ofxSampleConfig.h)

file(GLOB ofxLib_HEADERS
    Support/include/*.h
    OpenFX-1.4/include/*.h
)
# do not add ofxsHWNDInteract.cpp...
set(ofxLib_SOURCES
    Support/Library/ofxsCore.cpp
    Support/Library/ofxsImageEffect.cpp
    Support/Library/ofxsInteract.cpp
    Support/Library/ofxsLog.cpp
    Support/Library/ofxsMultiThread.cpp
    Support/Library/ofxsParams.cpp
    Support/Library/ofxsProperty.cpp
    Support/Library/ofxsPropertyValidation.cpp
)
# add_library(ofxLib SHARED ${ofxLib_SOURCES} ${ofxLib_HEADERS})
add_library(ofxLib STATIC ${ofxLib_SOURCES} ${ofxLib_HEADERS})
target_include_directories(ofxLib PUBLIC
    Support/include/
    OpenFX-1.4/include/
)

file(GLOB ofxSample_HEADERS
    src/*.h
)
file(GLOB ofxSample_SOURCES
    src/*.cpp
)

# link_directories must be before add_executable/add_library
link_directories(OpenCL/lib)

# add_executable(ofxSample ${ofxSample_SOURCES} ${ofxSample_HEADERS}) 
add_library(ofxSample SHARED ${ofxSample_SOURCES} ${ofxSample_HEADERS})
# link the new ofxLib target with the ofxSample target
target_link_libraries( ofxSample PRIVATE 
        ofxLib
        OpenCL
)
target_include_directories(ofxSample PUBLIC
    Support/include/
    OpenFX-1.4/include/
    OpenCL/include
)

target_include_directories(ofxSample PUBLIC
                           "${PROJECT_BINARY_DIR}"
)


# using OpenCL we could just turn off cuda (see #define USE_CUDA)
# error LNK2019: "void __cdecl RunCudaKernel
# https://www.liftgammagain.com/forum/index.php?threads/dctls-and-resolve-ofx.7046/
# https://gist.github.com/gavinb/c993f71cf33d2354515c4452a3f8ef30

# use the standard envvar OFX_PLUGIN_PATH
# TODO : create the bundle
if (DEFINED ENV{OFX_PLUGIN_PATH})
set(OFX_PLUGIN_PATH $ENV{OFX_PLUGIN_PATH})
message( "The ofx plugin will be installed in OFX_PLUGIN_PATH : ${OFX_PLUGIN_PATH}")
add_custom_command(TARGET ofxSample POST_BUILD
    COMMAND ${CMAKE_COMMAND}  -E copy_if_different $<TARGET_FILE:ofxSample>
                                    ${OFX_PLUGIN_PATH}/$<TARGET_FILE_NAME:ofxSample>
)
else()
message( "The OFX_PLUGIN_PATH env var is not defined : the ofx plugin won't be installed")
endif()
