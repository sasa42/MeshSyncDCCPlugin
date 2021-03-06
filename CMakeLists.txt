CMAKE_MINIMUM_REQUIRED(VERSION 3.5)
PROJECT(MeshSyncDCCPlugin)

set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/Build/cmake_modules)
set(SRC_ROOT ${CMAKE_CURRENT_SOURCE_DIR}/Src)
set(DIST_ROOT ${CMAKE_CURRENT_SOURCE_DIR}/Dist)
set(EXTERNAL_ROOT ${CMAKE_CURRENT_SOURCE_DIR}/External)

# prerequisites. 
find_package(Poco COMPONENTS Net)
include(ISPC)
set(ENABLE_ZSTD ON CACHE BOOL "Use ZSTD for scene cache compression")
find_package(ZSTD)
add_definitions(-DmsEnableZSTD)

#CCache variable to hold MeshSync version. This is overwritten by the passed argument
SET(MESHSYNC_VER "0.0.1-preview" CACHE STRING "MeshSync package version")

# MeshSync
include(MeshSync)
get_meshsync(${MESHSYNC_VER})
setup_meshsync(${MESHSYNC_VER})

# Force Unicode character set
add_definitions(-DUNICODE -D_UNICODE)

# ----------------------------------------------------------------------------------------------------------------------
# Call add_subdirectory using symbolic link
function(add_subdirectory_link to from)
    if(NOT EXISTS "${SRC_ROOT}/${to}")
        if(WIN32) 
            file(TO_NATIVE_PATH "${SRC_ROOT}/${from}" native_src)
            file(TO_NATIVE_PATH "${SRC_ROOT}/${to}"   native_dest)
            execute_process(WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}" COMMAND cmd.exe /c mklink /D ${native_dest} ${native_src})
        else()   
            execute_process(WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}" COMMAND ln -s "${SRC_ROOT}/${from}" "${SRC_ROOT}/${to}" )
        endif()
        
    endif()
    add_subdirectory("${SRC_ROOT}/${to}")
endfunction()

# ----------------------------------------------------------------------------------------------------------------------

# CXX Setup
if(APPLE)
    set(PLATFORM "Mac")
    set(MAC ON)
elseif(ANDROID)
    set(PLATFORM "Android")
elseif(${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
    set(PLATFORM "Linux")
    set(LINUX ON)

elseif(WIN32) 
    set(PLATFORM "Windows")
    
    # Additional Poco Net dependencies on Windows
    list(APPEND Poco_LIBRARIES "ws2_32.lib;Iphlpapi.lib" )       

    #Compile with MultiThreaded DLL flag to use dynamically-linked runtime library (Poco)
    set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /MD")
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MDd")        
   
    #Set /fp. We need so that -infinity == infinity == NaN. See Transform::setupDataFlags()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /fp:fast")
endif()


set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14")

# ----------------------------------------------------------------------------------------------------------------------

# Maya
list(APPEND MAYA_VERSIONS
    MAYA_2017
    MAYA_2018
    MAYA_2019
    MAYA_2020
) 

if(BUILD_MAYA_ALL)
    foreach(CUR_MAYA_VER IN LISTS MAYA_VERSIONS)
        option(BUILD_${CUR_MAYA_VER}_PLUGIN "Build MeshSync Plugin for ${CUR_MAYA_VER}" ON)
    endforeach(CUR_MAYA_VER)   
endif()

foreach(CUR_MAYA_VER IN LISTS MAYA_VERSIONS)
    option(BUILD_${CUR_MAYA_VER}_PLUGIN "Build MeshSync Plugin for ${CUR_MAYA_VER}" OFF)
    if (BUILD_${CUR_MAYA_VER}_PLUGIN)
        add_subdirectory_link(MeshSyncClient${CUR_MAYA_VER} MeshSyncClientMaya)
    endif()
endforeach(CUR_MAYA_VER)   


# ----------------------------------------------------------------------------------------------------------------------


# Blender
list(APPEND BLENDER_VERSIONS
    BLENDER_2.79
    BLENDER_2.80
) 
if(BUILD_BLENDER_ALL)
    foreach(CUR_BLENDER_VER IN LISTS BLENDER_VERSIONS)
        option(BUILD_${CUR_BLENDER_VER}_PLUGIN "Build MeshSync Plugin for ${CUR_BLENDER_VER}" ON)
    endforeach(CUR_BLENDER_VER)   
endif()

foreach(CUR_BLENDER_VER IN LISTS BLENDER_VERSIONS)
    option(BUILD_${CUR_BLENDER_VER}_PLUGIN "Build MeshSync Plugin for ${CUR_BLENDER_VER}" OFF)
    if (BUILD_${CUR_BLENDER_VER}_PLUGIN)
        add_subdirectory_link(MeshSyncClient${CUR_BLENDER_VER} MeshSyncClientBlender)
    endif()
endforeach(CUR_BLENDER_VER)   


# ----------------------------------------------------------------------------------------------------------------------

# 3DSMAX (Windows only)
list(APPEND 3DSMAX_VERSIONS
    3DSMAX_2017
    3DSMAX_2018
    3DSMAX_2019
    3DSMAX_2020
) 
if(BUILD_3DSMAX_ALL)
    foreach(CUR_3DSMAX_VER IN LISTS 3DSMAX_VERSIONS)
        option(BUILD_${CUR_3DSMAX_VER}_PLUGIN "Build MeshSync Plugin for ${CUR_3DSMAX_VER}" ON)
    endforeach(CUR_3DSMAX_VER)   
endif()

if(WIN32)
	foreach(CUR_3DSMAX_VER IN LISTS 3DSMAX_VERSIONS)
		option(BUILD_${CUR_3DSMAX_VER}_PLUGIN "Build MeshSync Plugin for ${CUR_3DSMAX_VER}" OFF)
		if (BUILD_${CUR_3DSMAX_VER}_PLUGIN)
			add_subdirectory_link(MeshSyncClient${CUR_3DSMAX_VER} MeshSyncClient3dsMax)
		endif()
	endforeach(CUR_3DSMAX_VER)   
endif()

# ----------------------------------------------------------------------------------------------------------------------
# MOTIONBUILDER (Windows only)
list(APPEND MOTIONBUILDER_VERSIONS
    MOTIONBUILDER_2017
    MOTIONBUILDER_2018
    MOTIONBUILDER_2019
    MOTIONBUILDER_2020
) 
if(BUILD_MOTIONBUILDER_ALL)
    foreach(CUR_MOTIONBUILDER_VER IN LISTS MOTIONBUILDER_VERSIONS)
        option(BUILD_${CUR_MOTIONBUILDER_VER}_PLUGIN "Build MeshSync Plugin for ${CUR_MOTIONBUILDER_VER}" ON)
    endforeach(CUR_MOTIONBUILDER_VER)   
endif()

if(WIN32 OR LINUX)
	foreach(CUR_MOTIONBUILDER_VER IN LISTS MOTIONBUILDER_VERSIONS)
		option(BUILD_${CUR_MOTIONBUILDER_VER}_PLUGIN "Build MeshSync Plugin for ${CUR_MOTIONBUILDER_VER}" OFF)
		if (BUILD_${CUR_MOTIONBUILDER_VER}_PLUGIN)
			add_subdirectory_link(MeshSyncClient${CUR_MOTIONBUILDER_VER} MeshSyncClientMotionBuilder)
		endif()
	endforeach(CUR_MOTIONBUILDER_VER)   
endif()



