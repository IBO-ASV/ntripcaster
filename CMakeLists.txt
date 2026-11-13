cmake_minimum_required(VERSION 3.8)
cmake_policy(SET CMP0069 NEW)
project(ntripcaster)
set(CMAKE_C_STANDARD 99)

find_package(ament_cmake REQUIRED)

foreach(dependency wsocket ulog)
  if(NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/${dependency}/CMakeLists.txt")
    message(FATAL_ERROR "Missing ${dependency}: run git submodule update --init --recursive src/driver/gnss/ntripcaster from the workspace root")
  endif()
endforeach()

if(NOT WIN32)
  find_path(LIBEV_INCLUDE_DIR ev.h)
  find_library(LIBEV_LIBRARY ev)
  if(LIBEV_INCLUDE_DIR AND LIBEV_LIBRARY)
    add_library(ntrip_libev INTERFACE)
    target_include_directories(ntrip_libev INTERFACE ${LIBEV_INCLUDE_DIR})
    target_link_libraries(ntrip_libev INTERFACE ${LIBEV_LIBRARY})
  elseif(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/libev/ev.c")
    # Keep the pinned, bundled dependency usable on hosts without libev-dev.
    add_library(ntrip_libev STATIC libev/ev.c)
    target_compile_definitions(ntrip_libev PRIVATE EV_STANDALONE=1)
    target_include_directories(ntrip_libev PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/libev)
    target_link_libraries(ntrip_libev PUBLIC m)
  else()
    message(FATAL_ERROR "Install libev-dev or initialize the libev submodule recursively")
  endif()
endif()

add_subdirectory(wsocket)
add_subdirectory(ulog)
add_subdirectory(cJSON)

IF(WIN32)
add_executable(${PROJECT_NAME} ntripcaster.c evwrap.c)
target_link_libraries(${PROJECT_NAME}
	wsocket
	ulog
	cJSON
)
ELSE()
add_executable(${PROJECT_NAME} ntripcaster.c)
target_link_libraries(${PROJECT_NAME}
	wsocket
	ulog
	cJSON
	ntrip_libev
)
ENDIF()

install(TARGETS ${PROJECT_NAME}
  RUNTIME DESTINATION lib/${PROJECT_NAME}
)
install(FILES config/ntripcaster.json
  DESTINATION share/${PROJECT_NAME}/config
)

ament_package()
