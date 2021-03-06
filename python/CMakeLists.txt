# Python cmake build script
# This requires swig, python sources and numpy to be installed
# See README.txt for documentaton on how to circumvent potential problems in detecting the libraries


find_package(SWIG REQUIRED)


#default: enable python bindings
option(BUILD_PYTHON_PACKAGE "Enable Python bindings" ON)

#1. find swig
INCLUDE(${SWIG_USE_FILE})

#find python
if(NOT PYTHON_EXECUTABLE)
       FIND_PACKAGE(PythonInterpWhich)
endif()

#we got everything?
if ( (NOT EXISTS ${PYTHON_EXECUTABLE} ) )
   message("Did not find python")
   set(BUILD_PYTHON_PACKAGE OFF CACHE BOOL "Enable Python bindings" FORCE)
endif()

#find python libs and includes
if (NOT PYTHON_LIBRARIES OR NOT PYTHON_INCLUDE_DIRS)
   FIND_PACKAGE(PythonLibsDist)
  #FIND_PACKAGE(PythonLibs)
endif()

#find dist-packages
if (NOT PYTHON_INSTDIR)
   FIND_PACKAGE(PythonInst)
endif()

#find numpy
if (NOT NUMPY_INCLUDE_DIR)
   FIND_PACKAGE(Numpy)
endif()


if ( (NOT EXISTS ${PYTHON_EXECUTABLE}))
   message("Did not find python executable, disabling python interface.")
   message("Try specifying executable manually in CMakeLists.txt")
   set(BUILD_PYTHON_PACKAGE OFF CACHE BOOL "Enable Python bindings" FORCE)
endif()

if ( (NOT EXISTS ${PYTHON_INCLUDE_DIRS}))
   message("Did not find python include dirs")
   message(${PYTHON_INCLUDE_DIRS})
   set(BUILD_PYTHON_PACKAGE OFF CACHE BOOL "Enable Python bindings" FORCE)
endif()


if ( (NOT EXISTS ${NUMPY_INCLUDE_DIR}) )
   message("Did not find numpy, disabling python interface")
   message(${NUMPY_INCLUDE_DIR})
   set(BUILD_PYTHON_PACKAGE OFF CACHE BOOL "Enable Python bindings" FORCE)
endif()


if ( NOT EXISTS ${PYTHON_INSTDIR})
   message("Did not find python install directory, disabling python interface")
   set(BUILD_PYTHON_PACKAGE OFF CACHE BOOL "Enable Python bindings" FORCE)
endif()



if(BUILD_PYTHON_PACKAGE)



#include_directories(${PYTHON_INCLUDE_DIR})
#include_directories(${NUMPY_INCLUDE_DIR})

include_directories(${PYTHON_INCLUDE_DIRS})
include_directories(${NUMPY_INCLUDE_DIR})


include_directories(./../External)
include_directories(./../External/Eigen)
include_directories(${PEER_SRC_DIR}/External/alglib)
include_directories(${PEER_SRC_DIR}/include)

set(CMAKE_SWIG_FLAGS "")
#SET(SWIGLINKLIBS ${PYTHON_LIBRARIES})

set_source_files_properties(peer.i PROPERTIES CPLUSPLUS ON)
set_source_files_properties(peer.i PROPERTIES SWIG_FLAGS "-includeall")
set_source_files_properties(peer.i PROPERTIES SWIG_FLAGS "-cpperraswarn")
swig_add_module(peer python peer.i)
swig_link_libraries(peer ${SWIGLINKLIBS} peerlib alglib)


IF(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")	 
  # Mac OS X specific code
  # Without this flag it does not find some of the libraries
  SET(CMAKE_MODULE_LINKER_FLAGS "-undefined dynamic_lookup")
ENDIF(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")



#Install python .so and python wrapper
INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/_peer.so DESTINATION ${PYTHON_INSTDIR})
INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/peer.py DESTINATION ${PYTHON_INSTDIR})
#install demo into the build directory

endif(BUILD_PYTHON_PACKAGE)
