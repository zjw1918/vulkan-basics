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
target_include_directories(vulkan_app PRIVATE ${GLM_INCLUDE_DIR} third_party)
target_link_libraries(vulkan_app PRIVATE Vulkan::Vulkan glfw)

# 第十二期：计算着色器粒子，另一个程序
add_executable(particles src/particles.cpp)
target_include_directories(particles PRIVATE ${GLM_INCLUDE_DIR})
target_link_libraries(particles PRIVATE Vulkan::Vulkan glfw)

# 着色器：glslc 把 GLSL 编译成 SPIR-V，放到 build/shaders/
#   shader.vert -> vert.spv，particles.comp -> particles_comp.spv
find_program(GLSLC glslc HINTS $ENV{VULKAN_SDK}/bin REQUIRED)
set(SHADER_SOURCES shaders/shader.vert shaders/shader.frag shaders/particles.vert shaders/particles.frag
    shaders/particles.comp)
foreach(SRC ${SHADER_SOURCES})
  get_filename_component(NAME ${SRC} NAME_WE)
  get_filename_component(STAGE ${SRC} LAST_EXT)
  string(SUBSTRING ${STAGE} 1 -1 STAGE)
  if(NAME STREQUAL "shader")
    set(SPV ${CMAKE_BINARY_DIR}/shaders/${STAGE}.spv)
  else()
    set(SPV ${CMAKE_BINARY_DIR}/shaders/${NAME}_${STAGE}.spv)
  endif()
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
add_dependencies(particles shaders)

# 贴图和模型：复制到 build/textures/、build/models/，程序按相对路径读取
add_custom_command(TARGET vulkan_app POST_BUILD
  COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_SOURCE_DIR}/textures ${CMAKE_BINARY_DIR}/textures
  COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_SOURCE_DIR}/models ${CMAKE_BINARY_DIR}/models)
