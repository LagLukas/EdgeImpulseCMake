cmake_minimum_required(VERSION 3.10)
project(EDGE_IMPULSE_HELLO_WORLD)

SET(FULL_TFLITE 0)
# available targets are linux-86, linux-armv7, linux-aarch64
SET(TARGET linux-x86)

# set C/C++ flags
SET(CMAKE_C_FLAGS "-Wall -g -Wno-strict-aliasing -Os -DNDEEBUG -DEI_CLASSIFIER_ENABLE_DETECTION_POSTPROCESS_OP")
if (${FULL_TFLITE} EQUAL 1)
        SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DEI_CLASSIFIER_USE_FULL_TFLITE=1")
else()
        SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DTF_LITE_DISABLE_X86_NEON=1")
endif()
SET(CMAKE_CXX_FLAGS "${CMAKE_C_FLAGS} -std=c++14")

# search for the tflite libs
if (${FULL_TFLITE} EQUAL 1)
        find_library(CPUINFO cpuinfo tflite/${TARGET})
        find_library(FARMHASH farmhash tflite/${TARGET})
        find_library(libfft2d_fftsg2d fft2d_fftsg2d tflite/${TARGET})
        find_library(libfft2d_fftsg fft2d_fftsg tflite/${TARGET})
        find_library(libtensorflow-lite tensorflow-lite tflite/${TARGET})
        find_library(libruy ruy tflite/${TARGET})
        find_library(libXNNPACK XNNPACK tflite/${TARGET})
endif()

# grab C source files edge impulse sdk
AUX_SOURCE_DIRECTORY(edge-impulse-sdk/CMSIS/DSP/Source/TransformFunctions TransformFunctions)
AUX_SOURCE_DIRECTORY(edge-impulse-sdk/CMSIS/DSP/Source/CommonTables CommonTables)
AUX_SOURCE_DIRECTORY(edge-impulse-sdk/CMSIS/DSP/Source/BasicMathFunctions BasicMathFunctions)
AUX_SOURCE_DIRECTORY(edge-impulse-sdk/CMSIS/DSP/Source/ComplexMathFunctions ComplexMathFunctions)
AUX_SOURCE_DIRECTORY(edge-impulse-sdk/CMSIS/DSP/Source/FastMathFunctions FastMathFunctions)
AUX_SOURCE_DIRECTORY(edge-impulse-sdk/CMSIS/DSP/Source/SupportFunctions SupportFunctions)
AUX_SOURCE_DIRECTORY(edge-impulse-sdk/CMSIS/DSP/Source/MatrixFunctions MatrixFunctions)
AUX_SOURCE_DIRECTORY(edge-impulse-sdk/CMSIS/DSP/Source/StatisticsFunctions StatisticsFunctions)
# grab C++ source files edge impulse sdk
AUX_SOURCE_DIRECTORY(edge-impulse-sdk/dsp/kissfft kissfft)
AUX_SOURCE_DIRECTORY(edge-impulse-sdk/dsp/dct dct)
SET(MEMORY_CPP edge-impulse-sdk/dsp/memory.cpp)
AUX_SOURCE_DIRECTORY(edge-impulse-sdk/porting/posix posix)
AUX_SOURCE_DIRECTORY(edge-impulse-sdk/porting/mingw32 mingw32)
# grab C++ model file
AUX_SOURCE_DIRECTORY(tflite-model model)

if (${FULL_TFLITE} EQUAL 1)
        add_executable(app source/camera.cpp ${TransformFunctions} ${CommonTables} ${BasicMathFunctions} ${ComplexMathFunctions} ${FastMathFunctions} ${SupportFunctions} ${MatrixFunctions} ${StatisticsFunctions} ${kissfft} ${dct} ${MEMORY_CPP} ${posix} ${mingw32} ${model})
        target_include_directories(app PUBLIC . model-parameters tflite-model third_party/ tensorflow-lite/ source)
        target_link_libraries(app m stdc++)
        if ((${TARGET} STREQUAL "linux-x86") OR (${TARGET} STREQUAL "linux-armv7"))
                target_link_options(app PUBLIC "-Wl,--no-as-needed")
        endif()
        target_link_libraries(app dl ${libtensorflow-lite} ${CPUINFO} ${FARMHASH} ${libfft2d_fftsg} ${libfft2d_fftsg2d} ${libruy} ${libXNNPACK} pthread)
else()
        # grab additional C/C++ source files
        SET(tf_common edge-impulse-sdk/tensorflow/lite/c/common.c)
        AUX_SOURCE_DIRECTORY(edge-impulse-sdk/tensorflow/lite/kernels/ tflite_kernels)
        AUX_SOURCE_DIRECTORY(edge-impulse-sdk/tensorflow/lite/kernels/internal tflite_internal)
        AUX_SOURCE_DIRECTORY(edge-impulse-sdk/tensorflow/lite/micro/kernels tfmicro_kernels)
        AUX_SOURCE_DIRECTORY(edge-impulse-sdk/tensorflow/lite/micro tfmicro)
        AUX_SOURCE_DIRECTORY(edge-impulse-sdk/tensorflow/lite/micro/memory_planner tfmemory_planner)
        AUX_SOURCE_DIRECTORY(edge-impulse-sdk/tensorflow/lite/core/api tf_api)
        add_executable(app source/camera.cpp ${TransformFunctions} ${CommonTables} ${BasicMathFunctions} ${ComplexMathFunctions} ${FastMathFunctions} ${SupportFunctions} ${MatrixFunctions} ${StatisticsFunctions} ${model} ${kissfft} ${dct} ${MEMORY_CPP} ${posix} ${mingw32} ${tf_common} ${tflite_kernels} ${tflite_internal} ${tfmicro_kernels} ${tfmicro} ${tfmemory_planner} ${tf_api})
        target_include_directories(app PUBLIC . source model-parameters tflite-model third_party/)
        target_link_libraries(app m stdc++)
endif()
