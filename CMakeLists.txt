cmake_minimum_required(VERSION 3.11)

project(SFML-dependencies)

# We'll use cmake FetchContent to acquire the source
include(FetchContent)

FetchContent_Declare(
  freetype2
  GIT_REPOSITORY git://git.sv.nongnu.org/freetype/freetype2.git
  GIT_TAG        VER-2-8
)

FetchContent_Declare(
  ogg
  GIT_REPOSITORY https://github.com/xiph/ogg.git
  GIT_TAG        26213e6456f7bd3c12114911948b9140ad397419 # v1.3.3 plus a few commits for better cmake support
) 

FetchContent_Declare(
  opus
  GIT_REPOSITORY https://github.com/xiph/opus.git
  GIT_TAG        v1.2.1
)

FetchContent_Declare(
  opusfile
  GIT_REPOSITORY https://github.com/xiph/opusfile.git
  GIT_TAG        v0.8
)

FetchContent_Declare(
  flac
  GIT_REPOSITORY https://github.com/xiph/flac.git
  GIT_TAG        cc15b7427a9d4c94233dcad24cd8ed4c43f6c261 # 1.3.2 plus a few commits for better cmake support
)

FetchContent_Declare(
  openal-soft
  GIT_REPOSITORY https://github.com/kcat/openal-soft.git
  GIT_TAG        openal-soft-1.19.1
)

FetchContent_Declare(
  vorbis
  GIT_REPOSITORY https://github.com/xiph/vorbis.git
  GIT_TAG        v1.3.6
)

FetchContent_Declare(
  stb
  GIT_REPOSITORY https://github.com/nothings/stb.git
)

FetchContent_Populate(freetype2)
FetchContent_Populate(ogg)
FetchContent_Populate(opusfile)
FetchContent_Populate(opus)
FetchContent_Populate(flac)
FetchContent_Populate(openal-soft)
FetchContent_Populate(vorbis)

# Disable harfbuzz for freetype
set(WITH_HarfBuzz FALSE)
add_subdirectory(${freetype2_SOURCE_DIR} ${freetype2_BINARY_DIR})

# No extra configuration for ogg
add_subdirectory(${ogg_SOURCE_DIR} ${ogg_BINARY_DIR})

# openal-soft is provided by emscripten so we don't need to build
# It also builds the extras by default, so disable them
if (NOT EMSCRIPTEN AND NOT IOS)
    set(ALSOFT_UTILS OFF CACHE BOOL "")
    set(ALSOFT_EXAMPLES OFF CACHE BOOL "")
    set(ALSOFT_TESTS OFF CACHE BOOL "")
    add_subdirectory(${openal-soft_SOURCE_DIR} ${openal-soft_BINARY_DIR})
endif()

# vorbis requires these ogg variables to be set
set(OGG_LIBRARIES $<TARGET_FILE:ogg>) 
set(OGG_INCLUDE_DIRS $<TARGET_PROPERTY:ogg,INCLUDE_DIRECTORIES>)
add_subdirectory(${vorbis_SOURCE_DIR} ${vorbis_BINARY_DIR})

# flac is still waiting for cmake support
# so we'll roll our own for now
set(FLAC_SRC
    ${flac_SOURCE_DIR}/src/libFLAC/bitmath.c
    ${flac_SOURCE_DIR}/src/libFLAC/bitreader.c
    ${flac_SOURCE_DIR}/src/libFLAC/bitwriter.c
    ${flac_SOURCE_DIR}/src/libFLAC/cpu.c
    ${flac_SOURCE_DIR}/src/libFLAC/crc.c
    ${flac_SOURCE_DIR}/src/libFLAC/fixed.c
    ${flac_SOURCE_DIR}/src/libFLAC/float.c
    ${flac_SOURCE_DIR}/src/libFLAC/format.c
    ${flac_SOURCE_DIR}/src/libFLAC/lpc.c
    ${flac_SOURCE_DIR}/src/libFLAC/md5.c
    ${flac_SOURCE_DIR}/src/libFLAC/memory.c
    ${flac_SOURCE_DIR}/src/libFLAC/metadata_iterators.c
    ${flac_SOURCE_DIR}/src/libFLAC/metadata_object.c
    ${flac_SOURCE_DIR}/src/libFLAC/ogg_decoder_aspect.c
    ${flac_SOURCE_DIR}/src/libFLAC/ogg_encoder_aspect.c
    ${flac_SOURCE_DIR}/src/libFLAC/ogg_helper.c
    ${flac_SOURCE_DIR}/src/libFLAC/ogg_mapping.c
    ${flac_SOURCE_DIR}/src/libFLAC/stream_decoder.c
    ${flac_SOURCE_DIR}/src/libFLAC/stream_encoder.c
    ${flac_SOURCE_DIR}/src/libFLAC/stream_encoder_framing.c
    ${flac_SOURCE_DIR}/src/libFLAC/window.c
)

if (WIN32)
    list (APPEND FLAC_SRC ${flac_SOURCE_DIR}/src/share/win_utf8_io/win_utf8_io.c ${flac_SOURCE_DIR}/src/libFLAC/windows_unicode_filenames.c)
endif()

add_library(FLAC ${FLAC_SRC})
target_compile_definitions(FLAC PRIVATE "-DPACKAGE_VERSION=\"1.3.2\"" "-DHAVE_LROUND" "-DFLAC__HAS_OGG=0" "-DHAVE_STDINT_H" "-DHAVE_INTTYPES_H" "-DFLAC_API_EXPORTS" "-DFLAC__HAS_X86INTRIN" "-DFLAC__NO_ASM" "DLL_EXPORT")
target_include_directories(FLAC PUBLIC ${flac_SOURCE_DIR}/src/libFLAC/include ${flac_SOURCE_DIR}/include PRIVATE ${OGG_INCLUDE_DIRS})
target_link_libraries(FLAC ogg)

if (BUILD_SHARED_LIBS)
    target_compile_definitions(FLAC PUBLIC "-DFLAC_API_EXPORTS")
else()
    target_compile_definitions(FLAC PUBLIC "-DFLAC__NO_DLL")
endif()
