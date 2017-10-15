cmake_minimum_required(VERSION 2.4.6)
project(lsd_slam_viewer)

# NOTES
# sudo apt-get -y purge libqglviewer2 libqglviewer-dev
# sudo apt-get -y install libqglviewer2-qt4 libqglviewer-dev-qt4

ADD_SUBDIRECTORY(${PROJECT_SOURCE_DIR}/thirdparty/Sophus)

LIST(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake")

find_package(catkin REQUIRED
  roscpp
  gencpp
  rospy
  genpy
  rosbag # for main_bagfile_driver
  cmake_modules
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
find_package(OpenGL REQUIRED)
find_package(GLUT REQUIRED)

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
  DEPENDS Boost OpenCV 
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
  ${OPENCV_INCLUDE_DIR}
  ${OPENGL_INCLUDE_DIR} 
  ${QGLVIEWER_INCLUDE_DIR})

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

target_link_libraries(viewer 
   ${catkin_LIBRARIES}  
   ${Boost_LIBRARIES} 
   ${OpenCV_LIBRARIES}
   ${Eigen3_LIBRARIES}
   ${QGLViewer_LIBRARIES}
   ${QT_LIBRARIES} 
#   ${QGLVIEWER_LIBRARY} 
   ${OPENGL_LIBRARIES}  
   ${GLUT_LIBRARIES}
   QGLViewer-qt4)

add_executable(videoStitch src/main_stitchVideos.cpp)
add_dependencies(videoStitch ${PROJECT_NAME}_gencfg ${PROJECT_NAME}_generate_messages_cpp ${PROJECT_NAME}_generate_messages_py)
target_link_libraries(videoStitch 
   ${catkin_LIBRARIES}  
   ${Boost_LIBRARIES} 
   ${OpenCV_LIBRARIES}
   ${Eigen3_LIBRARIES}
   ${QGLViewer_LIBRARIES}
   ${QT_LIBRARIES} 
#   ${QGLVIEWER_LIBRARY} 
   ${OPENGL_LIBRARIES}  
   ${GLUT_LIBRARIES}
   QGLViewer-qt4)
