# CMakeList.txt: GameEngine 的 CMake 项目，在此处包括源代码并定义
# 项目特定的逻辑。
#
cmake_minimum_required (VERSION 3.8)

set(CMAKE_CXX_STANDARD 20)

# 在游戏发布时将此设置为ON而不是OFF
# 设置是否为生产构建
# Make Sure PRODUCTION_BUILD WHEN SEND TO INTERNET
set(PRODUCTION_BUILD OFF CACHE BOOL "Make this a production build" FORCE)

if (MSVC)
	set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Release>:Release>")
	set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")
	add_compile_options(/arch:AVX2)
else()
	add_compile_options(-mavx2 -mfma)
endif()

project (GameEngine)

add_subdirectory(thirdparty/glad)
add_subdirectory(thirdparty/glfw)
add_subdirectory(thirdparty/glm)


# DON'T ADD THE SOURCES BY HAND, they are already added with this macro
file(GLOB_RECURSE MY_SOURCES CONFIGURE_DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp")

# 将源代码添加到此项目的可执行文件。
add_executable ("${CMAKE_PROJECT_NAME}")

set_property(TARGET "${CMAKE_PROJECT_NAME}" PROPERTY CXX_STANDARD 20)


if(PRODUCTION_BUILD)

	# 设置生产环境变量, 处于生产环境时为1
	# PRODUCTION_BUILD value is 1 WHEN IN PRODUCTION_BUILD
	target_compile_definitions("${CMAKE_PROJECT_NAME}" PUBLIC PRODUCTION_BUILD=1)

else()
	target_compile_definitions("${CMAKE_PROJECT_NAME}" PUBLIC PRODUCTION_BUILD=0)

endif()

target_sources("${CMAKE_PROJECT_NAME}" PRIVATE ${MY_SOURCES})

if(MSVC)
	
	target_compile_definitions("${CMAKE_PROJECT_NAME}" PUBLIC _CRT_SECURE_NO_WARNINGS)

	if(PRODUCTION_BUILD)
		set_target_properties("${CMAKE_PROJECT_NAME}"
		PROPERTIES LINK_FLAGS "/SUBSYSTEM:WINDOWS /ENTRY:mainCRTStartup")		# Remove black console
	endif()

elseif(MINGW)	# Windows / MinGW GCC / Clang
	if(PRODUCTION_BUILD)
		target_link_options("${CMAKE_PROJECT_NAME}" PRIVATE
		-mwindows)
	endif()

elseif(APPLE OR UNIX)	# MacOS / Linux
	if(PRODUCTION_BUILD)
		target_link_options("${CMAKE_PROJECT_NAME}" PRIVATE
		-s)
	endif()

endif()

target_include_directories("${CMAKE_PROJECT_NAME}" PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/src")

target_link_libraries("${CMAKE_PROJECT_NAME}" PRIVATE glad glfw glm)
