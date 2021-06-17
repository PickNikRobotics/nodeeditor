cmake_minimum_required(VERSION 3.4)
# version 3.4 is required as other do not work with C++14 and clang

project(NodeEditor CXX)

set(CMAKE_DISABLE_IN_SOURCE_BUILD ON)
set(CMAKE_DISABLE_SOURCE_CHANGES  ON)

option(BUILD_STATIC_LIBS "Build static library" OFF)
option(BUILD_SHARED_LIBS "Build shared library" ON)



find_package(ament_cmake REQUIRED)
# Find the QtWidgets library
find_package(Qt5 5.3 COMPONENTS
             Core
             Widgets
             Gui
             OpenGL
             Svg
             Xml REQUIRED)

qt5_add_resources(RESOURCES ./resources/resources.qrc)

# Unfortunately, as we have a split include/src, AUTOMOC doesn't work.
# We'll have to manually specify some files
set(CMAKE_AUTOMOC ON)

set(CPP_SOURCE_FILES
  src/Connection.cpp
  src/ConnectionBlurEffect.cpp
  src/ConnectionGeometry.cpp
  src/ConnectionGraphicsObject.cpp
  src/ConnectionPainter.cpp
  src/ConnectionState.cpp
  src/ConnectionStyle.cpp
  src/DataModelRegistry.cpp
  src/FlowScene.cpp
  src/FlowView.cpp
  src/FlowViewStyle.cpp
  src/Node.cpp
  src/NodeConnectionInteraction.cpp
  src/NodeDataModel.cpp
  src/NodeGeometry.cpp
  src/NodeGraphicsObject.cpp
  src/NodePainter.cpp
  src/NodeState.cpp
  src/NodeStyle.cpp
  src/Properties.cpp
  src/StyleCollection.cpp
)

set(THIS_PACKAGE_INCLUDE_DEPENDS
  Qt5
)

# If we want to give the option to build a static library,
# set BUILD_SHARED_LIBS option to OFF
add_library(nodes SHARED
  ${CPP_SOURCE_FILES}
  ${RESOURCES}
)

ament_target_dependencies(nodes ${THIS_PACKAGE_INCLUDE_DEPENDS})


add_library(NodeEditor::nodes ALIAS nodes)

target_include_directories(nodes
  PUBLIC
    $<INSTALL_INTERFACE:include>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  PRIVATE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/nodes/internal>
)

target_link_libraries(nodes
  PUBLIC
    Qt5::Core
    Qt5::Widgets
    Qt5::Gui
    Qt5::OpenGL
    Qt5::Xml
    Qt5::Svg
)

target_compile_definitions(nodes
  PUBLIC
    ${Qt5Widgets_DEFINITIONS}
    NODE_EDITOR_SHARED
  PRIVATE
    NODE_EDITOR_EXPORTS
    #NODE_DEBUG_DRAWING
)

target_compile_options(nodes
  PRIVATE
    $<$<CXX_COMPILER_ID:MSVC>:/W4 /wd4127 /EHsc>
    $<$<CXX_COMPILER_ID:GNU>:-Wall -Wextra -std=c++11>
    $<$<CXX_COMPILER_ID:Clang>:-Wall -Wextra -std=c++11>
)

target_compile_features(nodes
  PUBLIC
    cxx_generic_lambdas # Require C++14
)

# set_target_properties(nodes
#   PROPERTIES
#     ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib
#     LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib
#     RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin
# )

######
# Moc
##

file(GLOB_RECURSE HEADERS_TO_MOC include/nodes/internal/*.hpp)

qt5_wrap_cpp(nodes_moc
    ${HEADERS_TO_MOC}
  TARGET nodes
  OPTIONS --no-notes # Don't display a note for the headers which don't produce a moc_*.cpp
)

target_sources(nodes PRIVATE ${nodes_moc})

###########
# Examples
##

set(BUILD_EXAMPLES false)
if(BUILD_EXAMPLES)
  add_subdirectory(examples)
endif()

##################
# Automated Tests
##

set(BUILD_TESTING false)
if(BUILD_TESTING)
  add_subdirectory(test)
endif()

###############
# Installation
##

include(GNUInstallDirs)

set(INSTALL_CONFIGDIR lib/cmake/NodeEditor)

install(TARGETS nodes
  EXPORT export_nodes
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  INCLUDES DESTINATION include
)

install(DIRECTORY include/ DESTINATION include)

install(EXPORT export_nodes
  FILE NodeEditorTargets.cmake
  NAMESPACE NodeEditor::
  DESTINATION ${INSTALL_CONFIGDIR}
)

include(CMakePackageConfigHelpers)

configure_package_config_file(${CMAKE_CURRENT_LIST_DIR}/cmake/NodeEditorConfig.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/NodeEditorConfig.cmake
  INSTALL_DESTINATION ${INSTALL_CONFIGDIR}
)

install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/NodeEditorConfig.cmake
  DESTINATION ${INSTALL_CONFIGDIR}
)

ament_export_targets(export_nodes HAS_LIBRARY_TARGET)
ament_export_dependencies(${THIS_PACKAGE_INCLUDE_DEPENDS})
ament_package()