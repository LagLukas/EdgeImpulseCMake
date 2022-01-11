# EdgeImpulseCMake

CMake file for the Edge Impulse Standalone Linux Examples:

https://github.com/edgeimpulse/example-standalone-inferencing-linux

Note I did not include Mac and Jetson Nano as I just do not use them.

Tested the stuff with my local Edge Impulse projects (no bigger scope, data 01/11/2022).

It aims at building a small application called myapp.cpp but this can be used as a starting point.

Important Variables

FULL_TFLITE: enables full tflite support
TARGET: target architecture (not this does not choose some compiler, but only the correct static libraries).
available targets are linux-86, linux-armv7, linux-aarch64

The CMake file is to be used instead of the Makefile described in the repo example-standalone-inferencing-linux
