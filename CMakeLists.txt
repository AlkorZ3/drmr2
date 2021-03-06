#CMake file for DrMr

cmake_minimum_required(VERSION 2.6)
project (DrMr2)

set(LV2_INSTALL_DIR lib/lv2 CACHE PATH "Specifies where the LV2 libraries should be installed")
set(CMAKE_C_FLAGS "-Wall")

# Availble options
option(USE_NKNOB  "Use custom NKnob widgets for gain/pan instead of Gtk sliders" ON)
SET(SAMP_ZERO_POS "0" CACHE STRING "Default sample zero position 0=top left (tl) 1=bl 2=tr 3=br")

# check for our various libraries
find_package(PkgConfig)
pkg_check_modules(LV2 REQUIRED lv2>=0.1.0)
pkg_check_modules(SNDFILE REQUIRED sndfile>=1.0.20)
pkg_check_modules(SAMPLERATE REQUIRED samplerate>=0.1.5)
pkg_check_modules (GTK2 REQUIRED gtk+-2.0>=2.22.0)
find_package (EXPAT REQUIRED)

include_directories (${LV2_INCLUDE_DIRS} ${EXPAT_INCLUDE_DIRS} ${SNDFILE_INCLUDE_DIRS} ${SAMPLERATE_INCLUDE_DIRS} ${GTK2_INCLUDE_DIRS})
link_directories (${LV2_LIBRARY_DIRS} ${EXPAT_LIBRARY_DIRS} ${SNDFILE_LIBRARY_DIRS} ${SAMPLERATE_LIBRARY_DIRS} ${GTK2_LIBRARY_DIRS})

# config libraries

add_library(drmr2 SHARED
  drmr2.c
  drmr2_hydrogen.c
  drmr2.h
  drmr2_hydrogen.h
)

add_library(drmr2_ui SHARED
  drmr2_ui.c
  drmr2_hydrogen.c
  nknob.c
  drmr2_hydrogen.h
  nknob.h
)

set_target_properties(drmr2 PROPERTIES PREFIX "")
set_target_properties(drmr2_ui PROPERTIES PREFIX "")
set_target_properties(drmr2_ui 
  PROPERTIES
  LINK_FLAGS "-Wl,-znodelete"
)

target_link_libraries(drmr2 ${LV2_LIBRARIES} ${SNDFILE_LIBRARIES} ${SAMPLERATE_LIBRARIES} ${EXPAT_LIBRARIES} m)
target_link_libraries(drmr2_ui ${LV2_LIBRARIES} ${GTK2_LIBRARIES} ${SNDFILE_LIBRARIES} ${SAMPLERATE_LIBRARIES} ${EXPAT_LIBRARIES} m)


add_definitions ( -DPIC )

set_target_properties (drmr2
  PROPERTIES
  COMPILE_FLAGS "-std=gnu99"
)

set (ui_compile_flags "-std=gnu99")

if (NOT USE_NKNOB)
  set (ui_compile_flags "${ui_compile_flags} -DNO_NKNOB")
endif (NOT USE_NKNOB)

if (SAMP_ZERO_POS GREATER "-1" AND SAMP_ZERO_POS LESS "4")
   set (ui_compile_flags "${ui_compile_flags} -DDRMR2_UI_ZERO_SAMP=${SAMP_ZERO_POS}")
else (SAMP_ZERO_POS GREATER "-1" AND SAMP_ZERO_POS LESS "4")
   message(WARNING "Invalid SAMP_ZERO_POS, defaulting to 0 \(top left\).  Valid values are 0,1,2,3")
   set (ui_compile_flags "${ui_compile_flags} -DDRMR2_UI_ZERO_SAMP=0")
endif (SAMP_ZERO_POS GREATER "-1" AND SAMP_ZERO_POS LESS "4")

set_target_properties (drmr2_ui
  PROPERTIES
  COMPILE_FLAGS ${ui_compile_flags}
)

# drmr2.ttl generation
add_custom_command(TARGET drmr2 COMMAND ./drmr2-mkttl WORKING_DIRECTORY ..)

# Custom test commands
add_executable ( htest
  EXCLUDE_FROM_ALL
  drmr2_hydrogen.c
)

add_executable ( knobt
  EXCLUDE_FROM_ALL
  nknob.c
)

# config test executables
target_link_libraries(htest ${LV2_LIBRARIES} ${GTK2_LIBRARIES} ${SNDFILE_LIBRARIES} ${SAMPLERATE_LIBRARIES} ${EXPAT_LIBRARIES} m)
set_target_properties ( htest
  PROPERTIES
  COMPILE_FLAGS "-D_TEST_HYDROGEN_PARSER"
)
target_link_libraries(knobt ${LV2_LIBRARIES} ${GTK2_LIBRARIES} ${SNDFILE_LIBRARIES} ${SAMPLERATE_LIBRARIES} ${EXPAT_LIBRARIES} m)
set_target_properties ( knobt
  PROPERTIES
  COMPILE_FLAGS "-D_TEST_N_KNOB"
)

# config install
install(TARGETS drmr2 drmr2_ui
  LIBRARY
  DESTINATION ${CMAKE_INSTALL_PREFIX}/${LV2_INSTALL_DIR}/drmr2.lv2
)

install (FILES manifest.ttl drmr2.ttl knob.png led_on.png led_off.png
  DESTINATION ${CMAKE_INSTALL_PREFIX}/${LV2_INSTALL_DIR}/drmr2.lv2
)
