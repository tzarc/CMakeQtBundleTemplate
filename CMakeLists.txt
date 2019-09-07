cmake_minimum_required(VERSION 3.0.0 FATAL_ERROR)

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(project_name
	CMakeQtBundleTemplate)

project(${project_name} LANGUAGES CXX)

find_package(Qt5 COMPONENTS REQUIRED Core Gui Widgets)

get_target_property(_qmake_executable Qt5::qmake IMPORTED_LOCATION)
get_filename_component(_qt_bin_dir "${_qmake_executable}" DIRECTORY)
find_program(DEPLOYQT_EXECUTABLE NAMES windeployqt macdeployqt HINTS "${_qt_bin_dir}")

set(project_sources
	mainwindow.cpp
	main.cpp)

set(project_headers
	mainwindow.h)

set(project_ui
	mainwindow.ui)

qt5_wrap_ui(ui_wrap ${project_ui})
qt5_wrap_cpp(moc_sources ${project_headers})

if(APPLE)
    add_executable(${project_name} MACOSX_BUNDLE ${project_sources} ${project_headers} ${ui_wrap} ${moc_sources})
    configure_file(${CMAKE_SOURCE_DIR}/Info.plist.in ${CMAKE_BINARY_DIR}/Info.plist)
    set_target_properties(${project_name} PROPERTIES MACOSX_BUNDLE_INFO_PLIST ${CMAKE_BINARY_DIR}/Info.plist)
    add_custom_command(TARGET ${project_name}
        POST_BUILD
        COMMAND ${DEPLOYQT_EXECUTABLE} "$<TARGET_BUNDLE_DIR:${project_name}>" -qmldir=${CMAKE_SOURCE_DIR}
    )
elseif(WIN32)
    add_executable(${project_name} WIN32 ${project_sources} ${project_headers} ${ui_wrap} ${moc_sources})
    add_custom_command(TARGET ${project_name}
        POST_BUILD
        COMMAND ${DEPLOYQT_EXECUTABLE} "$<TARGET_FILE:${project_name}>" --$<LOWER_CASE:$<CONFIG>> --qmldir ${CMAKE_SOURCE_DIR}
    )
endif()

target_link_libraries(${project_name} Qt5::Core Qt5::Gui Qt5::Widgets)
