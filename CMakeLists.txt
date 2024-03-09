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

# On UNIX/APPLE, use sudo apt install ocl-icd-opencl-dev (like the github workflow)
set(OpenCL_INCLUDE_DIR ${PROJECT_SOURCE_DIR}/OpenCL/include)
message( "OpenCL_INCLUDE_DIR : ${OpenCL_INCLUDE_DIR}")
if(WIN32)
# for Windows operating system in general
set(OpenCL_LIBRARY ${PROJECT_SOURCE_DIR}/OpenCL/lib/x86_64/OpenCL.lib)
message( "OpenCL_LIBRARY : ${OpenCL_LIBRARY}")
elseif(UNIX AND NOT APPLE)
# for Linux, BSD, Solaris, Minix
# set(OpenCL_LIBRARY ${PROJECT_SOURCE_DIR}/OpenCL/lib/x86_64/libOpenCL.so)
set(OpenCL_LIBRARY /usr/lib/x86_64-linux-gnu/libOpenCL.so)
message( "OpenCL_LIBRARY : ${OpenCL_LIBRARY}")
elseif(APPLE)
# for MacOS X or iOS, watchOS, tvOS (since 3.10.3)
# set(OpenCL_LIBRARY ${PROJECT_SOURCE_DIR}/OpenCL/lib/x86_64/libOpenCL.so)
 # For compilers to find ocl-icd you may need to set:
  # export LDFLAGS="-L/usr/local/opt/ocl-icd/lib"
  # export CPPFLAGS="-I/usr/local/opt/ocl-icd/include"
set(OpenCL_LIBRARY /usr/local/opt/ocl-icd/lib/libOpenCL.so)
message( "OpenCL_LIBRARY : ${OpenCL_LIBRARY}")
else()
message( "OpenCL_LIBRARY : unknown platform...")
endif()

find_package(OpenCL REQUIRED)


# target_link_libraries(${CMAKE_PROJECT_NAME} ofxLib "${OpenCL_LIBRARY}") #  error: No rule to make target
# get_filename_component(OpenCL_LIBRARY_DIR ${OpenCL_LIBRARY} DIRECTORY )
# get_filename_component(OpenCL_LIBRARY_NAME ${OpenCL_LIBRARY} NAME )
# target_link_directories(${CMAKE_PROJECT_NAME} PUBLIC ${OpenCL_LIBRARY_DIR})
# target_link_libraries(${CMAKE_PROJECT_NAME} ofxLib ${OpenCL_LIBRARY_NAME}) # cannot find -lOpenCL: No such file or directory
add_library(OpenCLlib SHARED IMPORTED)
# set_target_properties(OpenCLlib PROPERTIES IMPORTED_LOCATION "${OpenCL_LIBRARY}")
# https://discourse.cmake.org/t/imported-implib-not-set-for-imported-target/5398/6
set_target_properties(OpenCLlib PROPERTIES 
    IMPORTED_LOCATION_RELEASE "${OpenCL_LIBRARY}"
    IMPORTED_IMPLIB "${OpenCL_LIBRARY}"
)

# add_executable(${CMAKE_PROJECT_NAME} ${ofxSample_SOURCES} ${ofxSample_HEADERS}) 
add_library(${CMAKE_PROJECT_NAME} SHARED ${ofxSample_SOURCES} ${ofxSample_HEADERS})

# set_property(TARGET OpenCLlib PROPERTY IMPORTED_LOCATION "${OpenCL_LIBRARY}")
target_link_libraries(${CMAKE_PROJECT_NAME} ofxLib OpenCLlib)

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

option(AUTO_DEPLOY "Install the plugin on build" OFF)

if(AUTO_DEPLOY)
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
endif()
