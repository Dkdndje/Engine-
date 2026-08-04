cmake_minimum_required(VERSION 3.10)
project(Torque2DApp C CXX)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Proje kaynak kodları ve dosyaları
file(GLOB_RECURSE ENGINE_SOURCES 
    "engine/src/*.cpp"
    "engine/src/*.c"
    "src/*.cpp"
    "src/*.c"
)

# Android derlemesi için kütüphane oluşturma
if(ANDROID)
    add_library(Torque2D SHARED ${ENGINE_SOURCES})
    
    # Android sistem kütüphanelerini bağlama (Log, OpenGL ES 2, Android Native Window)
    target_link_libraries(Torque2D
        android
        log
        GLESv2
        EGL
    )
else()
    # Masaüstü (Desktop) derlemesi
    add_executable(Torque2D ${ENGINE_SOURCES})
endif()

# Include dizinlerini ekleme
target_include_directories(Torque2D PUBLIC
    "engine/src"
    "src"
)
