cmake_minimum_required(VERSION 3.0.2)
project(ORBSLAM3)

## Compile as C++11, supported in ROS Kinetic and newer
# add_compile_options(-std=c++11)
IF(NOT CMAKE_BUILD_TYPE)
  SET(CMAKE_BUILD_TYPE Release)
ENDIF()

MESSAGE("Build type: " ${CMAKE_BUILD_TYPE})

find_package(catkin REQUIRED COMPONENTS
  cv_bridge
  image_transport
  roscpp
  sensor_msgs
  tf
)
catkin_package()

include_directories(
# include
  ${catkin_INCLUDE_DIRS}
)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}  -Wall   -O3")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall   -O3")
# set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -march=native")
# set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -march=native")
SET(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O3 -march=native")
SET(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -O3 -march=native")

# Check C++11 or C++0x support
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
if(COMPILER_SUPPORTS_CXX11)
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14")
   add_definitions(-DCOMPILEDWITHC11)
   message(STATUS "Using flag -std=c++14.")
elseif(COMPILER_SUPPORTS_CXX0X)
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
   add_definitions(-DCOMPILEDWITHC0X)
   message(STATUS "Using flag -std=c++0x.")
else()
   message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++14 support. Please use a different C++ compiler.")
endif()

set(OpenCV_DIR /home/zdf/cppLibraries/opencv-4.5.4/build)
find_package(OpenCV 4.4)
   if(NOT OpenCV_FOUND)
      message(FATAL_ERROR "OpenCV > 4.4 not found.")
   endif()
MESSAGE("OPENCV VERSION:")
MESSAGE(${OpenCV_VERSION})

find_package(Eigen3 3.1.0 REQUIRED)
set(Pangolin_DIR /home/zdf/cppLibraries/Pangolin/build)
find_package(Pangolin REQUIRED)
MESSAGE("Pangolin VERSION:")
MESSAGE(${Pangolin_VERSION})

include_directories(
${PROJECT_SOURCE_DIR}
${PROJECT_SOURCE_DIR}/include
${PROJECT_SOURCE_DIR}/include/CameraModels
${PROJECT_SOURCE_DIR}/Thirdparty/Sophus
${EIGEN3_INCLUDE_DIR}
${Pangolin_INCLUDE_DIRS}
)

add_library(${PROJECT_NAME} SHARED
${PROJECT_SOURCE_DIR}/src/src/System.cc
${PROJECT_SOURCE_DIR}/src/src/Tracking.cc
${PROJECT_SOURCE_DIR}/src/src/LocalMapping.cc
${PROJECT_SOURCE_DIR}/src/src/LoopClosing.cc
${PROJECT_SOURCE_DIR}/src/src/ORBextractor.cc
${PROJECT_SOURCE_DIR}/src/src/ORBmatcher.cc
${PROJECT_SOURCE_DIR}/src/src/FrameDrawer.cc
${PROJECT_SOURCE_DIR}/src/src/Converter.cc
${PROJECT_SOURCE_DIR}/src/src/MapPoint.cc
${PROJECT_SOURCE_DIR}/src/src/KeyFrame.cc
${PROJECT_SOURCE_DIR}/src/src/Atlas.cc
${PROJECT_SOURCE_DIR}/src/src/Map.cc
${PROJECT_SOURCE_DIR}/src/src/MapDrawer.cc
${PROJECT_SOURCE_DIR}/src/src/Optimizer.cc
${PROJECT_SOURCE_DIR}/src/src/Frame.cc
${PROJECT_SOURCE_DIR}/src/src/KeyFrameDatabase.cc
${PROJECT_SOURCE_DIR}/src/src/Sim3Solver.cc
${PROJECT_SOURCE_DIR}/src/src/Viewer.cc
${PROJECT_SOURCE_DIR}/src/src/ImuTypes.cc
${PROJECT_SOURCE_DIR}/src/src/G2oTypes.cc
${PROJECT_SOURCE_DIR}/src/src/CameraModels/Pinhole.cpp
${PROJECT_SOURCE_DIR}/src/src/CameraModels/KannalaBrandt8.cpp
${PROJECT_SOURCE_DIR}/src/src/OptimizableTypes.cpp
${PROJECT_SOURCE_DIR}/src/src/MLPnPsolver.cpp
${PROJECT_SOURCE_DIR}/src/src/GeometricTools.cc
${PROJECT_SOURCE_DIR}/src/src/TwoViewReconstruction.cc
${PROJECT_SOURCE_DIR}/src/src/Config.cc
${PROJECT_SOURCE_DIR}/src/src/Settings.cc
${PROJECT_SOURCE_DIR}/include/System.h
${PROJECT_SOURCE_DIR}/include/Tracking.h
${PROJECT_SOURCE_DIR}/include/LocalMapping.h
${PROJECT_SOURCE_DIR}/include/LoopClosing.h
${PROJECT_SOURCE_DIR}/include/ORBextractor.h
${PROJECT_SOURCE_DIR}/include/ORBmatcher.h
${PROJECT_SOURCE_DIR}/include/FrameDrawer.h
${PROJECT_SOURCE_DIR}/include/Converter.h
${PROJECT_SOURCE_DIR}/include/MapPoint.h
${PROJECT_SOURCE_DIR}/include/KeyFrame.h
${PROJECT_SOURCE_DIR}/include/Atlas.h
${PROJECT_SOURCE_DIR}/include/Map.h
${PROJECT_SOURCE_DIR}/include/MapDrawer.h
${PROJECT_SOURCE_DIR}/include/Optimizer.h
${PROJECT_SOURCE_DIR}/include/Frame.h
${PROJECT_SOURCE_DIR}/include/KeyFrameDatabase.h
${PROJECT_SOURCE_DIR}/include/Sim3Solver.h
${PROJECT_SOURCE_DIR}/include/Viewer.h
${PROJECT_SOURCE_DIR}/include/ImuTypes.h
${PROJECT_SOURCE_DIR}/include/G2oTypes.h
${PROJECT_SOURCE_DIR}/include/CameraModels/GeometricCamera.h
${PROJECT_SOURCE_DIR}/include/CameraModels/Pinhole.h
${PROJECT_SOURCE_DIR}/include/CameraModels/KannalaBrandt8.h
${PROJECT_SOURCE_DIR}/include/OptimizableTypes.h
${PROJECT_SOURCE_DIR}/include/MLPnPsolver.h
${PROJECT_SOURCE_DIR}/include/GeometricTools.h
${PROJECT_SOURCE_DIR}/include/TwoViewReconstruction.h
${PROJECT_SOURCE_DIR}/include/SerializationUtils.h
${PROJECT_SOURCE_DIR}/include/Config.h
${PROJECT_SOURCE_DIR}/include/Settings.h)

add_subdirectory(${PROJECT_SOURCE_DIR}/Thirdparty/g2o)

target_link_libraries(${PROJECT_NAME}
${OpenCV_LIBS}
${EIGEN3_LIBS}
${Pangolin_LIBRARIES}
${PROJECT_SOURCE_DIR}/Thirdparty/DBoW2/lib/libDBoW2.so
${PROJECT_SOURCE_DIR}/Thirdparty/g2o/lib/libg2o.so
-lboost_serialization
-lcrypto
)

# set(LIBS 
# # ${catkin_LIBRARIES}
# ${OpenCV_LIBS} 
# ${EIGEN3_LIBS}
# ${Pangolin_LIBRARIES}
# ${PROJECT_SOURCE_DIR}/Thirdparty/DBoW2/lib/libDBoW2.so
# ${PROJECT_SOURCE_DIR}/Thirdparty/g2o/lib/libg2o.so
# # ${PROJECT_SOURCE_DIR}/lib/libORB_SLAM3.so
# -lboost_system
# )

# target_link_libraries(${PROJECT_NAME}
# ${LIBS}
# -lboost_serialization
# )


add_executable(Mono_Inertial src/ros_mono_inertial.cc)
target_link_libraries(Mono_Inertial
${catkin_LIBRARIES}
# ${LIBS}
${PROJECT_NAME}
)

add_executable(Mono src/ros_mono.cc)
target_link_libraries(Mono
${catkin_LIBRARIES}
# ${LIBS}
${PROJECT_NAME}
)

# add_dependencies(${PROJECT_NAME}_node ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

# target_link_libraries(${PROJECT_NAME}_node
#   ${catkin_LIBRARIES}
# )

