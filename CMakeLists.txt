##### HEADER #####
cmake_minimum_required(VERSION 2.8)
project(easyloggingpp)

###### IMPORT DEPENDENCIES (build_depends) ######

## System ##

###### ROS ######

# This is a top-level algorithm package that is ros-agnostic
# It should not have any ROS dependencies, but may have other ros-agnostic dependencies.
# We add a flag that lets us disable catkin so we can build without ROS.
set(CATKIN_ENABLED "TRUE" CACHE BOOL "Catkin will export package information (for use with ROS)")
if (CATKIN_ENABLED)
    # Adding packages as components adds them to our ${catkin_INCLUDE_DIRS} and ${catkin_LIBRARIES} variables.
    find_package(catkin REQUIRED
                 #COMPONENTS [ros_package_1] [ros_package_2]
    )
    ##### EXPORT MYSELF TO CATKIN #####
    #   "INCLUDE_DIRS" specifies to location of your package's headers for inclusion in inheriting ${catkin_INCLUDE_DIRS}.
    #   "LIBRARIES" specifies the libraries your package provides for inclusion in inheriting ${catkin_LIBRARIES}. This should match all add_library directives.
    #   "DEPENDS" specifies the system dependencies 
    #   "CATKIN_DEPENDS" specifies the ros dependencies
    catkin_package(
                   INCLUDE_DIRS .
                   LIBRARIES
                   ##Dependencies (build_depends and run_depends)
                   DEPENDS #[system_dependency]
                   #CATKIN_DEPENDS [ros_package]
    )
endif()

######################################################################################
### Set up install
######################################################################################

#setup installation locations
set(INSTALL_INCLUDE_DIR include CACHE PATH "Installation directory for header files")
if(WIN32 AND NOT CYGWIN)
    set(DEF_INSTALL_CMAKE_DIR CMake)
else()
    set(DEF_INSTALL_CMAKE_DIR lib/cmake/easyloggingpp)
endif()
set(INSTALL_CMAKE_DIR ${DEF_INSTALL_CMAKE_DIR} CACHE PATH "Installation directory for CMake files")

# Make relative paths absolute (needed later on)
foreach(p INCLUDE CMAKE)
    set(var INSTALL_${p}_DIR)
    if(NOT IS_ABSOLUTE "${${var}}")
        set(${var} "${CMAKE_INSTALL_PREFIX}/${${var}}")
    endif()
endforeach()

install(
  FILES ${CMAKE_CURRENT_SOURCE_DIR}/easylogging++.h
  DESTINATION ${INSTALL_INCLUDE_DIR}
)

######################################################################################
### Enable find_package()
######################################################################################

### Create the RobochunkConfig.cmake and RobochunkConfigVersion files

# Export the package for use from the build-tree
export(PACKAGE easyloggingpp)

# Create variable for the local build tree
get_property(CONF_INCLUDE_DIRS DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY INCLUDE_DIRECTORIES)

# Change the include location for the case of an install location
set(CONF_INCLUDE_DIRS ${CONF_INCLUDE_DIRS} ${INSTALL_INCLUDE_DIR})

# Create the <name>Config.cmake for the installation tree
configure_file(easyloggingppConfig.cmake.in
    "${PROJECT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/easyloggingppConfig.cmake" @ONLY)

# Install the easyloggingppConfig.cmake and RobochunkConfigVersion.cmake
install(FILES
    "${PROJECT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/easyloggingppConfig.cmake"
    DESTINATION "${INSTALL_CMAKE_DIR}" COMPONENT dev)

add_custom_target(uninstall
    COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)
