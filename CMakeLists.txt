#CMake file for DrMr

cmake_minimum_required(VERSION 2.6)
project (DrMr)

set(LV2_INSTALL_DIR lib/lv2 CACHE PATH "Specifies where the LV2 libraries should be installed")
set(CMAKE_C_FLAGS "-Wall")

# check for our various libraries
find_package(PkgConfig)
pkg_check_modules(LV2 REQUIRED lv2-plugin>=1.0.4)
pkg_check_modules(SNDFILE REQUIRED sndfile>=1.0.20)
pkg_check_modules(SAMPLERATE REQUIRED samplerate>=0.1.8)
pkg_check_modules (GTK2 REQUIRED gtk+-2.0>=2.22.0)
find_package (EXPAT REQUIRED)

include_directories (${LV2_INCLUDE_DIRS} ${EXPAT_INCLUDE_DIRS} ${SNDFILE_INCLUDE_DIRS} ${SAMPLERATE_INCLUDE_DIRS} ${GTK2_INCLUDE_DIRS})
link_directories (${LV2_LIBRARY_DIRS} ${EXPAT_LIBRARY_DIRS} ${SNDFILE_LIBRARY_DIRS} ${SAMPLERATE_LIBRARY_DIRS} ${GTK2_LIBRARY_DIRS})

# config libraries

add_library(drmr SHARED
  drmr.c
  drmr_hydrogen.c
  drmr.h
  drmr_hydrogen.h
)

add_library(drmr_ui SHARED
  drmr_ui.c
  drmr_hydrogen.c
  nknob.c
  drmr_hydrogen.h
  nknob.h
)
set_target_properties(drmr PROPERTIES PREFIX "")
set_target_properties(drmr_ui PROPERTIES PREFIX "")

target_link_libraries(drmr ${LV2_LIBRARIES} ${SNDFILE_LIBRARIES} ${SAMPLERATE_LIBRARIES} ${EXPAT_LIBRARIES} m)
target_link_libraries(drmr_ui ${LV2_LIBRARIES} ${GTK2_LIBRARIES} ${SNDFILE_LIBRARIES} ${SAMPLERATE_LIBRARIES} ${EXPAT_LIBRARIES} m)

add_definitions ( -DPIC )

set_target_properties (drmr_ui 
  PROPERTIES 
  DEFINE_SYMBOL "INSTALL_DIR=\"${CMAKE_INSTALL_PREFIX}/${LV2_INSTALL_DIR}\""
)


# config install
install(TARGETS drmr drmr_ui
  LIBRARY
  DESTINATION ${CMAKE_INSTALL_PREFIX}/${LV2_INSTALL_DIR}/drmr.lv2
)

install (FILES manifest.ttl drmr.ttl knob.png
  DESTINATION ${CMAKE_INSTALL_PREFIX}/${LV2_INSTALL_DIR}/drmr.lv2
)