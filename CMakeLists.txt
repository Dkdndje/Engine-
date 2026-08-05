cmake_minimum_required(VERSION 3.10)
project(Torque2DApp C CXX)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# FreeType ve Torque2D için gerekli makrolar
add_definitions(-DFT2_BUILD_LIBRARY -DFT_CONFIG_CONFIG_H=<ftconfig.h> -DFT_CONFIG_MODULES_H=<ftmodule.h>)

# 1. Torque2D C/C++ Kaynak Kodlarını Tara
file(GLOB_RECURSE ENGINE_SOURCES 
    "engine/src/*.cpp"
    "engine/src/*.c"
    "engine/lib/*.cpp"
    "engine/lib/*.c"
)

# 2. FreeType'ın sorun çıkaran alt modül dosyalarını (autofit alt parçaları vb.) süz
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/autofit/.*")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/builds/.*")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/amiga.*")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/mac.*")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/win32.*")

# ftmac.c dosyasını özellikle de hariç tut (macOS-only, CoreServices gerektirir)
list(FILTER ENGINE_SOURCES EXCLUDE REGEX ".*/ftmac\\.c$")

# gxvalid ve otvalid modüllerini tamamen hariç tut
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/gxvalid/.*")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/otvalid/.*")

# src/tools klasörü: FreeType geliştirici araçları
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/src/tools/.*")

# gzip modülü: sadece ftgzip.c derlenmeli
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/gzip/adler32\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/gzip/infblock\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/gzip/infcodes\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/gzip/inflate\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/gzip/inftrees\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/gzip/infutil\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/gzip/zutil\\.c$")

# FreeType ana kaynak dosyasını ekle
file(GLOB FREETYPE_BASE_SOURCES "engine/lib/freetype/android/freetype-2.4.12/src/base/*.c")

# base dosyaları eklemeden önce ftmac.c'yi bu listeden de çıkar
list(FILTER FREETYPE_BASE_SOURCES EXCLUDE REGEX ".*/ftmac\\.c$")

list(APPEND ENGINE_SOURCES ${FREETYPE_BASE_SOURCES})

# 3. Android İçin Paylaşımlı Kütüphane Oluştur
add_library(Torque2D SHARED ${ENGINE_SOURCES})

# 4. Include Dizinleri
target_include_directories(Torque2D PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/src
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/freetype
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/freetype/include
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/freetype/android/freetype-2.4.12/include
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/freetype/android/freetype-2.4.12/include/freetype
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/freetype/android/freetype-2.4.12/include/freetype/config
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/zlib
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/png
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/libogg/include
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/libvorbis/include
)

# 5. Android Sistem Kütüphanelerini Bağla
target_link_libraries(Torque2D
    android
    log
    GLESv2
    EGL
)
