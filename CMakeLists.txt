cmake_minimum_required(VERSION 2.8.3)
# the project name should match that of the external project (kinda sketchy)
project(easyloggingpp)

find_package(catkin REQUIRED)

include(ExternalProject)

# the name here doesn't really matter, but it's used further below
set(EP_NAME easyloggingpp)
ExternalProject_Add(${EP_NAME}
  # prefix for the external project in the build folder
  PREFIX external
  # the source
  URL https://github.com/easylogging/easyloggingpp/archive/v9.83.zip
  # don't update, we lock the version
  UPDATE_COMMAND ""
  # Fix the required cmake version. To generate: diff -Naur old-src new-src > patchfile
  PATCH_COMMAND echo "Patching easylogging ..." && patch -p1 -tN < ${CMAKE_CURRENT_SOURCE_DIR}/patchfile && echo "done."
  # Install where catkin wants us to
  INSTALL_DIR ${CATKIN_DEVEL_PREFIX}/${PROJECT_NAME}
  # Pass the install dir on to cmake
  CMAKE_ARGS -DCMAKE_INSTALL_PREFIX:string=<INSTALL_DIR>
  # $(MAKE) uses the catkin jobserver (vs make)
  BUILD_COMMAND $(MAKE)
  # Don't install yet, wait for catkin to install
  INSTALL_COMMAND $(MAKE) install && rsync -r ../${EP_NAME}/cmake <INSTALL_DIR>
)

# generate our PackageConfig.cmake and -version.cmake
catkin_package()

# grab the source and install dir from the external project
ExternalProject_Get_Property(${EP_NAME} SOURCE_DIR)
ExternalProject_Get_Property(${EP_NAME} INSTALL_DIR)

# Helpful variables
SET(PACKAGE_INSTALL_PREFIX ${CMAKE_INSTALL_PREFIX}/${PROJECT_NAME})
SET(PC_NAME ${PROJECT_NAME}Config.cmake)

# generate PackageConfig.cmake that points to the FindModule.cmake
set(GENERATE_DIR ${CMAKE_CURRENT_BINARY_DIR}/catkin_generated/installspace)
set(CONFIG_DIR ${INSTALL_DIR}) # used in Config.cmake.in
# one for build, that overwrites the catkin one
configure_file(Config.cmake.in ${GENERATE_DIR}/${PC_NAME} @ONLY)
# one for devel, that overwrites the catkin one
configure_file(Config.cmake.in ${CATKIN_DEVEL_PREFIX}/share/${PROJECT_NAME}/cmake/${PC_NAME} @ONLY)
# one for install, that will be installed over the catkin one
set(CONFIG_DIR ${PACKAGE_INSTALL_PREFIX}) # used in Config.cmake.in
set(INSTALL_PC ${GENERATE_DIR}/install/${PC_NAME})
configure_file(Config.cmake.in ${INSTALL_PC} @ONLY)

# install the package
install(DIRECTORY ${INSTALL_DIR}/ DESTINATION ${PACKAGE_INSTALL_PREFIX})
# install our PackageConfig.cmake that points to the install folder, removing catkin's :(
install(CODE "file(REMOVE ${CMAKE_INSTALL_PREFIX}/share/${PROJECT_NAME}/cmake/${PC_NAME})")
install(FILES ${INSTALL_PC} DESTINATION ${CMAKE_INSTALL_PREFIX}/share/${PROJECT_NAME}/cmake)
