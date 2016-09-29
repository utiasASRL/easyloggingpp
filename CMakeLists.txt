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
  # Pass the install dir on to cmake
  CMAKE_ARGS -DCMAKE_INSTALL_PREFIX:string=<INSTALL_DIR>
  # $(MAKE) uses the catkin jobserver (vs make)
  BUILD_COMMAND $(MAKE)
  # $(MAKE) again here for the jobserver
  INSTALL_COMMAND $(MAKE) install
)

# generate our PackageConfig.cmake and -version.cmake
catkin_package()

# grab the source and install dir from the external project
ExternalProject_Get_Property(${EP_NAME} SOURCE_DIR)
ExternalProject_Get_Property(${EP_NAME} INSTALL_DIR)
# overwrite Catkin's PackageConfig.cmake with our own that points to FindModule.cmake
#./build/easyloggingpp_catkin/catkin_generated/installspace/easyloggingpp_catkinConfig.cmake
configure_file(Config.cmake.in ${CMAKE_CURRENT_BINARY_DIR}/catkin_generated/installspace/${PROJECT_NAME}Config.cmake @ONLY)
configure_file(Config.cmake.in ${CATKIN_DEVEL_PREFIX}/share/${PROJECT_NAME}/cmake/${PROJECT_NAME}Config.cmake @ONLY)
