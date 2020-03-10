cmake_minimum_required(VERSION 3.5)
project(serial)

find_package(ament_cmake REQUIRED)

ament_export_include_directories(include)
ament_export_libraries(${PROJECT_NAME})

# Sources Add serial library
add_library(${PROJECT_NAME} SHARED src/serial.cc)
target_include_directories(
  ${PROJECT_NAME}
  PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
         "$<INSTALL_INTERFACE:include>")

if(APPLE) # macOS
  find_library(IOKIT_LIBRARY IOKit)
  find_library(FOUNDATION_LIBRARY Foundation)
  target_sources(${PROJECT_NAME} PRIVATE src/impl/unix.cc
                                         src/impl/list_ports/list_ports_osx.cc)
  target_link_libraries(${PROJECT_NAME} ${FOUNDATION_LIBRARY} ${IOKIT_LIBRARY})
elseif(UNIX) # .*nix
  target_sources(
    ${PROJECT_NAME} PRIVATE src/impl/unix.cc
                            src/impl/list_ports/list_ports_linux.cc)
  target_link_libraries(${PROJECT_NAME} rt pthread)
elseif(WIN32) # Windows
  target_sources(${PROJECT_NAME} PRIVATE src/impl/win.cc
                                         src/impl/list_ports/list_ports_win.cc)
  target_link_libraries(${PROJECT_NAME} setupapi)
  ament_export_libraries(setupapi)
endif()

# Include headers
target_include_directories(${PROJECT_NAME} PRIVATE include)

# Uncomment for example add_executable(serial_example
# examples/serial_example.cc) add_dependencies(serial_example ${PROJECT_NAME})
# target_link_libraries(serial_example ${PROJECT_NAME})

# Install executable
install(
  TARGETS ${PROJECT_NAME}
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib)

install(
  TARGETS ${PROJECT_NAME}
  EXPORT export_${PROJECT_NAME}
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  INCLUDES
  DESTINATION include)

ament_export_include_directories(include)
ament_export_libraries(${PROJECT_NAME})
ament_export_interfaces(export_${PROJECT_NAME} HAS_LIBRARY_TARGET)

# Install headers
install(DIRECTORY include/ DESTINATION include)

# Tests if(BUILD_TESTING) add_subdirectory(tests) endif()

ament_package()
