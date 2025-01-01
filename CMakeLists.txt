cmake_minimum_required(VERSION 3.5)
project(bspline)

# 默认使用C++14标准
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 17)
endif()

# 设置构建类型为Release，并配置编译选项
set(CMAKE_BUILD_TYPE "Release")
set(CMAKE_CXX_FLAGS_RELEASE "-O3 -Wall -g")

# 查找必要的包
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(std_msgs REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(PCL REQUIRED COMPONENTS common io) # 根据需要添加PCL组件
find_package(rosidl_default_generators REQUIRED)

# 生成消息文件
rosidl_generate_interfaces(${PROJECT_NAME}
  "msg/Bspline.msg"
  DEPENDENCIES std_msgs geometry_msgs builtin_interfaces
)

# 包含目录
include_directories(
    include
    ${EIGEN3_INCLUDE_DIRS}
    ${PCL_INCLUDE_DIRS}
)

# 添加库
add_library(non_uniform_bspline
    src/non_uniform_bspline.cpp
)

# 指定库的依赖关系
ament_target_dependencies(non_uniform_bspline
    rclcpp
    std_msgs
    geometry_msgs
    Eigen3
    builtin_interfaces
)

# 链接PCL库
target_link_libraries(non_uniform_bspline
    ${PCL_LIBRARIES}
)

# 安装目标
install(TARGETS non_uniform_bspline
  EXPORT export_${PROJECT_NAME}
  LIBRARY DESTINATION lib
)

# 安装头文件
install(DIRECTORY include/
  DESTINATION include/
)

# 确保消息类型支持在编译时生成
rosidl_get_typesupport_target(cpp_typesupport_target "${PROJECT_NAME}" "rosidl_typesupport_cpp")
add_dependencies(non_uniform_bspline ${cpp_typesupport_target})

# 导出依赖关系
ament_export_dependencies(rosidl_default_runtime)
ament_export_targets(export_${PROJECT_NAME})

# 结束ament包配置
ament_package()
