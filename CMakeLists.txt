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
