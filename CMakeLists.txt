cmake_minimum_required(VERSION 3.20)
project(VulkanBasics CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
# 默认 Debug：没有定义 NDEBUG，验证层会打开
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Debug)
endif()

find_package(Vulkan REQUIRED)
find_package(glfw3 REQUIRED)
find_path(GLM_INCLUDE_DIR glm/glm.hpp REQUIRED)

add_executable(vulkan_app src/main.cpp)
target_include_directories(vulkan_app PRIVATE ${GLM_INCLUDE_DIR})
target_link_libraries(vulkan_app PRIVATE Vulkan::Vulkan glfw)

# 着色器：glslc 把 GLSL 编译成 SPIR-V，放到 build/shaders/（shader.vert -> vert.spv）
find_program(GLSLC glslc HINTS $ENV{VULKAN_SDK}/bin REQUIRED)
set(SHADER_SOURCES shaders/shader.vert shaders/shader.frag)
foreach(SRC ${SHADER_SOURCES})
  get_filename_component(STAGE ${SRC} LAST_EXT)
  string(SUBSTRING ${STAGE} 1 -1 STAGE)
  set(SPV ${CMAKE_BINARY_DIR}/shaders/${STAGE}.spv)
  add_custom_command(
    OUTPUT ${SPV}
    COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_BINARY_DIR}/shaders
    COMMAND ${GLSLC} ${CMAKE_SOURCE_DIR}/${SRC} -o ${SPV}
    DEPENDS ${SRC}
    COMMENT "Compiling ${SRC}")
  list(APPEND SPV_FILES ${SPV})
endforeach()
add_custom_target(shaders DEPENDS ${SPV_FILES})
add_dependencies(vulkan_app shaders)
