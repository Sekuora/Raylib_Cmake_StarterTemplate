# Minimum required version of CMake
cmake_minimum_required(VERSION 3.29)

# Project setup
project(RaylibDefaultProject)

# Set C++ standard
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(FETCHCONTENT_BASE_DIR "${CMAKE_SOURCE_DIR}/.vessel/libs")

include(FetchContent)

### Fetch Github Libraries ###

FetchContent_Declare(
    raylib
    GIT_REPOSITORY https://github.com/raysan5/raylib.git
    GIT_TAG 5.5  # Replace this with the desired Raylib version
)

### Add libraries as needed here ###


### Make External Libraries Available ###
FetchContent_MakeAvailable(raylib)

# Set the name of your game and replace the default "Game" folder name accordingly.
set(GAME_NAME "Game")

# Include directories for header files - place your headers in this path to separate them from source.
include_directories("${PROJECT_SOURCE_DIR}/${GAME_NAME}/include")


# Add the executables
add_executable(${CMAKE_PROJECT_NAME}
"${PROJECT_SOURCE_DIR}/${GAME_NAME}/source/main.cpp"
)



### Link Exernal Libraries ###
target_link_libraries(RaylibDefaultProject PRIVATE raylib)



# Disable MSVC compiler debug
if(MSVC) 

	target_compile_definitions("${CMAKE_PROJECT_NAME}" PUBLIC _CRT_SECURE_NO_WARNINGS)

	set_target_properties("${CMAKE_PROJECT_NAME}" PROPERTIES LINK_FLAGS "/SUBSYSTEM:WINDOWS /ENTRY:mainCRTStartup")
	

endif()
