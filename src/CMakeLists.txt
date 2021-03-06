cmake_minimum_required(VERSION 2.8.8)

find_package(Qt5Core)

set(CMAKE_AUTOMOC ON)

## FindQt4 does not add the include directory for OpenGL
find_package(OpenGL REQUIRED)
include_directories(${OPENGL_INCLUDE_DIR})

include_directories( ../inst/local/include/smoke
                     ${R_INCLUDE_DIR} ${QTBASE_DIR}/local/include
                     ${QTBASE_DIR}/include ${CMAKE_CURRENT_SOURCE_DIR}
                     ${CMAKE_CURRENT_BINARY_DIR} )

set(qtpaint_LIB_SRCS Layer.cpp Painter.cpp QtPainter.cpp imports.cpp
    OpenGLPainter.cpp RLayer.cpp init.cpp PainterWrappers.cpp
    ScenePainter.cpp paintUtils.cpp PlotView.cpp
    scene.cpp QtBasePainter.cpp convert.cpp TestWidget.cpp
    ${CMAKE_CURRENT_BINARY_DIR}/smokedata.cpp
    ${CMAKE_CURRENT_BINARY_DIR}/x_1.cpp)

if(WIN32) # Toughest Win32 part: generating the defs file for the DLL
foreach(qtpaint_lib_src ${qtpaint_LIB_SRCS})
  get_filename_component(qtpaint_lib_src_basename ${qtpaint_lib_src} NAME)
  list(APPEND qtpaint_LIB_OBJS "${CMAKE_CURRENT_BINARY_DIR}/CMakeFiles/qtpaint.dir/${qtpaint_lib_src_basename}${CMAKE_CXX_OUTPUT_EXTENSION}")
endforeach(qtpaint_lib_src ${qtpaint_LIB_SRCS})
add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/qtpaint.def 
                   COMMAND sh ARGS ${CMAKE_CURRENT_SOURCE_DIR}/mkdef.sh
                   DEPENDS ${qtpaint_LIB_OBJS}) 
SET(CMAKE_LINK_DEF_FILE_FLAG "qtpaint.def ") # space needs to be there
add_library(qtpaint SHARED qtpaint.def ${qtpaint_LIB_SRCS} ${qtpaint_MOC_FILES})
else(WIN32)
add_library(qtpaint SHARED ${qtpaint_LIB_SRCS} ${qtpaint_MOC_FILES})
endif(WIN32)

qt5_use_modules(qtpaint Widgets OpenGL)

target_link_libraries(qtpaint ${QT_LIBRARIES} ${R_LIBRARIES} ${OPENGL_LIBRARIES})

set_target_properties(qtpaint PROPERTIES PREFIX "")
if(NOT WIN32) # Need to force .so on Mac, but Windows needs to be .dll
set_target_properties(qtpaint PROPERTIES SUFFIX ".so")
install(TARGETS qtpaint LIBRARY DESTINATION . )
else(NOT WIN32)
install(TARGETS qtpaint RUNTIME DESTINATION . )
endif(NOT WIN32)

INCLUDE(CMakeForceCompiler)
CMAKE_FORCE_CXX_COMPILER(${R_CXX} R_CXX)
