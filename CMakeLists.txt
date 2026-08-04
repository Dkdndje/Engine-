cmake_minimum_required(VERSION 3.10)
project(Torque2DApp C CXX)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Projedeki tüm C/C++ kaynak dosyalarını otomatik tara
file(GLOB_RECURSE ENGINE_SOURCES 
    "*.cpp"
    "*.c"
    "*.cc"
    "engine/*.cpp"
    "engine/*.c"
    "source/*.cpp"
    "source/*.c"
)

# Dosya bulunamazsa açık hata mesajı ver
list(LENGTH ENGINE_SOURCES SOURCE_COUNT)
if(SOURCE_COUNT EQUAL 0)
    message(FATAL_ERROR "Derlenecek C/C++ dosyasi bulunamadi! Lutfen klasor yapisini kontrol edin.")
endif()

# Android için paylaşımlı kütüphane oluştur
add_library(Torque2D SHARED ${ENGINE_SOURCES})

# Android sistem kütüphanelerini bağla
target_link_libraries(Torque2D
    android
    log
    GLESv2
    EGL
)

# Include dizinlerini ekle
target_include_directories(Torque2D PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}
    "engine"
    "source"
    "include"
)
