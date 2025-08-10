cmake_minimum_required(VERSION 4.0)

project (imgui VERSION 1.90.0 LANGUAGES CXX)

option(IMGUI_BACKEND_GLFW "Build GLFW Backend" ON)
option(IMGUI_BACKEND_OPENGL3 "Build OpenGL3 Backend" ON)
option(IMGUI_BACKEND_SDL2 "Build SDL2 Backend" OFF)

set(IMGUI_USER_CONFIG "" CACHE FILEPATH "Path to custom imconfig.h")

add_library(imgui STATIC)
add_library(imgui::imgui ALIAS imgui)

file(GLOB IMGUI_BACKEND_CPPS "${CMAKE_CURRENT_SOURCE_DIR}/src/backends/*.cpp")
foreach(_f IN LISTS IMGUI_BACKEND_CPPS)
    file(READ "${_f}" _contents)
    if(_contents MATCHES "\\.\\./\\.\\./backends/")
        message(FATAL_ERROR "Legacy relative include found in ${_f}. Use <imgui/backends/...> instead.")
    endif()
endforeach()

target_sources(imgui
        PRIVATE
        src/imgui.cpp
        src/imgui_demo.cpp
        src/imgui_draw.cpp
        src/imgui_tables.cpp
        src/imgui_widgets.cpp)

if(IMGUI_BACKEND_GLFW)
    target_sources(imgui PRIVATE src/backends/imgui_impl_glfw.cpp)
    target_compile_definitions(imgui PUBLIC IMGUI_HAS_GLFW=1)
    target_link_libraries(imgui PUBLIC glfw)
    target_compile_definitions(imgui PUBLIC IMGUI_HAS_GLFW=1)
endif()

if(IMGUI_BACKEND_OPENGL3)
    target_sources(imgui PRIVATE src/backends/imgui_impl_opengl3.cpp)
    target_compile_definitions(imgui PUBLIC IMGUI_HAS_OPENGL3=1)
    # If you standardise on a GL loader, expose the define here, e.g. GLAD:
    # target_link_libraries(imgui PUBLIC glad::glad)
    # target_compile_definitions(imgui PUBLIC IMGUI_IMPL_OPENGL_LOADER_GLAD=1)
endif()

if(IMGUI_BACKEND_SDL2)
    target_sources(imgui PRIVATE src/backends/imgui_impl_sdl2.cpp)
    target_compile_definitions(imgui PUBLIC IMGUI_HAS_SDL2=1)
    # Optionally: find_package(SDL2 CONFIG REQUIRED) ; target_link_libraries(imgui PUBLIC SDL2::SDL2)
endif()

target_include_directories(imgui
        PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>)

if(IMGUI_USER_CONFIG)
    target_compile_definitions(imgui PUBLIC IMGUI_USER_CONFIG="${IMGUI_USER_CONFIG}")
endif()

target_compile_features(imgui PUBLIC cxx_std_11)

include(CMakePackageConfigHelpers)

install(TARGETS imgui
        EXPORT imguiTargets
        ARCHIVE DESTINATION lib
        LIBRARY DESTINATION lib
        RUNTIME DESTINATION bin
        INCLUDES DESTINATION include)

install(DIRECTORY include/ DESTINATION include)

write_basic_package_version_file(
        "${CMAKE_CURRENT_BINARY_DIR}/imguiConfigVersion.cmake"
        VERSION ${PROJECT_VERSION}
        COMPATIBILITY SameMajorVersion)

configure_file(
        ${CMAKE_CURRENT_SOURCE_DIR}/cmake/imguiConfig.cmake.in
        ${CMAKE_CURRENT_BINARY_DIR}/imguiConfig.cmake
        @ONLY)

install(EXPORT imguiTargets
        NAMESPACE imgui::
        FILE imguiTargets.cmake
        DESTINATION lib/cmake/imgui)

install(FILES
        "${CMAKE_CURRENT_BINARY_DIR}/imguiConfig.cmake"
        "${CMAKE_CURRENT_BINARY_DIR}/imguiConfigVersion.cmake"
        DESTINATION lib/cmake/imgui)