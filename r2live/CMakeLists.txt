# cmake_minimum_required(VERSION 2.8.3)
# project(r2live)
cmake_minimum_required(VERSION 3.0.2)
if(COMMAND cmake_policy)
  cmake_policy(SET CMP0048 NEW)
endif()
project(r2live VERSION 0.1.0)

# ADD_COMPILE_OPTIONS(-std=c++14 )
# set(CMAKE_BUILD_TYPE "Release")
# set(CMAKE_CXX_FLAGS "-std=c++14")
# #-DEIGEN_USE_MKL_ALL")
# set(CMAKE_CXX_FLAGS_RELEASE "-O3")

set(CMAKE_BUILD_TYPE "Release")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -std=c++17 -O3 -w")
set (CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

add_definitions(-DROOT_DIR=\"${CMAKE_CURRENT_SOURCE_DIR}/\")

find_package(catkin REQUIRED COMPONENTS
    roscpp
    std_msgs
    geometry_msgs
    nav_msgs
    tf
    cv_bridge
    message_generation
    livox_ros_driver
    )


find_package(OpenMP QUIET)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}   ${OpenMP_C_FLAGS}")
find_package(OpenCV REQUIRED)

# message(WARNING "OpenCV_VERSION: ${OpenCV_VERSION}")
list(APPEND CMAKE_PREFIX_PATH "/usr/local/lib/cmake/Ceres")

find_package(Ceres REQUIRED)
# message(WARNING "Ceres_VERSION: ${Ceres_VERSION}")
# message(WARNING "CERES_INCLUDE_DIRS: ${CERES_INCLUDE_DIRS}")
find_package(PCL REQUIRED)

find_package(PythonLibs REQUIRED)

include_directories(${catkin_INCLUDE_DIRS} ${CERES_INCLUDE_DIRS})

# include_directories(/usr/local/include/ceres)
# link_directories(/usr/local/lib)

add_message_files(
  FILES
  Pose6D.msg
  States.msg
)

generate_messages(
 DEPENDENCIES
 geometry_msgs
)

set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)
find_package(Eigen3)
include_directories(
  ${catkin_INCLUDE_DIRS}
  ${EIGEN3_INCLUDE_DIR}
  ${PCL_INCLUDE_DIRS}
  ${PYTHON_INCLUDE_DIRS}
  ./src/fast_lio/include
  ./src/tools/
)

catkin_package()

add_executable(lio_feat_extract src/fast_lio/feature_extract.cpp)
target_link_libraries(lio_feat_extract ${catkin_LIBRARIES} ${PCL_LIBRARIES})

add_executable(r2live
    src/estimator_node.cpp
    src/parameters.cpp
    src/estimator.cpp
    src/estimator_optimization.cpp
    src/feature_manager.cpp
    src/factor/pose_local_parameterization.cpp
    src/factor/projection_factor.cpp
    src/factor/projection_td_factor.cpp
    src/utility/utility.cpp
    src/utility/visualization.cpp
    src/utility/CameraPoseVisualization.cpp
    src/initial/solve_5pts.cpp
    src/initial/initial_aligment.cpp
    src/initial/initial_sfm.cpp
    src/initial/initial_ex_rotation.cpp
    src/fast_lio/include/kd_tree/ikd_Tree.cpp
    src/fast_lio/include/FOV_Checker/FOV_Checker.cpp 
    src/fast_lio/IMU_Processing.cpp
    )

target_link_libraries(r2live ${catkin_LIBRARIES} ${OpenCV_LIBS} ${CERES_LIBRARIES}) 


