cmake_minimum_required(VERSION 3.19)
project(ghostling C)

# Generate compile_commands.json for clangd/IDE support.
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

include(FetchContent)

# --- Raylib ------------------------------------------------------------------
# Use a system-installed raylib if available, otherwise fetch from GitHub.
set(RAYLIB_VERSION 5.5)
find_package(raylib ${RAYLIB_VERSION} QUIET)
if (NOT raylib_FOUND)
  FetchContent_Declare(
    raylib
    DOWNLOAD_EXTRACT_TIMESTAMP OFF
    URL https://github.com/raysan5/raylib/archive/refs/tags/${RAYLIB_VERSION}.tar.gz
  )
  FetchContent_GetProperties(raylib)
  if (NOT raylib_POPULATED)
    set(FETCHCONTENT_QUIET NO)
    FetchContent_MakeAvailable(raylib)
  endif()
endif()

# --- libghostty-vt -----------------------------------------------------------
# Virtual terminal emulator library from Ghostty. Requires `zig` on PATH.
# The Ghostty CMakeLists.txt delegates to `zig build lib-vt` to produce the
# shared library. See https://ghostty.org/docs/install/build for the
# required Zig version.
FetchContent_Declare(ghostty
  GIT_REPOSITORY https://github.com/ghostty-org/ghostty.git
  GIT_TAG main
)
FetchContent_MakeAvailable(ghostty)

# --- Our Project -------------------------------------------------------------
add_executable(${PROJECT_NAME} main.c)
target_compile_features(${PROJECT_NAME} PRIVATE c_std_23)
target_link_libraries(${PROJECT_NAME} raylib ghostty-vt)

# macOS requires these system frameworks for raylib's windowing/input/rendering.
if (APPLE)
  target_link_libraries(${PROJECT_NAME} "-framework IOKit")
  target_link_libraries(${PROJECT_NAME} "-framework Cocoa")
  target_link_libraries(${PROJECT_NAME} "-framework OpenGL")
endif()
