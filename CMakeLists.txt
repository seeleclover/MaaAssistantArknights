cmake_minimum_required(VERSION 2.8.12)
project(MaaAssistantArknights)

option(BUILD_JNI "build jni" OFF)
option(BUILD_TEST "build a demo" OFF)

set(CMAKE_CXX_STANDARD 20)

if (WIN32)
    add_compile_options("$<$<C_COMPILER_ID:MSVC>:/utf-8>")
    add_compile_options("$<$<CXX_COMPILER_ID:MSVC>:/utf-8>")
    add_compile_options("$<$<C_COMPILER_ID:MSVC>:/MP>")
    add_compile_options("$<$<CXX_COMPILER_ID:MSVC>:/MP>")

    set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /MT")
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MTd")
endif ()

add_compile_options("$<$<CXX_COMPILER_ID:MSVC>:/W4;/WX>")
add_compile_options("$<$<NOT:$<CXX_COMPILER_ID:MSVC>>:-Wall;-Wextra;-Wpedantic>")

add_definitions(-DASST_DLL_EXPORTS)

include_directories(include 3rdparty/include)
aux_source_directory(src/MeoAssistant SRC)

add_library(MeoAssistant SHARED ${SRC})

find_library(PaddleOCR_LIB NAMES ppocr PATHS 3rdparty/lib)

if (WIN32)
    find_library(OpenCV NAMES opencv_world453 PATHS 3rdparty/lib)
    find_library(ZLIB NAMES zlibstatic PATHS 3rdparty/lib)
    target_link_libraries(MeoAssistant ws2_32)
else ()
    find_package(OpenCV REQUIRED)
    find_package(ZLIB REQUIRED)
endif ()

target_link_libraries(MeoAssistant ${OpenCV} ${PaddleOCR_LIB} ${ZLIB})

if (BUILD_JNI)
    find_package(JNI REQUIRED)
    include_directories(${JNI_INCLUDE_DIRS})

    set(JNI_SRC src/Java/jni/meoasst_jni.cpp)
    add_library(MeoAssistantJni SHARED ${JNI_SRC})
    target_link_libraries(MeoAssistantJni MeoAssistant)
endif(BUILD_JNI)

if (BUILD_TEST)
    add_executable(test tools/TestCaller/main.cpp)
    target_link_libraries(test MeoAssistant)
endif(BUILD_TEST)
