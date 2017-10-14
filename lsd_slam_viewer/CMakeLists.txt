cmake_minimum_required(VERSION 2.4.6)
project(lsd_slam_viewer)
#include($ENV{ROS_ROOT}/core/rosbuild/rosbuild.cmake)

# Set the build type.  Options are:
#  Coverage       : w/ debug symbols, w/o optimization, w/ code-coverage
#  Debug          : w/ debug symbols, w/o optimization
#  Release        : w/o debug symbols, w/ optimization
#  RelWithDebInfo : w/ debug symbols, w/ optimization
#  MinSizeRel     : w/o debug symbols, w/ optimization, stripped binaries
#set(ROS_BUILD_TYPE Release)

# NOTES
# sudo apt-get -y install libqglviewer-dev

#rosbuild_init()

ADD_SUBDIRECTORY(${PROJECT_SOURCE_DIR}/thirdparty/Sophus)

#set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)
#set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)
#set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${PROJECT_SOURCE_DIR}/cmake)
LIST(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake")

find_package(catkin REQUIRED
  roscpp
  gencpp
  rospy
  genpy
  rosbag # for main_bagfile_driver
  cmake_modules
  cv_bridge
  eigen_conversions
  geometry_msgs
  sensor_msgs
  image_geometry
  image_transport
  nodelet
  std_msgs
  tf
  tf2
  tf2_ros
  cmake_modules
  dynamic_reconfigure
  message_generation
  message_runtime)

find_package(cmake_modules REQUIRED)   
find_package(OpenGL REQUIRED)
set(QT_USE_QTOPENGL TRUE)
set(QT_USE_QTXML TRUE)
find_package(QGLViewer REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(Boost REQUIRED)
find_package(OpenCV REQUIRED)

generate_dynamic_reconfigure_options(cfg/LSDSLAMViewerParams.cfg)

add_message_files(
  FILES
  keyframeMsg.msg
  keyframeGraphMsg.msg
)

generate_messages(
  DEPENDENCIES
  std_msgs
)

catkin_package(
#  INCLUDE_DIRS include
#  LIBRARIES uncc_rgbd_slam
  CATKIN_DEPENDS message_runtime
  DEPENDS Boost Eigen3 OpenCV 
  LIBRARIES ${PROJECT_NAME}
)

include_directories(
  ${PROJECT_SOURCE_DIR}/cfg/cpp
  ${PROJECT_SOURCE_DIR}/thirdparty/Sophus
  ${catkin_INCLUDE_DIRS} 
#  ${sophus_INCLUDE_DIRS}
  ${Boost_INCLUDE_DIRS}
  ${OpenCV_INCLUDE_DIRS}
  ${QT_INCLUDES} 
  ${EIGEN3_INCLUDE_DIRS} 
  ${QGLVIEWER_INCLUDE_DIR})
#include_directories(
#  ${PROJECT_SOURCE_DIR}/thirdparty/Sophus
#)  

#rosbuild_find_ros_package(dynamic_reconfigure)
#include(${dynamic_reconfigure_PACKAGE_PATH}/cmake/cfgbuild.cmake)
#gencfg()

# Messages & Services
#rosbuild_genmsg()


# SSE flags
#rosbuild_check_for_sse()
#set(CMAKE_CXX_FLAGS
#   "${SSE_FLAGS}"
#)

# SSE Sources files
set(SOURCE_FILES         
  src/PointCloudViewer.cpp
  src/KeyFrameDisplay.cpp
  src/KeyFrameGraphDisplay.cpp
  src/settings.cpp
)

set(HEADER_FILES     
  src/PointCloudViewer.h
  src/KeyFrameDisplay.h
  src/KeyFrameGraphDisplay.h
  src/settings.h
)

add_library(${PROJECT_NAME}
  ${SOURCE_FILES})
add_dependencies(${PROJECT_NAME} ${PROJECT_NAME}_gencfg ${PROJECT_NAME}_generate_messages_cpp ${PROJECT_NAME}_generate_messages_py)

target_link_libraries(${PROJECT_NAME} 
  ${catkin_LIBRARIES} 
  ${Boost_LIBRARIES} 
  ${OpenCV_LIBRARIES}  
  ${Eigen3_LIBRARIES})

add_executable(viewer 
    src/main_viewer.cpp ${SOURCE_FILES})
add_dependencies(viewer ${PROJECT_NAME}_gencfg ${PROJECT_NAME}_generate_messages_cpp ${PROJECT_NAME}_generate_messages_py)
#    ${HEADER_FILES})

target_link_libraries(viewer 
   ${catkin_LIBRARIES}  
   ${Boost_LIBRARIES} 
   ${OpenCV_LIBRARIES}
   ${Eigen3_LIBRARIES}
   ${QGLViewer_LIBRARIES}
   ${QT_LIBRARIES} 
   ${QGLVIEWER_LIBRARY} 
   GL glut GLU)

#rosbuild_link_boost(viewer thread)

add_executable(videoStitch src/main_stitchVideos.cpp)
add_dependencies(videoStitch ${PROJECT_NAME}_gencfg ${PROJECT_NAME}_generate_messages_cpp ${PROJECT_NAME}_generate_messages_py)
target_link_libraries(videoStitch 
   ${catkin_LIBRARIES}  
   ${Boost_LIBRARIES} 
   ${OpenCV_LIBRARIES}
   ${Eigen3_LIBRARIES}
   ${QGLViewer_LIBRARIES}
   ${QT_LIBRARIES} 
   ${QGLVIEWER_LIBRARY} 
   GL glut GLU)
