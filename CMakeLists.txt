cmake_minimum_required(VERSION 3.10)

project(xrick VERSION 1.0 LANGUAGES C)

option(XRICK_USE_BUNDLED_SDLMAIN "Build xcode/xrick/SDLMain.m instead of relying on libSDLmain (macOS only)" OFF)

set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS ON)

if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type" FORCE)
endif()

# SDL 1.2 (or sdl12-compat); the code uses the SDL 1.x API.
find_package(SDL QUIET)
if(SDL_FOUND)
  set(XRICK_SDL_INCLUDE_DIRS ${SDL_INCLUDE_DIR})
  set(XRICK_SDL_LIBRARIES ${SDL_LIBRARY})
else()
  find_package(PkgConfig QUIET)
  if(PKG_CONFIG_FOUND)
    pkg_check_modules(SDL12 QUIET IMPORTED_TARGET sdl12_compat)
    if(NOT SDL12_FOUND)
      pkg_check_modules(SDL12 QUIET IMPORTED_TARGET sdl)
    endif()
  endif()
  if(SDL12_FOUND)
    set(XRICK_SDL_LIBRARIES PkgConfig::SDL12)
  else()
    message(FATAL_ERROR "SDL 1.2 (or sdl12-compat) not found")
  endif()
endif()

find_package(ZLIB REQUIRED)

set(XRICK_SOURCES
  src/control.c
  src/dat_ents.c
  src/dat_maps.c
  src/dat_picsPC.c
  src/dat_picsST.c
  src/dat_screens.c
  src/dat_snd.c
  src/dat_spritesPC.c
  src/dat_spritesST.c
  src/dat_tilesPC.c
  src/dat_tilesST.c
  src/data.c
  src/devtools.c
  src/draw.c
  src/e_bomb.c
  src/e_bonus.c
  src/e_box.c
  src/e_bullet.c
  src/e_rick.c
  src/e_sbonus.c
  src/e_them.c
  src/ents.c
  src/game.c
  src/maps.c
  src/rects.c
  src/scr_gameover.c
  src/scr_getname.c
  src/scr_imain.c
  src/scr_imap.c
  src/scr_pause.c
  src/scr_xrick.c
  src/scroller.c
  src/sysarg.c
  src/sysevt.c
  src/sysjoy.c
  src/syskbd.c
  src/syssnd.c
  src/system.c
  src/sysvid.c
  src/unzip.c
  src/util.c
  src/xrick.c
)

if(WIN32)
  enable_language(RC)
  list(APPEND XRICK_SOURCES src/xrick.rc)
endif()

if(APPLE AND XRICK_USE_BUNDLED_SDLMAIN)
  enable_language(OBJC)
  list(APPEND XRICK_SOURCES
    xcode/xrick/SDLMain.m
    xcode/xrick/SDLAppDelegate.m
  )
endif()

add_executable(xrick ${XRICK_SOURCES})

target_include_directories(xrick PRIVATE
  ${CMAKE_CURRENT_SOURCE_DIR}/include
  ${XRICK_SDL_INCLUDE_DIRS}
)

target_link_libraries(xrick PRIVATE ${XRICK_SDL_LIBRARIES} ZLIB::ZLIB)

if(APPLE)
  target_link_libraries(xrick PRIVATE "-framework Cocoa")
  if(XRICK_USE_BUNDLED_SDLMAIN)
    target_include_directories(xrick PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/xcode/xrick)
  endif()
endif()

if(WIN32 AND MSVC)
  target_compile_definitions(xrick PRIVATE _CRT_SECURE_NO_WARNINGS)
endif()

if(NOT MSVC)
  target_compile_options(xrick PRIVATE -Wall)
endif()

# Run from the build tree with the game data next to the executable.
if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/data.zip")
  add_custom_command(TARGET xrick POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
            "${CMAKE_CURRENT_SOURCE_DIR}/data.zip" "$<TARGET_FILE_DIR:xrick>/data.zip")
endif()

include(GNUInstallDirs)
install(TARGETS xrick RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/data.zip")
  install(FILES data.zip DESTINATION ${CMAKE_INSTALL_DATADIR}/xrick)
endif()
