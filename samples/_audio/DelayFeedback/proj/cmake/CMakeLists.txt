cmake_minimum_required( VERSION 3.0 FATAL_ERROR )
set( CMAKE_VERBOSE_MAKEFILE ON )

project( audio-DelayFeedback )

get_filename_component( CINDER_PATH "${CMAKE_CURRENT_SOURCE_DIR}/../../../../.." ABSOLUTE )
get_filename_component( SAMPLE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../../" ABSOLUTE )

include( "${CINDER_PATH}/proj/cmake/modules/cinderMakeApp.cmake" )

set( RESOURCE_FILES RESOURCES
	${SAMPLE_DIR}/resources/smoothCircle.vert
	${SAMPLE_DIR}/resources/smoothCircle.frag
)

ci_make_app(
	SOURCES     ${SAMPLE_DIR}/src/DelayFeedbackApp.cpp
	CINDER_PATH ${CINDER_PATH}
	INCLUDES    ${SAMPLE_DIR}/include
	RESOURCES   ${RESOURCE_FILES}
)
