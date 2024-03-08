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
# link_directories(OpenCL/lib)
# link_directories(OpenCL/lib/x86_64)


set(OpenCL_INCLUDE_DIR ${PROJECT_SOURCE_DIR}/OpenCL/include)
message( "OpenCL_INCLUDE_DIR : ${OpenCL_INCLUDE_DIR}")
if(WIN32)
set(OpenCL_LIBRARY ${PROJECT_SOURCE_DIR}/OpenCL/lib/x86_64/OpenCL.lib)
message( "OpenCL_LIBRARY : ${OpenCL_LIBRARY}")
elseif(UNIX)
set(OpenCL_LIBRARY ${PROJECT_SOURCE_DIR}/OpenCL/lib/x86_64/libOpenCL.so)
message( "OpenCL_LIBRARY : ${OpenCL_LIBRARY}")
elseif(APPLE)
set(OpenCL_LIBRARY ${PROJECT_SOURCE_DIR}/OpenCL/lib/x86_64/libOpenCL.so)
message( "OpenCL_LIBRARY : ${OpenCL_LIBRARY}")
else()
message( "OpenCL_LIBRARY : unknown platform...")
endif()

find_package(OpenCL REQUIRED)

# add_executable(${CMAKE_PROJECT_NAME} ${ofxSample_SOURCES} ${ofxSample_HEADERS}) 
add_library(${CMAKE_PROJECT_NAME} SHARED ${ofxSample_SOURCES} ${ofxSample_HEADERS})

target_link_libraries(${CMAKE_PROJECT_NAME} ofxLib ${OpenCL_LIBRARY})

target_include_directories(${CMAKE_PROJECT_NAME} PUBLIC
    Support/include/
    OpenFX-1.4/include/
    "${OpenCL_INCLUDE_DIR}"
)

target_include_directories(${CMAKE_PROJECT_NAME} PUBLIC
                           "${PROJECT_BINARY_DIR}"
)


# using OpenCL we could just turn off cuda (see #define USE_CUDA)
# error LNK2019: "void __cdecl RunCudaKernel
# https://www.liftgammagain.com/forum/index.php?threads/dctls-and-resolve-ofx.7046/
# https://gist.github.com/gavinb/c993f71cf33d2354515c4452a3f8ef30

# create the bundle
# copied from https://github.com/AcademySoftwareFoundation/openfx/blob/36cf59759b8262250a3e2ca23e021e12e6ab9396/cmake/OpenFX.cmake#L25
if(APPLE)
  set(PLUGINDIR "/Library/OFX/Plugins")
  set(ARCHDIR "MacOS")
elseif(WIN32)
  set(PLUGINDIR "C:/Program Files (x86)/Common Files/OFX/Plugins")
  set(ARCHDIR "Win64")
elseif(UNIX)
  set(PLUGINDIR "/usr/OFX/Plugins")
  set(ARCHDIR "Linux-x86-64")
else()
  set(PLUGINDIR "/unknown-os")
  set(ARCHDIR "unknown-arch")
endif()

set_target_properties(${CMAKE_PROJECT_NAME} PROPERTIES SUFFIX ".ofx" PREFIX "")
if (DEFINED ENV{OFX_PLUGIN_PATH})
# use the standard envvar OFX_PLUGIN_PATH (custom plugin location) this could avoid write permissions problems
# set(OFX_PLUGIN_PATH $ENV{OFX_PLUGIN_PATH})
# avoid backslashes under windows...
file(TO_CMAKE_PATH $ENV{OFX_PLUGIN_PATH} OFX_PLUGIN_PATH)
message( "The ofx plugin will be installed in OFX_PLUGIN_PATH : ${OFX_PLUGIN_PATH}/${CMAKE_PROJECT_NAME}.ofx.bundle/Contents/${ARCHDIR}")
add_custom_command(TARGET ${CMAKE_PROJECT_NAME} POST_BUILD
    COMMAND ${CMAKE_COMMAND}  -E copy_if_different $<TARGET_FILE:${CMAKE_PROJECT_NAME}>
                                    ${OFX_PLUGIN_PATH}/${CMAKE_PROJECT_NAME}.ofx.bundle/Contents/${ARCHDIR}/$<TARGET_FILE_NAME:${CMAKE_PROJECT_NAME}>
)
# install(TARGETS ${TARGET} DESTINATION "${OFX_PLUGIN_PATH}/${TARGET}.ofx.bundle/Contents/${ARCHDIR}")
else()
# use the PLUGINDIR (standard plugin location)
message( "The OFX_PLUGIN_PATH env var is not defined : the ofx plugin will be installed in standard location : ${PLUGINDIR}/${CMAKE_PROJECT_NAME}.ofx.bundle/Contents/${ARCHDIR}")
add_custom_command(TARGET ${CMAKE_PROJECT_NAME} POST_BUILD
    COMMAND ${CMAKE_COMMAND}  -E copy_if_different $<TARGET_FILE:${CMAKE_PROJECT_NAME}>
                                    ${PLUGINDIR}/${CMAKE_PROJECT_NAME}.ofx.bundle/Contents/${ARCHDIR}/$<TARGET_FILE_NAME:${CMAKE_PROJECT_NAME}>
)
# install(TARGETS ${TARGET} DESTINATION "${PLUGINDIR}/${TARGET}.ofx.bundle/Contents/${ARCHDIR}")
endif()
