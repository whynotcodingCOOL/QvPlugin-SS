project(QvPlugin-SS)
cmake_minimum_required(VERSION 3.1.0)

set(CMAKE_CXX_STANDARD 14)

if(CMAKE_VERSION VERSION_LESS "3.7.0")
    set(CMAKE_INCLUDE_CURRENT_DIR ON)
endif()

set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTORCC ON)

include(CTest)
find_package(Qt5 COMPONENTS Core Widgets Gui Network REQUIRED)

set(CMAKE_POSITION_INDEPENDENT_CODE ON)
include(cmake/shadowsocks-uvw.cmake)
set(QVPLUGIN_INTERFACE_INCLUDE_DIR "interface/")
set(QVPLUGIN_HAS_HTTP_TO_SOCKS_HELPER ON)
include(interface/QvPluginInterface.cmake)
include_directories(${QVPLUGIN_INTERFACE_INCLUDE_DIR})

add_library(${PROJECT_NAME} MODULE
    SSPlugin.hpp
    SSPlugin.cpp
    common/CommonHelpers.cpp
    common/CommonHelpers.hpp
    core/Serializer.cpp
    core/Serializer.hpp
    core/EventHandler.cpp
    core/EventHandler.hpp
    core/kernel/SSRInstance.cpp
    core/kernel/SSInstance.hpp
    ui/SSROutboundEditor.cpp
    ui/SSROutboundEditor.hpp
    ui/SSROutboundEditor.ui
    resx.qrc
    ${QVPLUGIN_INTERFACE_HEADERS})

if(APPLE)
    add_custom_command(TARGET ${PROJECT_NAME}
        POST_BUILD
        COMMAND ${CMAKE_INSTALL_NAME_TOOL} -add_rpath "@executable_path/../Frameworks/" $<TARGET_FILE:${PROJECT_NAME}>)
endif()

target_link_libraries(${PROJECT_NAME}
    Qt5::Core
    Qt5::Gui
    Qt5::Widgets
    Qt5::Network
    ${SHADOWSOCKS_UVW_LIBRARY})

install(TARGETS ${PROJECT_NAME} DESTINATION share/qv2ray/plugins)
if(BUILD_TESTING)
    add_executable(testparsess test/TestParseSS.cpp test/catch.hpp
    common/CommonHelpers.cpp
    common/CommonHelpers.hpp
    core/Serializer.cpp
    core/Serializer.hpp
    ${QVPLUGIN_INTERFACE_HEADERS})
    target_link_libraries(testparsess 
    Qt5::Core
    Qt5::Gui
    Qt5::Widgets
    Qt5::Network)
    add_test(NAME testparsess COMMAND $<TARGET_FILE:testparsess>)
endif()
