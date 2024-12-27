

# Minimum required version of CMake
cmake_minimum_required(VERSION 3.29)

# Set Executable File Name
set(EXECUTABLE_NAME "Raylib Game")
set(INSTALLER_PACKAGE_NAME "Installer Package Name")

# Set the name of your game project folder and replace the default "Game" folder name accordingly.
set(GAME_DIR_NAME "Game")

### Project version managment ###
set(PROJECT_VERSION_MAJOR 1)
set(PROJECT_VERSION_MINOR 0)
set(PROJECT_VERSION_PATCH 1.0)

# Project version
set(PROJECT_VERSION "${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}")

# Project setup 
project(RaylibGame VERSION ${PROJECT_VERSION})


# Set C++ standard
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Release Build Settings - such as turning off console for MSVC compiler and more to be added as required.
# Check the readme for more.
if(CMAKE_BUILD_TYPE STREQUAL "Release")
    set(RELEASE_BUILD TRUE)
else()
    set(RELEASE_BUILD FALSE)
endif()

### External Libraries Setup ###

# .vessel is the folder where all fetched libraries will be located
# Notice .vessel is added to gitignore and you might not see it in your IDE.
# Look for it in the system explorer.
set(FETCHCONTENT_BASE_DIR "${CMAKE_SOURCE_DIR}/.vessel/libs")


# Fetch Github Libraries #

include(FetchContent)

FetchContent_Declare(
    raylib
    GIT_REPOSITORY https://github.com/raysan5/raylib.git
    GIT_TAG 5.5  # Replace this with the desired Raylib version
)

### Add libraries as needed here ###


# Make External Libraries Available #
FetchContent_MakeAvailable(raylib)


# Include directories for header files - place your headers in this path to separate them from source
include_directories("${PROJECT_SOURCE_DIR}/${GAME_DIR_NAME}/include")


# Add the executables
add_executable(${CMAKE_PROJECT_NAME}
"${PROJECT_SOURCE_DIR}/${GAME_DIR_NAME}/source/main.cpp" ${PROJECT_SOURCE_DIR}/${GAME_DIR_NAME}/resources/icon/icon.rc
 )

  # Set the executable name
 set_target_properties(${CMAKE_PROJECT_NAME} PROPERTIES OUTPUT_NAME ${EXECUTABLE_NAME})


### Link Exernal Libraries ###
target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE raylib)

### Set resource Folder, such as icons sounds, etc. ###

# Define the resources folder to be copied onto the compiled project folder
file(COPY "${PROJECT_SOURCE_DIR}/${GAME_DIR_NAME}/resources" DESTINATION "${CMAKE_BINARY_DIR}")


# Define a compile-time constant for the resources path - The resources path reference to be used for the compiled project
target_compile_definitions(${CMAKE_PROJECT_NAME} PUBLIC 
    $<$<CONFIG:Release>:RESOURCES_PATH="${CMAKE_BINARY_DIR}/resources">
    $<$<CONFIG:Debug>:RESOURCES_PATH="${PROJECT_SOURCE_DIR}/${GAME_DIR_NAME}/resources"> 
)

# Setup header directories for install and build
target_include_directories(${CMAKE_PROJECT_NAME}
INTERFACE
$<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/${GAME_DIR_NAME}/include>
$<INSTALL_INTERFACE:${CMAKE_INSTALL_PREFIX}/include
)

### Release Build Settings ###
if(RELEASE_BUILD)

    # Disable MSVC compiler debug console
    if(MSVC) 

	    target_compile_definitions("${CMAKE_PROJECT_NAME}" PUBLIC _CRT_SECURE_NO_WARNINGS)

	    set_target_properties("${CMAKE_PROJECT_NAME}" PROPERTIES LINK_FLAGS "/SUBSYSTEM:WINDOWS /ENTRY:mainCRTStartup")
    
    # Disable CLANG AND GCC compiler debug console
    elseif(CMAKE_CXX_COMPILER_ID STREQUAL "GNU" OR CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
        set_target_properties("${CMAKE_PROJECT_NAME}" PROPERTIES LINK_FLAGS "-mwindows")
    endif()

endif()


### INSTALLER INSTRUCTIONS ###
# Instructions to create the installer using cpack


# Set package details
set(CPACK_PACKAGE_NAME ${INSTALLER_PACKAGE_NAME})

# Package Generator
set(CPACK_GENERATOR "ZIP")
set(CPACK_SOURCE_GENERATOR "ZIP")

# Package Name
set(CPACK_PACKAGE_FILE_NAME
"${CPACK_PACKAGE_NAME}-${CMAKE_SYSTEM_NAME}-${CMAKE_BUILD_TYPE}")

# Package Directory
set(CPACK_PACKAGE_DIRECTORY
"${PROJECT_SOURCE_DIR}/packages")

# License
set(CPACK_RESOURCE_FILE_LICENSE "${PROJECT_SOURCE_DIR}/LICENSE")

# Include Dependencies / Resources - at the root installation folder "."

# Install the target project to generate exe and dependencies
install(TARGETS ${CMAKE_PROJECT_NAME} DESTINATION ".")

# Install include files
install(FILES "${PROJECT_SOURCE_DIR}/${GAME_DIR_NAME}/include/main.h" DESTINATION "./include")

# Install resources directory files
install(DIRECTORY "${PROJECT_SOURCE_DIR}/${GAME_DIR_NAME}/resources"
    DESTINATION "."
    PATTERN "icon.rc" EXCLUDE
    PATTERN "icon.aps" EXCLUDE
)

# Pack required system libraries - attempt improved stability
include(InstallRequiredSystemLibraries)

### Package Versions Managment ###

### Package Version - the version of the packaging / installer - different than project version ###
set(CPACK_PACKAGE_VERSION_MAJOR 1)
set(CPACK_PACKAGE_VERSION_MINOR 0)
set(CPACK_PACKAGE_VERSION_PATCH 0)

set(CPACK_PACKAGE_VERSION "${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}.${CPACK_PACKAGE_VERSION_PATCH}")


# Install Miscellaneous files
install(FILES "${PROJECT_SOURCE_DIR}/LICENSE" DESTINATION ".") 

# Install the README file
install(FILES "${PROJECT_SOURCE_DIR}/README.md" DESTINATION ".") 

# Include CPack at the END of the file
include(CPack)

