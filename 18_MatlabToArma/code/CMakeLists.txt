CMAKE_MINIMUM_REQUIRED(VERSION 2.8)

SET( PROJECT_NAME MatlabToArma )

# Set project name 
PROJECT( ${PROJECT_NAME} )
 
#SET(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${PROJECT_SOURCE_DIR}/cmake_modules/")

#Find libraries
FIND_PACKAGE( ITK REQUIRED )
INCLUDE( ${ITK_USE_FILE} )

SET( REQUIRED_LIBS )
SET( BLAS_LIBRARIES "" )
SET( LAPACK_LIBRARIES "")

# for Windows 64 bit machines use precompiled blas and lapack libs
IF(WIN32 AND ((CMAKE_SYSTEM_PROCESSOR MATCHES "x86_64") OR (CMAKE_SYSTEM_PROCESSOR MATCHES "AMD64")) )
  SET( BLAS_LIBRARIES "${PROJECT_SOURCE_DIR}/win64_blas/blas_win64_MT.lib" )
  SET( LAPACK_LIBRARIES "${PROJECT_SOURCE_DIR}/win64_blas/lapack_win64_MT.lib")
ELSE()
  # for all other machines, find appropriate libs
  FIND_PACKAGE( LAPACK REQUIRED )  
  FIND_PACKAGE( BLAS REQUIRED )
  
  # gcc needs linker flags for blas and lapack
  SET( CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${BLAS_LINKER_FLAGS} ${LAPACK_LINKER_FLAGS}" )
ENDIF()

SET( REQUIRED_LIBS ${REQUIRED_LIBS} ${BLAS_LIBRARIES} ${LAPACK_LIBRARIES} )

FIND_PACKAGE( Armadillo REQUIRED )
INCLUDE_DIRECTORIES(${ARMADILLO_INCLUDE_DIRS})
#SET( REQUIRED_LIBS ${REQUIRED_LIBS} ${ARMADILLO_LIBRARIES} )

FIND_PACKAGE(OpenMP REQUIRED)
SET( CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}" )
SET( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}" )

# add c++11 flag if it is GCC v<5  
IF(CMAKE_COMPILER_IS_GNUCC OR CMAKE_COMPILER_IS_GNUCXX)  
  IF(CMAKE_CXX_COMPILER_VERSION VERSION_LESS 5.0)  
    INCLUDE( CheckCXXCompilerFlag )  
    CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)  
    CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)  
    IF( COMPILER_SUPPORTS_CXX11 )  
    	SET( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")  
    elseif(COMPILER_SUPPORTS_CXX0X )  
    	SET( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")  
    else()  
    	MESSAGE(ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")  
    endif()  
  ENDIF()  
ENDIF()   

INCLUDE_DIRECTORIES(${PROJECT_SOURCE_DIR}/src)

# remove some MSVC flags
IF(MSVC)
	ADD_DEFINITIONS(-D_CRT_SECURE_NO_WARNINGS -D_CRT_NONSTDC_NO_DEPRECATE -D_SCL_SECURE_NO_WARNINGS)
ENDIF(MSVC)

# enable project directories
SET_PROPERTY( GLOBAL PROPERTY USE_FOLDERS ON )

FILE( GLOB_RECURSE AlgorithmHeaders "${PROJECT_SOURCE_DIR}/src/*.h" )
FILE( GLOB_RECURSE AlgorithmSources "${PROJECT_SOURCE_DIR}/src/*.cpp" )

# Add sources to executable
ADD_EXECUTABLE(
  ${PROJECT_NAME}
  ${PROJECT_SOURCE_DIR}/src/main.cxx
  ${AlgorithmHeaders}
  ${AlgorithmSources}
)

# Link the libraries to be used
TARGET_LINK_LIBRARIES(
  ${PROJECT_NAME}
  ${ITK_LIBRARIES}
  ${REQUIRED_LIBS}
  ${ARMADILLO_LIBRARIES}
)
