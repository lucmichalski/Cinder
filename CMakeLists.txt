#
# Modified by Luc Michalski - 2016
#

cmake_minimum_required( VERSION 3.0 FATAL_ERROR )

string(COMPARE NOTEQUAL "$ENV{TRAVIS_TAG}" "" travis_deploy)
string(COMPARE EQUAL "$ENV{APPVEYOR_REPO_TAG}" "true" appveyor_deploy)

if(travis_deploy)
  set(version "$ENV{TRAVIS_TAG}")
elseif(appveyor_deploy)
  set(version "$ENV{APPVEYOR_REPO_TAG_NAME}")
else()
  set(version "v0.1.0")
endif()

string(REGEX REPLACE "^v" "" version "${version}")

### Hunter snapshot that will be used ###
include("cmake/HunterGate.cmake")
HunterGate(
    URL "https://github.com/ruslo/hunter/archive/v0.14.26.tar.gz"
    SHA1 "45ac03944db2bc7fc55dfceca42d89ec7fa1dca2"
)

set( CMAKE_VERBOSE_MAKEFILE ON )

project( cinder VERSION ${version})

set(is_android ${ANDROID}) # syntax compatibility
string(COMPARE EQUAL "${CMAKE_OSX_SYSROOT}" "iphoneos" is_ios)
string(COMPARE EQUAL "${CMAKE_SYSTEM_NAME}" "Linux" is_linux)
string(COMPARE EQUAL "${CMAKE_GENERATOR}" "Xcode" is_xcode)

message("Android: ${is_android}")
message("iOS    : ${is_ios}")
message("Linux  : ${is_linux}")
message("Xcode  : ${is_xcode}")

if(is_xcode)
  set(CMAKE_CONFIGURATION_TYPES "Debug;Release" CACHE STRING "Configuration types")
endif()

option( CINDER_BUILD_SAMPLES "Build all samples." OFF )
option( CINDER_VERBOSE "Build all samples." ON )
option( BUILD_SHARED_LIBS "Build all samples." OFF )

#set( CINDER_COMPILER_FLAGS "-std=c++14" )
#CLANG_CXX_LANGUAGE_STANDARD
set( CLANG_CXX_LANGUAGE_STANDARD "-std=c++11" )
set( CINDER_COMPILER_FLAGS "-std=c++11" )

set( CINDER_BUILD_SAMPLE "" CACHE STRING "Build a specific sample by specifying its path relative to the samples directory (ex. '_opengl/Cube')." )

set( CINDER_PATH      "${CMAKE_CURRENT_SOURCE_DIR}" )
set( CINDER_CMAKE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/proj/cmake" )

include( ${CINDER_CMAKE_DIR}/configure.cmake )
include( ${CINDER_CMAKE_DIR}/libcinder_configure.cmake )
include( ${CINDER_CMAKE_DIR}/libcinder_source_files.cmake )

if( CINDER_LINUX )
	include( "${CINDER_CMAKE_DIR}/platform_linux.cmake" )
elseif( CINDER_MAC )
	include( "${CINDER_CMAKE_DIR}/platform_macosx.cmake" )
elseif( CINDER_ANDROID )
	include( "${CINDER_CMAKE_DIR}/platform_android.cmake" )
elseif( CINDER_MSW )
	include( "${CINDER_CMAKE_DIR}/platform_msw.cmake" )
else()
	message( FATAL_ERROR "no target defined for system: '${CMAKE_SYSTEM_NAME}.'" )
endif()

include( ${CINDER_CMAKE_DIR}/libcinder_target.cmake )

if( CINDER_BUILD_SAMPLES )
	include( ${CINDER_CMAKE_DIR}/modules/findCMakeDirs.cmake )

	set( allSamples "" )
	findCMakeDirs( allSamples "${CINDER_PATH}/samples" )

	foreach( sampleDir ${allSamples} )
		if( CINDER_VERBOSE )
			message( STATUS "adding sample: ${sampleDir}" )
		endif()
		add_subdirectory( ${sampleDir} )
	endforeach()
elseif( CINDER_BUILD_SAMPLE )
	if( CINDER_VERBOSE )
		message( STATUS "adding sample: ${CINDER_BUILD_SAMPLE}" )
	endif()
	add_subdirectory( ${CINDER_PATH}/samples/${CINDER_BUILD_SAMPLE}/proj/cmake )
endif()
