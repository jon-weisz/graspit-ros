cmake_minimum_required(VERSION 2.8.3)
project(graspit)

set(PACKAGE_DEPS
  household_objects_database
)

find_package(catkin REQUIRED COMPONENTS ${PACKAGE_DEPS})

catkin_package(
  CATKIN_DEPENDS ${PACKAGE_DEPS}
  INCLUDE_DIRS . graspit_source graspit_source/include
)

include_directories(${catkin_INCLUDE_DIRS})

set(GRASPIT_REVISION 61fec) #The version of GraspIt to check out after cloning the repository.

if(NOT EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/graspit_source)
    add_custom_command(
        OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/graspit_source/graspit.pro
        COMMAND git clone https://github.com/roamlab/graspit.git graspit_source
        COMMAND git --git-dir=${CMAKE_CURRENT_SOURCE_DIR}/graspit_source/.git --work-tree=${CMAKE_CURRENT_SOURCE_DIR}/graspit_source/.git checkout ${GRASPIT_REVISION} #Run git command in source directory to checkout tested revision
        COMMAND patch -N -d graspit_source -p0 < graspit_project.patch
        COMMAND patch -N -d graspit_source -p0 < graspit_dbase.patch
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
    )
endif(NOT EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/graspit_source)



add_custom_target(graspit_build ALL
  DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/graspit_source/graspit.pro
  COMMAND qmake "EXT_DESTDIR = ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_BIN_DESTINATION}" ${CMAKE_CURRENT_SOURCE_DIR}/graspit_source/graspit.pro
  COMMAND make
)

add_dependencies(graspit_build household_objects_database)

install(DIRECTORY graspit_source/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  PATTERN ".svn" EXCLUDE
)
