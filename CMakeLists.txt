cmake_minimum_required(VERSION 3.10)
project(Torque2DApp C CXX)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 1. Projedeki C/C++ dosyalarını tara
file(GLOB_RECURSE ENGINE_SOURCES 
    "engine/src/*.cpp"
    "engine/src/*.c"
    "engine/lib/freetype/src/*.c"
    "engine/lib/tamxml/*.cpp"
    "engine/lib/tinyxml/*.cpp"
    "engine/lib/zlib/*.c"
    "engine/lib/png/*.c"
)

# 2. Amiga, Mac, Symbian, Win32 gibi gereksiz/hatalı dosyaları listeden çıkar
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*builds/.*")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*amiga.*")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*mac.*")

# Android için özel FreeType sistem dosyasını ekle
list(APPEND ENGINE_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/freetype/src/base/ftsystem.c")
list(APPEND ENGINE_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/freetype/src/base/ftdebug.c")

# 3. Android için paylaşımlı kütüphane oluştur
add_library(Torque2D SHARED ${ENGINE_SOURCES})

# 4. Include Dizinleri (ft2build.h için gerekli tüm yollar)
target_include_directories(Torque2D PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/src
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/freetype/include
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/freetype/include/freetype
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
