cmake_minimum_required(VERSION 3.2)

##
# Hunter Cross-platform package manager
##
include("cmake/HunterGate.cmake")
HunterGate(
    URL "https://github.com/ruslo/hunter/archive/v0.22.19.tar.gz"
    SHA1 "54749ffc945f78362aa680a7abbafcaa40ff9069"
)

project(glsl-optimizer VERSION 0.1)

find_package(OpenGL REQUIRED)

file(GLOB glcpp-library_sources src/glsl/glcpp/*.c src/util/*.c)
#file(GLOB glcpp-library_sources_remove src/glsl/glcpp/glcpp.c)
#list(REMOVE_ITEM glcpp-library_sources ${glcpp-library_sources_remove})
add_library(glcpp-library ${glcpp-library_sources})
target_include_directories(glcpp-library PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include> $<INSTALL_INTERFACE:include> PRIVATE src src/mesa)

file(GLOB mesa_sources src/mesa/program/*.c src/mesa/main/*.c)
add_library(mesa ${mesa_sources})
target_include_directories(mesa PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include> $<INSTALL_INTERFACE:include> PRIVATE src src/mesa)

file(GLOB glsl_sources src/glsl/*.cpp src/glsl/*.c)
file(GLOB glsl_sources_remove src/glsl/main.cpp src/glsl/builtin_stubs.cpp)
list(REMOVE_ITEM glsl_sources ${glsl_sources_remove})
add_library(glsl_optimizer ${glsl_sources})
target_link_libraries(glsl_optimizer PUBLIC glcpp-library mesa ${OPENGL_LIBRARIES})
target_include_directories(glsl_optimizer PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include> $<INSTALL_INTERFACE:include> PRIVATE src src/mesa)

if(MSVC)
    add_executable(glsl_compiler src/glsl/main.cpp src/wingetopt/getopt.c)
    target_include_directories(glsl_compiler PRIVATE src/wingetopt)
else()
    add_executable(glsl_compiler src/glsl/main.cpp)
endif()
target_link_libraries(glsl_compiler glsl_optimizer)
target_include_directories(glsl_compiler PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include> $<INSTALL_INTERFACE:include> PRIVATE src src/glsl src/mesa)

file(GLOB glsl_test_sources tests/*.cpp)
add_executable(glsl_test ${glsl_test_sources})
target_link_libraries(glsl_test glsl_optimizer)

file(GLOB glslopt_sources contrib/glslopt/*.cpp)
add_executable(glslopt ${glslopt_sources})
target_link_libraries(glslopt glsl_optimizer)
target_include_directories(glslopt PRIVATE src src/glsl)

#add_executable(glcpp src/glsl/glcpp/glcpp.c)
#target_link_libraries(glcpp glsl_optimizer)

set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")
set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

file(GLOB glsl_optimizer_src_glsl_headers "src/glsl/*.h")
install(FILES ${glsl_optimizer_src_glsl_headers} DESTINATION include/glsl)

install(TARGETS glsl_optimizer glcpp-library mesa
  EXPORT "${targets_export_name}"
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  PUBLIC_HEADER DESTINATION include
  FRAMEWORK DESTINATION Library/Frameworks
)

include(CMakePackageConfigHelpers)

write_basic_package_version_file(
    "${version_config}" COMPATIBILITY AnyNewerVersion
)

# Use 'targets_export_name'
configure_package_config_file(
    "cmake/Config.cmake.in"
    "${project_config}"
    INSTALL_DESTINATION "${config_install_dir}"
)

install(
    FILES "${project_config}" "${version_config}"
    DESTINATION "${config_install_dir}"
)

install(
    EXPORT "${targets_export_name}"
    NAMESPACE "${namespace}"
    DESTINATION "${config_install_dir}"
)

