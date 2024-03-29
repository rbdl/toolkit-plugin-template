PROJECT({{ plugin_name }}Plugin)
{% if not intree -%}
CMAKE_MINIMUM_REQUIRED(VERSION 3.10)

{%- endif %}
	
SET(Plugin_SRCS
	{% if add_extension -%}
	{{ plugin_name }}ModelExtension.cc
	{%- endif %}
	{{ plugin_name }}Plugin.cc
)

SET({{ plugin_name }}Plugin_MOC_HDRS
	include/toolkit_interfaces.h
	{{ plugin_name }}Plugin.h
)

SET(CMAKE_AUTORCC ON )
SET(CMAKE_AUTOUIC ON )
SET(CMAKE_AUTOMOC ON )

{% if not intree -%}
SET(TOOLKIT_SOURCE_PATH "" CACHE PATH "Path to Toolkit Source Code")
SET(TOOLKIT_BUILD_PATH "" CACHE PATH "Path to Toolkit Build Dir")

if(TOOLKIT_BUILD_PATH)
else()
  MESSAGE( FATAL_ERROR "You need to supply the path to the build dir of the toolkit, since you will require certain headers generated during the build process!")
endif()


if(TOOLKIT_SOURCE_PATH)
else()
	MESSAGE( FATAL_ERROR "You need to supply the path to the toolkit source code")
endif()

LIST( APPEND CMAKE_MODULE_PATH ${TOOLKIT_SOURCE_PATH}/cmake )

SET(CUSTOM_QT_PATH "" CACHE PATH "Path to Qt Installation")
IF (CUSTOM_QT_PATH)
	SET(CMAKE_CXX_STANDARD_REQUIRED ON)
	SET(CMAKE_SKIP_BUILD_RPATH FALSE)
	SET(CMAKE_BUILD_WITH_INSTALL_RPATH FALSE)
	SET(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")
	SET(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
	SET(CMAKE_PREFIX_PATH "${CUSTOM_QT_PATH}/lib/cmake")
ENDIF(CUSTOM_QT_PATH)

FIND_PACKAGE (Qt5 REQUIRED COMPONENTS Core Gui Widgets 3DCore 3DExtras 3DRender 3DInput)
FIND_PACKAGE (RBDL COMPONENTS LUAMODEL REQUIRED)
FIND_PACKAGE (Eigen3 REQUIRED)

SET( PLUGIN_INSTALL_DIR share/ORB/rbdl-toolkit/plugins )
{%- endif %}
	
{% if not intree -%}
INCLUDE_DIRECTORIES( ${TOOLKIT_BUILD_PATH} )
INCLUDE_DIRECTORIES( ${TOOLKIT_BUILD_PATH}/include )
INCLUDE_DIRECTORIES( ${TOOLKIT_SOURCE_PATH} )
INCLUDE_DIRECTORIES (${TOOLKIT_SOURCE_PATH}/include)
{%- else -%}
INCLUDE_DIRECTORIES( ${CMAKE_SOURCE_DIR} )
INCLUDE_DIRECTORIES (${CMAKE_SOURCE_DIR}/include)
{%- endif %}
INCLUDE_DIRECTORIES (${QT_INCLUDE_DIR})
INCLUDE_DIRECTORIES (${CMAKE_CURRENT_SOURCE_DIR})
INCLUDE_DIRECTORIES (${CMAKE_CURRENT_SOURCE_DIR}/include)
INCLUDE_DIRECTORIES (${RBDL_INCLUDE_DIR})
INCLUDE_DIRECTORIES (${EIGEN3_INCLUDE_DIR})

{% if not intree -%}
FIND_LIBRARY(TOOLKIT_LIB NAMES toolkitlib PATHS ${TOOLKIT_BUILD_PATH})
{%- endif %}

ADD_DEFINITIONS (${QT_DEFINITIONS})
ADD_DEFINITIONS (-DQT_PLUGIN)
ADD_DEFINITIONS (-DQT_SHARED)

ADD_LIBRARY ({{ plugin_name | lower }}plugin SHARED ${Plugin_SRCS})
TARGET_COMPILE_DEFINITIONS ({{ plugin_name | lower }}plugin PRIVATE TOOLKIT_COMPILE_PLUGIN_{{ plugin_name | upper}})

{% if not intree -%}
TARGET_LINK_LIBRARIES ({{ plugin_name | lower }}plugin 
	${QT_LIBRARIES} 
	${QT_QTXML_LIBRARY} 
	Qt5::Core
	Qt5::Gui
	Qt5::Widgets
	Qt5::3DCore
	Qt5::3DExtras
	Qt5::3DRender
	Qt5::3DInput
	${RBDL_LIBRARY}
	${TOOLKIT_LIB}
)
{%- else -%}
TARGET_LINK_LIBRARIES ({{ plugin_name | lower }}plugin
	${QT_LIBRARIES}
	${QT_QTXML_LIBRARY}
	Qt5::Core
	Qt5::Gui
	Qt5::Widgets
	Qt5::3DCore
	Qt5::3DExtras
	Qt5::3DRender
	Qt5::3DInput
	${RBDL_LIBRARY}
	toolkitlib
)
{%- endif %}

INSTALL( TARGETS {{ plugin_name | lower }}plugin LIBRARY DESTINATION ${PLUGIN_INSTALL_DIR})	
