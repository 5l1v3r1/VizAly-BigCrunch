# external project CMakeLists.txt template

# minimum cmake version requirement
cmake_minimum_required(VERSION 3.6)

# project name
project(blosc)

# enable external project cmake module
include(ExternalProject)

# download external project
ExternalProject_Add(
    ${PROJECT_NAME}-external
    #URL https://github.com/Blosc/c-blosc/archive/master.zip
	GIT_REPOSITORY https://github.com/Blosc/c-blosc.git
	GIT_TAG v1.14.4
    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-external
    # disable install step
    INSTALL_COMMAND ""
    # disable anoying constant checks during build
    UPDATE_DISCONNECTED TRUE
    # Enable fPIC for static->dynamic builds
    CMAKE_ARGS "-DCMAKE_POSITION_INDEPENDENT_CODE:BOOL=true"
)

# get external project source and binary directories
ExternalProject_Get_Property(${PROJECT_NAME}-external
    SOURCE_DIR
    BINARY_DIR
)

# create source directory with include path.
# this is necessary to configure interface include directories.
execute_process(
    COMMAND ${CMAKE_COMMAND} -E make_directory ${SOURCE_DIR}/blosc
)

# create imported target to be used as a dependency by programs
add_library(${PROJECT_NAME} STATIC IMPORTED GLOBAL)

# set project properties
if (UNIX)
set_target_properties(${PROJECT_NAME} PROPERTIES
		IMPORTED_LOCATION ${BINARY_DIR}/blosc/${CMAKE_CFG_INTDIR}/libblosc.a
)
else() #WIN32
set_target_properties(${PROJECT_NAME} PROPERTIES
		IMPORTED_LOCATION ${BINARY_DIR}/blosc/${CMAKE_CFG_INTDIR}/libblosc.lib
)
endif()

# add external project as dependency
add_dependencies(${PROJECT_NAME}
    ${PROJECT_NAME}-external
)
# target_link_libraries(${PROJECT_NAME}
#     INTERFACE ${PROJECT_NAME}-external
# )

# add external project include directories
# ISSUE: https://gitlab.kitware.com/cmake/cmake/issues/15689
# FIXED: https://gitlab.kitware.com/cmake/cmake/merge_requests/1264
# CMake 3.11+
# target_include_directories(${PROJECT_NAME}
#    INTERFACE ${SOURCE_DIR}/blosc
#)

# Should work with older CMakes
set_property(TARGET ${PROJECT_NAME} APPEND PROPERTY INTERFACE_INCLUDE_DIRECTORIES ${SOURCE_DIR}/blosc)

