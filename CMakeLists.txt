
PROJECT(manus)
CMAKE_MINIMUM_REQUIRED(VERSION 2.8 FATAL_ERROR)
GET_FILENAME_COMPONENT(PROJECT_ROOT . ABSOLUTE)
GET_FILENAME_COMPONENT(BUILD_ROOT ${CMAKE_CURRENT_BINARY_DIR} ABSOLUTE)
SET(CMAKE_MODULE_PATH ${PROJECT_ROOT}/cmake)
SET(EXECUTABLE_OUTPUT_PATH ${CMAKE_CURRENT_BINARY_DIR})
SET(LIBRARY_OUTPUT_PATH ${CMAKE_CURRENT_BINARY_DIR})

IF(WIN32)
	ADD_DEFINITIONS(-DPLATFORM_WINDOWS)
	FIND_PACKAGE(OpenCVWindows REQUIRED)
	SET(OpenCV_INCLUDE_DIR ${OPENCV_INCLUDE_DIR})
	SET(OpenCV_LIBS "${OPENCV_CORE_LIBRARY};${OPENCV_IMGPROC_LIBRARY};${OPENCV_VIDEO_LIBRARY};${OPENCV_HIGHGUI_LIBRARY}")
ELSEIF(APPLE)  
	ADD_DEFINITIONS(-DPLATFORM_OSX)
	FIND_PACKAGE(OpenCV REQUIRED core imgproc video highgui)
ELSEIF(UNIX)
	ADD_DEFINITIONS(-DPLATFORM_LINUX)
	FIND_PACKAGE(OpenCV REQUIRED core imgproc video highgui)
ELSE()
	MESSAGE(FATAL_ERROR "Unrecognized platform")
ENDIF()

SET(CMAKE_CXX_FLAGS "-std=c++0x")
#SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")

LINK_LIBRARIES(${OpenCV_LIBS})
INCLUDE_DIRECTORIES(AFTER ${CMAKE_CURRENT_BINARY_DIR} ${OpenCV_INCLUDE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/src)

ADD_DEFINITIONS(-DMONGOOSE_NO_AUTH -DMONGOOSE_NO_CGI -DMONGOOSE_NO_DAV -DMONGOOSE_NO_USER -DMONGOOSE_NO_WEBSOCKET)

IF ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
	SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pthread")
ENDIF ()

SET(SERVER_SOURCES 
    src/mongoose.c 
    src/threads.c
    src/json.cpp
    src/server.cpp
    src/manus.cpp
    src/filesystem.cpp
    src/arm.cpp
    src/simulation.cpp
)

IF(CMAKE_BUILD_TYPE MATCHES DEBUG)
    ADD_DEFINITIONS(-DSERVER_DOCUMENT_ROOT="${CMAKE_CURRENT_SOURCE_DIR}/web")
ELSE(CMAKE_BUILD_TYPE MATCHES DEBUG)
    INCLUDE(Embed REQUIRED)
    EMBED_GENERATE(PREFIX "/" DESTINATION "${CMAKE_CURRENT_BINARY_DIR}" FILES web/index.html web/jquery.js web/bootstrap.min.js web/manus.js web/manus.css web/bootstrap.min.css web/phoria.min.js web/glmatrix.min.js)
    SET(SERVER_SOURCES ${SERVER_SOURCES} ${CMAKE_CURRENT_BINARY_DIR}/embedded.c)
ENDIF(CMAKE_BUILD_TYPE MATCHES DEBUG)

ADD_EXECUTABLE(manus ${SERVER_SOURCES})
