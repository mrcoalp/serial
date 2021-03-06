cmake_minimum_required(VERSION 3.10)

project(serial)

if (APPLE)
    find_library(IOKIT_LIBRARY IOKit)
    find_library(FOUNDATION_LIBRARY Foundation)
endif ()

## Sources
set(serial_SRCS src/serial.cc)

if (APPLE)
    # If OSX
    list(APPEND serial_SRCS src/impl/unix.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_osx.cc)
elseif (UNIX)
    # If unix
    list(APPEND serial_SRCS src/impl/unix.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_linux.cc)
else ()
    # If windows
    list(APPEND serial_SRCS src/impl/win.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_win.cc)
endif ()

## Add serial library
add_library(${PROJECT_NAME} ${serial_SRCS})

if (APPLE)
    target_link_libraries(${PROJECT_NAME} ${FOUNDATION_LIBRARY} ${IOKIT_LIBRARY})
elseif (UNIX)
    target_link_libraries(${PROJECT_NAME} rt pthread)
else ()
    target_link_libraries(${PROJECT_NAME} setupapi)
endif ()

## Uncomment for example
#add_executable(serial_example examples/serial_example.cc)
#add_dependencies(serial_example ${PROJECT_NAME})
#target_link_libraries(serial_example ${PROJECT_NAME})

## Include headers
target_include_directories(serial PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>)

## Install executable
install(TARGETS ${PROJECT_NAME}
        EXPORT ${PROJECT_NAME}-targets
        PUBLIC_HEADER DESTINATION include
        ARCHIVE DESTINATION lib
        LIBRARY DESTINATION lib
        RUNTIME DESTINATION bin)

install(EXPORT ${PROJECT_NAME}-targets
        NAMESPACE ${PROJECT_NAME}::
        FILE ${PROJECT_NAME}-config.cmake
        DESTINATION lib/cmake/${PROJECT_NAME})

## Install headers
#install(FILES include/serial/serial.h include/serial/v8stdint.h
#  DESTINATION ${CATKIN_GLOBAL_INCLUDE_DESTINATION}/serial)
