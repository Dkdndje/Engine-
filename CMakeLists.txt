cmake_minimum_required(VERSION 3.10)
project(Torque2DApp C CXX)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# FreeType ve Torque2D için derleme makroları
add_definitions(-DFT2_BUILD_LIBRARY)

# 1. Torque2D C/C++ Kaynak Kodlarını Tara
file(GLOB_RECURSE ENGINE_SOURCES 
    "engine/src/*.cpp"
    "engine/src/*.c"
    "engine/lib/*.cpp"
    "engine/lib/*.c"
)

# 2. Amiga, Mac, Win32 gibi uyumsuz platform dosyalarını süz
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/builds/.*")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/amiga.*")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/mac.*")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/win32.*")

# 3. Android İçin Paylaşımlı Kütüphane Oluştur
add_library(Torque2D SHARED ${ENGINE_SOURCES})

# 4. Include Dizinleri (FreeType internal & config yolları dahil)
target_include_directories(Torque2D PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/src
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/freetype
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/freetype/include
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/freetype/android/freetype-2.4.12/include
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/freetype/android/freetype-2.4.12/include/freetype
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/freetype/android/freetype-2.4.12/include/freetype/config
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/freetype/android/freetype-2.4.12/include/freetype/internal
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/zlib
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/png
)

# 5. Android Sistem Kütüphanelerini Bağla
target_link_libraries(Torque2D
    android
    log
    GLESv2
    EGL
)
