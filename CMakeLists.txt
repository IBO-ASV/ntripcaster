cmake_minimum_required(VERSION 3.8)
cmake_policy(SET CMP0069 NEW)
project(ntripcaster)
set(CMAKE_C_STANDARD 99)

find_package(ament_cmake REQUIRED)

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
	ev
)
ENDIF()

install(TARGETS ${PROJECT_NAME}
  RUNTIME DESTINATION lib/${PROJECT_NAME}
)
install(FILES ntripcaster.json
  DESTINATION share/${PROJECT_NAME}/config
)

ament_package()
