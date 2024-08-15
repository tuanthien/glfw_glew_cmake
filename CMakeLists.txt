cmake_minimum_required(VERSION 3.27)

project("Demo" VERSION 0.0.1 DESCRIPTION "Demo")


include(PreventInSourceBuilds.cmake)
assure_out_of_source_builds()

add_executable(DemoTarget Demo.cpp)


set(CPM_DOWNLOAD_VERSION 0.38.7)
set(CPM_DOWNLOAD_LOCATION "${CMAKE_CURRENT_SOURCE_DIR}/CPM_${CPM_DOWNLOAD_VERSION}.cmake")

include(CPMPackageManager.cmake)
CPMAddPackage(
  GITHUB_REPOSITORY glfw/glfw
  GIT_SHALLOW    ON
  GIT_TAG 3.4
  OPTIONS
    "BUILD_SHARED_LIBS OFF"
  SYSTEM YES
  EXCLUDE_FROM_ALL YES
)

# since GLEW cmake script doesn't work because the git repo need to generate headers with perl
# so we download the release instead

if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
  set(GLEW_URL https://github.com/nigels-com/glew/releases/download/glew-2.2.0/glew-2.2.0-win32.zip)
  set(GLEW_SHA_256 ea6b14a1c6c968d0034e61ff6cb242cff2ce0ede79267a0f2b47b1b0b652c164)
  CPMAddPackage(
    NAME glew_download
    VERSION 2.2.0
    URL ${GLEW_URL}
    URL_HASH SHA256=${GLEW_SHA_256}
    DOWNLOAD_ONLY
  )

  if (glew_download_ADDED)
    list(APPEND CMAKE_PREFIX_PATH "${glew_download_SOURCE_DIR}")
    set(GLEW_USE_STATIC_LIBS ON)
    find_package(GLEW REQUIRED)
  else()
    message(FATAL_ERROR "Unable to add glew from \"${GLEW_URL}\"")
  endif()
elseif(CMAKE_SYSTEM_NAME STREQUAL "Linux")
  find_package(GLEW REQUIRED) # NOT TESTED try to find package from Linux distribution package manager
endif()

if(NOT TARGET GLEW::glew_s)
  message(FATAL_ERROR "Unable to add glew from \"${GLEW_URL}\"")
endif()

target_compile_definitions(DemoTarget PUBLIC GLEW_STATIC)
target_link_libraries(
  DemoTarget
  PUBLIC
  $<$<PLATFORM_ID:Windows>:Opengl32>
  $<$<PLATFORM_ID:Windows>:glu32.lib>
  glfw
  GLEW::glew_s
)

target_include_directories(
  DemoTarget
  PUBLIC
)