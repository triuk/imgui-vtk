cmake_minimum_required(VERSION 3.14.0 FATAL_ERROR)
project(imgui-vtk)

set(CMAKE_CXX_STANDARD 11)

# CPM
set(CPM_DOWNLOAD_VERSION 0.36.0)

if(CPM_SOURCE_CACHE)
  set(CPM_DOWNLOAD_LOCATION "${CPM_SOURCE_CACHE}/cpm/CPM_${CPM_DOWNLOAD_VERSION}.cmake")
elseif(DEFINED ENV{CPM_SOURCE_CACHE})
  set(CPM_DOWNLOAD_LOCATION "$ENV{CPM_SOURCE_CACHE}/cpm/CPM_${CPM_DOWNLOAD_VERSION}.cmake")
else()
  set(CPM_DOWNLOAD_LOCATION "${CMAKE_BINARY_DIR}/cmake/CPM_${CPM_DOWNLOAD_VERSION}.cmake")
endif()

if(NOT(EXISTS ${CPM_DOWNLOAD_LOCATION}))
  message(STATUS "Downloading CPM.cmake to ${CPM_DOWNLOAD_LOCATION}")
  file(DOWNLOAD
    https://github.com/TheLartians/CPM.cmake/releases/download/v${CPM_DOWNLOAD_VERSION}/CPM.cmake
    ${CPM_DOWNLOAD_LOCATION}
  )
endif()

include(${CPM_DOWNLOAD_LOCATION})

# OpenGL
find_package(OpenGL REQUIRED)

# gl3w
CPMAddPackage(
  NAME gl3w_D
  GITHUB_REPOSITORY skaslev/gl3w
  GIT_TAG origin/master
  DOWNLOAD_ONLY
)

set(GL3W_DIR ${CPM_PACKAGE_gl3w_D_SOURCE_DIR})
include_directories(${GL3W_DIR}/include)
set(GL3W_SRC ${GL3W_DIR}/src/gl3w.c)

if(NOT(EXISTS ${GL3W_SRC}))
  execute_process(COMMAND python3 gl3w_gen.py WORKING_DIRECTORY ${GL3W_DIR})
endif()

add_library(gl3w ${GL3W_SRC})

# ImGui
find_package(imgui QUIET)

if(NOT imgui_FOUND)
  CPMAddPackage(
    NAME imgui
    GITHUB_REPOSITORY ocornut/imgui
    GIT_TAG origin/docking
  )
endif()

set(IMGUI_DIR ${CPM_PACKAGE_imgui_SOURCE_DIR})
include_directories(${IMGUI_DIR} ${IMGUI_DIR}/backends)
set(IMGUI_SRC ${IMGUI_DIR}/imgui.cpp ${IMGUI_DIR}/imgui_demo.cpp ${IMGUI_DIR}/imgui_draw.cpp ${IMGUI_DIR}/imgui_tables.cpp ${IMGUI_DIR}/imgui_widgets.cpp ${IMGUI_DIR}/backends/imgui_impl_glfw.cpp ${IMGUI_DIR}/backends/imgui_impl_opengl3.cpp)
add_library(imgui ${IMGUI_SRC})

# VTK
find_package(VTK COMPONENTS
  CommonCore
  CommonColor
  CommonDataModel
  FiltersCore
  InteractionStyle
  RenderingCore
  RenderingFreeType
  RenderingGL2PSOpenGL2
  RenderingOpenGL2
  QUIET
)

# imgui-vtk (VTK Viewer class)
include_directories(src)
add_library(imgui_vtk_viewer src/VtkViewer.cpp)
target_link_libraries(imgui_vtk_viewer ${VTK_LIBRARIES})

# vtk_module_autoinit is needed
vtk_module_autoinit(
  TARGETS imgui_vtk_viewer
  MODULES ${VTK_LIBRARIES}
)
