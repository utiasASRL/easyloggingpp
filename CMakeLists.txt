cmake_minimum_required(VERSION 2.8.3)
# the project name should match that of the external project (kinda sketchy)
project(easyloggingpp)

find_package(catkin_optional REQUIRED)
co_build_type() # cache the build type

include(ExternalProject)

# version for downloading and package config
set(MAJOR_VERSION 9)
set(MINOR_VERSION 83)
set(PATCH_VERSION 0)
set(PROJECT_VERSION ${MAJOR_VERSION}.${MINOR_VERSION}.${PATCH_VERSION})

# the name here doesn't really matter, but it's used further below
set(EP_NAME easyloggingpp)
ExternalProject_Add(${EP_NAME}
  # prefix for the external project in the build folder
  PREFIX external
  # the source
  URL https://github.com/easylogging/easyloggingpp/archive/v${MAJOR_VERSION}.${MINOR_VERSION}.zip
  # don't update, we lock the version
  UPDATE_COMMAND ""
  # Fix the required cmake version. To generate: diff -Naur old-src new-src > patchfile
  PATCH_COMMAND echo "Patching easylogging ..." && patch -p1 -tN < ${CMAKE_CURRENT_SOURCE_DIR}/patchfile && echo "done."
  # Install where catkin wants us to
  INSTALL_DIR ${CMAKE_DEVEL_PREFIX}/${PROJECT_NAME}
  # Pass the install dir on to cmake
  CMAKE_ARGS -DCMAKE_INSTALL_PREFIX:string=<INSTALL_DIR>
  # $(MAKE) uses the catkin jobserver (vs make)
  BUILD_COMMAND $(MAKE)
  # Don't install yet, wait for catkin to install
  INSTALL_COMMAND $(MAKE) install && rsync -r ../${EP_NAME}/cmake <INSTALL_DIR>
)

# grab the source and install dir from the external project
ExternalProject_Get_Property(${EP_NAME} SOURCE_DIR)
ExternalProject_Get_Property(${EP_NAME} INSTALL_DIR)

# we're not using catkin_package, so we have to manually set some paths
set(CATKIN_PACKAGE_SHARE_DESTINATION ${CATKIN_GLOBAL_SHARE_DESTINATION}/${PROJECT_NAME})
set(PACKAGE_INSTALL_PREFIX ${CMAKE_INSTALL_PREFIX}/${PROJECT_NAME})

# Helpful variables for PackageConfig creation
set(PC_NAME ${PROJECT_NAME}Config.cmake)
set(PC_VERSION_NAME ${PROJECT_NAME}ConfigVersion.cmake)
set(DEVEL_PC ${CMAKE_DEVEL_PREFIX}/${CATKIN_PACKAGE_SHARE_DESTINATION}/cmake/${PC_NAME})
set(DEVEL_PC_VERSION ${CMAKE_DEVEL_PREFIX}/${CATKIN_PACKAGE_SHARE_DESTINATION}/cmake/${PC_VERSION_NAME})
set(INSTALL_PC ${PROJECT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/${PC_NAME})

# generate PackageConfig.cmake that points to the FindModule.cmake
# one for devel
set(CONFIG_DIR ${INSTALL_DIR}) # used in Config.cmake.in
configure_file(Config.cmake.in ${DEVEL_PC} @ONLY)
# one for install
set(CONFIG_DIR ${PACKAGE_INSTALL_PREFIX}) # used in Config.cmake.in
configure_file(Config.cmake.in ${INSTALL_PC} @ONLY)

# generate the PackageConfigVersion.cmake since easyloggingpp doesn't come with one
configure_file(ConfigVersion.cmake.in ${DEVEL_PC_VERSION} @ONLY)

# install the package
install(DIRECTORY ${INSTALL_DIR}/ DESTINATION ${PACKAGE_INSTALL_PREFIX})
# install our PackageConfig.cmake that points to the install folder
install(FILES ${INSTALL_PC} ${DEVEL_PC}
        DESTINATION ${CMAKE_INSTALL_PREFIX}/cmake)
