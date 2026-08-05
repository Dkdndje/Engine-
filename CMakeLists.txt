cmake_minimum_required(VERSION 3.10)
project(Torque2DApp C CXX)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Eski C kütüphanelerinin (OpenAL, ljpeg, freetype, zlib) Clang 18 ile derlenebilmesi için uyumluluk bayrakları
add_compile_options(
    -Wno-implicit-function-declaration
    -Wno-error=implicit-function-declaration
    -Wno-int-conversion
    -Wno-error=int-conversion
    -fcommon
)

# FreeType, Torque2D ve OpenAL için gerekli makrolar
add_definitions(
    -DFT2_BUILD_LIBRARY 
    -DFT_CONFIG_CONFIG_H=<ftconfig.h> 
    -DFT_CONFIG_MODULES_H=<ftmodule.h>
    -DAL_ALEXT_PROTOTYPES
    -D_GNU_SOURCE
)

# 1. Torque2D C/C++ Kaynak Kodlarını Tara
file(GLOB_RECURSE ENGINE_SOURCES 
    "engine/src/*.cpp"
    "engine/src/*.c"
    "engine/lib/*.cpp"
    "engine/lib/*.c"
)

# 2. Android Dışı Platform Kodlarını Süz
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/src/platformWin32/.*")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/src/platformOSX/.*")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/src/platformMac/.*")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/src/platformX86UNIX/.*")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/src/platformiOS/.*")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/src/platformEmscripten/.*")

# 3. FreeType ve Diğer Kütüphanelerin Sorunlu Araç/Gereçlerini Süz
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/autofit/.*")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/builds/.*")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/amiga.*")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/mac.*")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/win32.*")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX ".*/ftmac\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/gxvalid/.*")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/otvalid/.*")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/src/tools/.*")

# FreeType Gzip içindeki tekil derlenmeyen dosyaları süz
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/gzip/adler32\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/gzip/infblock\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/gzip/infcodes\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/gzip/inflate\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/gzip/inftrees\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/gzip/infutil\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/gzip/zutil\\.c$")

# FreeType çakışmaya yol açan birleştirilmiş (wrapper / amalgamation) ve iç içe dahil edilen dosyaları süz
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/cff/cff\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/cid/type1cid\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/cache/ftcache\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/raster/raster\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/smooth/smooth\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/pfr/pfr\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/pcf/pcf\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/truetype/truetype\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/type1/type1\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/type42/type42\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/psaux/psaux\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/pshinter/pshinter\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/sfnt/sfnt\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/base/ftbase\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/bdf/bdf\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/lzw/ftzopen\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/psnames/psnames\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/freetype/.*/sfnt/ttsbit0\\.c$")

# --- AUTOFIT MODÜLÜ BİRLEŞTİRİCİSİ (autofit_module_class ÇÖZÜMÜ) ---
# autofit altındaki tekil dosyaları filtrelemiştik; ana birleştirici dosyayı açıkça ekliyoruz:
file(GLOB AUTOFIT_MAIN "engine/lib/freetype/android/freetype-2.4.12/src/autofit/autofit.c")
if(AUTOFIT_MAIN)
    list(APPEND ENGINE_SOURCES ${AUTOFIT_MAIN})
endif()

# libvorbis içindeki standalone test programlarını süz
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/libvorbis/barkmel\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/libvorbis/tone\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/libvorbis/psytune\\.c$")

# ljpeg (JPEG) kütüphanesinin ek/araç programlarını ve çakışan bellek yöneticilerini süz
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/ljpeg/extras/.*")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/ljpeg/ansi2knr\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/ljpeg/cjpeg\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/ljpeg/djpeg\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/ljpeg/jpegtran\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/ljpeg/rdjpgcom\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/ljpeg/wrjpgcom\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/ljpeg/example\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/ljpeg/jmemdos\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/ljpeg/jmemmac\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/ljpeg/jmemname\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/ljpeg/jmemansi\\.c$")

# libpng test dosyasını süz
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/lpng/pngtest\\.c$")

# OpenAL masaüstü ve diğer platform sürücülerini süz (Android'de çalışmazlar)
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/openal/win32/.*")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/openal/mac/.*")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/openal/iOS/.*")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/openal/.*/examples/.*")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/openal/.*/tests/.*")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/openal/.*/Alc/backends/.*")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/openal/.*/alsa\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/openal/.*/oss\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/openal/.*/dsound\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/openal/.*/winmm\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/openal/.*/pulseaudio\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/openal/.*/portaudio\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/openal/.*/coreaudio\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/openal/.*/solaris\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/openal/.*/sndio\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/openal/.*/qsa\\.c$")
list(FILTER ENGINE_SOURCES EXCLUDE REGEX "engine/lib/openal/.*/jack\\.c$")

# 4. Android İçin Paylaşımlı Kütüphane Oluştur
add_library(Torque2D SHARED ${ENGINE_SOURCES})

# 5. Include Dizinleri
target_include_directories(Torque2D PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/src
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/freetype
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/freetype/include
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/freetype/android/freetype-2.4.12/include
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/freetype/android/freetype-2.4.12/include/freetype
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/freetype/android/freetype-2.4.12/include/freetype/config
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/freetype/android/freetype-2.4.12/include/freetype/internal
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/freetype/android/freetype-2.4.12/src/autofit
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/zlib
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/png
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/ljpeg
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/libogg
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/libogg/include
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/libvorbis
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/libvorbis/include
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/libvorbis/lib
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/openal
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/openal/include
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/openal/include/AL
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/openal/Android/openal-soft-master/jni/OpenAL
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/openal/Android/openal-soft-master/jni/OpenAL/include
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/openal/Android/openal-soft-master/jni/OpenAL/include/AL
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/openal/Android/openal-soft-master/jni/OpenAL/Alc
    ${CMAKE_CURRENT_SOURCE_DIR}/engine/lib/openal/Android/openal-soft-master/jni/OpenAL/OpenAL32/Include
)

# 6. Android Sistem Kütüphanelerini Bağla
target_link_libraries(Torque2D
    android
    log
    GLESv2
    EGL
    OpenSLES
    atomic
    m
)
