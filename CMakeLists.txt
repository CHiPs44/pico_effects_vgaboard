cmake_minimum_required(VERSION 3.13)

set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 17)

set(PICO_BOARD vgaboard)

# # Default path to the SDK
# if (NOT PICO_SDK_PATH)
#   set(PICO_SDK_PATH "../pico-sdk")
# endif ()

# # Default path to the SDK extras
# if (NOT PICO_EXTRAS_PATH)
#   set(PICO_EXTRAS_PATH "../pico-extras")
# endif ()

# Pull in Pico SDK (must be before project) and extras
# include(${PICO_SDK_PATH}/pico_sdk_init.cmake)
# include(${PICO_EXTRAS_PATH}/external/pico_extras_import.cmake)
# include(pico_extras_import.cmake)

# hagl/CMakeLists.txt wants this
set(PICO_SDK 1)

# Pull in Pico SDK (must be before project)
include(pico_sdk_import.cmake)
include(pico_extras_import.cmake)

project(pico_effects_vgaboard C CXX ASM)

pico_sdk_init()

add_executable(pico_effects_vgaboard main.c metaballs.c plasma.c rotozoom.c deform.c)

pico_set_program_name(pico_effects_vgaboard "Pico Effects for VGA board")
pico_set_program_version(pico_effects_vgaboard "0.1")

# Enable usb output, disable uart output
pico_enable_stdio_usb(pico_effects_vgaboard 1)
pico_enable_stdio_uart(pico_effects_vgaboard 0)

# Add the standard library to the build
target_link_libraries(pico_effects_vgaboard pico_stdlib)

# Add any user requested libraries
target_link_libraries(pico_effects_vgaboard
        pico_scanvideo_dpi
        pico_multicore
        hardware_interp
        hagl
        hagl_hal
        )

pico_add_extra_outputs(pico_effects_vgaboard)

add_subdirectory(external/hagl)
add_subdirectory(external/hagl_hal)

#
# If you have a large screen and run out of memory use
# the HAGL_HAL_PIXEL_SIZE=2 setting.
#
target_compile_definitions(pico_effects_vgaboard PRIVATE
  HAGL_HAL_USE_SINGLE_BUFFER
#  HAGL_HAL_USE_DOUBLE_BUFFER
#  HAGL_HAL_USE_TRIPLE_BUFFER
#  HAGL_HAL_USE_DMA
  HAGL_HAL_DEBUG
#  HAGL_HAL_PIXEL_SIZE=2
  # needed for 640x480 modes
  # -DPICO_SCANVIDEO_MAX_SCANLINE_BUFFER_WORDS=322
  # needed for 800x600 modes
  # -DPICO_SCANVIDEO_MAX_SCANLINE_BUFFER_WORDS=402
  # needed for 1024x768 modes?
  -DPICO_SCANVIDEO_MAX_SCANLINE_BUFFER_WORDS=514
  -DPICO_SCANVIDEO_SCANLINE_BUFFER_COUNT=8
  # -DUSE_INTERP=1
  -DUSE_INTERP=0
  # TINY: OK for 640x400 based modes and below
  # -DPICO_VGABOARD_FRAMEBUFFER_SIZE=32768
  # NORMAL: OK for 640x480 based modes and below
  -DPICO_VGABOARD_FRAMEBUFFER_SIZE=65536
  # BIG: OK for 256 colors based modes in "hi-res"
  # -DPICO_VGABOARD_FRAMEBUFFER_SIZE=98304
  # TOO MUCH?
  # -DPICO_VGABOARD_FRAMEBUFFER_SIZE=131072
  -DHAGL_HAL_DEBUG=1
  -DPICO_VGABOARD_DEBUG=1
  -DUSE_LED=1
)

# EOF
