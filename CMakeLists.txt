cmake_minimum_required(VERSION 3.16)
project(FunctionalComponents CXX)

################################################################################
# Configure paths
################################################################################
set(GTA_SA_DIR F:/GTASanAndreas)
set(PLUGIN_SDK_DIR $ENV{PLUGIN_SDK_DIR})
################################################################################

################################################################################
# Source groups
################################################################################
set(src_files
    "src/Brakes.cpp"
    "src/Brakes.h"
    "src/Chain.cpp"
    "src/Chain.h"
    "src/Common.cpp"
    "src/Common.h"
    "src/Gear.cpp"
    "src/Gear.h"
    "src/Meter.cpp"
    "src/Meter.h"
    "src/pch.cpp"
    "src/pch.h"
    "src/FunctionalComponents.cpp"
)

################################################################################
# Target
################################################################################
add_library(${PROJECT_NAME} SHARED ${src_files})
target_precompile_headers(${PROJECT_NAME} PUBLIC "src/pch.h")

string(CONCAT "MSVC_RUNTIME_LIBRARY_STR"
    $<$<CONFIG:Release>:
        MultiThreaded
    >
    $<$<CONFIG:Debug>:
        MultiThreadedDebug
    >
)

set_target_properties(${PROJECT_NAME} PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY  "${GTA_SA_DIR}/$<0:>/"
    SUFFIX ".asi"
    MSVC_RUNTIME_LIBRARY ${MSVC_RUNTIME_LIBRARY_STR}
)

################################################################################
# Include directories
################################################################################
include_directories(
    "${PLUGIN_SDK_DIR}/plugin_sa"
    "${PLUGIN_SDK_DIR}/plugin_sa/game_sa"
    "${PLUGIN_SDK_DIR}/shared"
    "${PLUGIN_SDK_DIR}/shared/game"
)

################################################################################
# Compile definitions
################################################################################
target_compile_definitions(${PROJECT_NAME} PRIVATE
    "$<$<CONFIG:Release>:"
        "_NDEBUG"
    ">"
    "$<$<CONFIG:Debug>:"
        "_DEBUG"
    ">"
    "_CRT_SECURE_NO_WARNINGS;"
    "_CRT_NON_CONFORMING_SWPRINTFS"
    "GTASA"
    "PLUGIN_SGV_10US"
    "_MBCS"
)

################################################################################
# Compile and link options
################################################################################
if(MSVC)
    target_compile_options(${PROJECT_NAME} PRIVATE
        $<$<CONFIG:Release>:
            /O2;
            /Oi;
            /Gy
        >
        $<$<CONFIG:Debug>:
            /Od
        >
        /std:c++latest;
        /sdl-;
        /W3;
        ${DEFAULT_CXX_DEBUG_INFORMATION_FORMAT};
        ${DEFAULT_CXX_EXCEPTION_HANDLING}
        /w44005
    )
    target_link_options(${PROJECT_NAME} PRIVATE
        $<$<CONFIG:Release>:
            /SAFESEH:NO;
            /OPT:REF;
            /LTCG;
            /OPT:ICF
        >
        $<$<CONFIG:Debug>:
            /DEBUG:FULL;
        >
        /SUBSYSTEM:WINDOWS
    )
endif()

################################################################################
# Pre build events
################################################################################
add_custom_command(
    TARGET ${PROJECT_NAME} 
    PRE_BUILD
    COMMAND  taskkill /f /fi "imagename eq gta_sa.exe"
)

################################################################################
# Dependencies
################################################################################
target_link_libraries(${PROJECT_NAME} PUBLIC 
optimized plugin
debug plugin_d
)

target_link_directories(${PROJECT_NAME} PUBLIC
    "${PLUGIN_SDK_DIR}/output/lib/"
)



