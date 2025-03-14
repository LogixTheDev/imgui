cmake_minimum_required(VERSION 3.30.2)
project(ImGui)

set(IMGUI_SOURCES src/imgui.cpp
                  src/imgui_draw.cpp
                  src/imgui_tables.cpp
                  src/imgui_widgets.cpp
                  src/backends/imgui_impl_glfw.cpp
                  src/backends/imgui_impl_opengl3.cpp
                  )

add_library(ImGui STATIC ${IMGUI_SOURCES})

target_include_directories(ImGui PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)