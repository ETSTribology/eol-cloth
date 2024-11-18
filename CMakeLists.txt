# Author: Shinjiro Sueda
# Edited: Nick Weidner
cmake_minimum_required(VERSION 3.10)

# Project Name
project(eol-cloth)

# Specify C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)

#-----------------------------------------------
# USER SETTINGS
#-----------------------------------------------

# Compiler Options
option(ONLINE "Build with online rendering supported" ON)
option(MOSEK "Build with Mosek support" OFF)
option(GUROBI "Build with Gurobi support" OFF)
option(IGL "Build with libIGL support" OFF)

# Enable CMake's package registry
set(CMAKE_PREFIX_PATH "${CMAKE_PREFIX_PATH};${CMAKE_CURRENT_SOURCE_DIR}/vcpkg")

#-----------------------------------------------
# FIND PACKAGES
#-----------------------------------------------

# Eigen (header-only)
find_package(Eigen3 CONFIG REQUIRED)
if (NOT Eigen3_FOUND)
    message(FATAL_ERROR "Eigen3 not found!")
endif()

# GLM (header-only)
find_package(glm CONFIG REQUIRED)
if (NOT glm_FOUND)
    message(FATAL_ERROR "GLM not found!")
endif()

# GLFW
find_package(glfw3 CONFIG REQUIRED)
if (NOT glfw3_FOUND)
    message(FATAL_ERROR "GLFW3 not found!")
endif()

# GLEW
find_package(GLEW REQUIRED)
if (NOT GLEW_FOUND)
    message(FATAL_ERROR "GLEW not found!")
endif()

# libIGL (if enabled)
if(IGL)
    find_package(igl CONFIG REQUIRED)
    if(NOT igl_FOUND)
        message(FATAL_ERROR "libIGL not found!")
    endif()
endif()

# Mosek (manual handling)
if(MOSEK)
    # Assuming MOSEK is installed and environment variables are set
    find_package(Mosek REQUIRED)
    if(NOT Mosek_FOUND)
        message(FATAL_ERROR "Mosek not found!")
    endif()
endif()

# Gurobi (manual handling)
if(GUROBI)
    # Assuming GUROBI is installed and environment variables are set
    find_package(Gurobi REQUIRED)
    if(NOT Gurobi_FOUND)
        message(FATAL_ERROR "Gurobi not found!")
    endif()
endif()

#-----------------------------------------------
# FILE GLOB
#-----------------------------------------------

# Gather source files
file(GLOB_RECURSE SOURCES "src/*.cpp" "src/*.c")
file(GLOB_RECURSE HEADERS "src/*.hpp" "src/*.h")
file(GLOB_RECURSE GLSL "resources/*.glsl")

# Optional sources
if(ONLINE)
    file(GLOB_RECURSE ONLINE_SRC "src/online/*.{cpp,c}")
    file(GLOB_RECURSE ONLINE_HEADERS "src/online/*.{hpp,h}")
    list(APPEND SOURCES ${ONLINE_SRC})
    list(APPEND HEADERS ${ONLINE_HEADERS})
endif()

if(MOSEK)
    file(GLOB_RECURSE MOSEK_SRC "src/external/SolverWrappers/Mosek/*.{cpp,c}")
    file(GLOB_RECURSE MOSEK_HEADERS "src/external/SolverWrappers/Mosek/*.{hpp,h}")
    list(APPEND SOURCES ${MOSEK_SRC})
    list(APPEND HEADERS ${MOSEK_HEADERS})
endif()

if(GUROBI)
    file(GLOB_RECURSE GUROBI_SRC "src/external/SolverWrappers/Gurobi/*.{cpp,c}")
    file(GLOB_RECURSE GUROBI_HEADERS "src/external/SolverWrappers/Gurobi/*.{hpp,h}")
    list(APPEND SOURCES ${GUROBI_SRC})
    list(APPEND HEADERS ${GUROBI_HEADERS})
endif()

#-----------------------------------------------
# ADD EXECUTABLE
#-----------------------------------------------

add_executable(${PROJECT_NAME} ${SOURCES} ${HEADERS} ${GLSL})

#-----------------------------------------------
# INCLUDE DIRECTORIES
#-----------------------------------------------

target_include_directories(${PROJECT_NAME} PRIVATE 
    ${EIGEN3_INCLUDE_DIR}
    ${GLM_INCLUDE_DIRS}
    ${GLFW_INCLUDE_DIRS}
    ${GLEW_INCLUDE_DIRS}
)

if(IGL)
    target_include_directories(${PROJECT_NAME} PRIVATE ${IGL_INCLUDE_DIRS})
endif()

if(MOSEK)
    target_include_directories(${PROJECT_NAME} PRIVATE ${MOSEK_INCLUDE_DIRS})
endif()

if(GUROBI)
    target_include_directories(${PROJECT_NAME} PRIVATE ${GUROBI_INCLUDE_DIRS})
endif()

#-----------------------------------------------
# LINK LIBRARIES
#-----------------------------------------------

target_link_libraries(${PROJECT_NAME} PRIVATE 
    Eigen3::Eigen
    glm::glm
    glfw
    GLEW::GLEW
)

if(IGL)
    target_link_libraries(${PROJECT_NAME} PRIVATE igl::core igl::opengl)
endif()

if(ONLINE)
    if(WIN32)
        target_link_libraries(${PROJECT_NAME} PRIVATE opengl32)
    elseif(APPLE)
        target_link_libraries(${PROJECT_NAME} PRIVATE "-framework OpenGL -framework Cocoa -framework IOKit -framework CoreVideo")
    else()
        target_link_libraries(${PROJECT_NAME} PRIVATE GL)
    endif()
endif()

if(MOSEK)
    target_link_libraries(${PROJECT_NAME} PRIVATE Mosek::Mosek)
endif()

if(GUROBI)
    target_link_libraries(${PROJECT_NAME} PRIVATE Gurobi::Gurobi)
endif()

#-----------------------------------------------
# DEFINITIONS
#-----------------------------------------------

target_compile_definitions(${PROJECT_NAME} PRIVATE 
    _USE_MATH_DEFINES
    _CRT_NO_VA_START_VALIDATION
)

if(ONLINE)
    target_compile_definitions(${PROJECT_NAME} PRIVATE EOLC_ONLINE)
endif()

if(MOSEK)
    target_compile_definitions(${PROJECT_NAME} PRIVATE 
        EOLC_MOSEK
        kMosekLicensePath=\"C:/Users/weid691/mosek/mosek.lic\"
    )
endif()

if(GUROBI)
    target_compile_definitions(${PROJECT_NAME} PRIVATE EOLC_GUROBI)
endif()

if(IGL)
    target_compile_definitions(${PROJECT_NAME} PRIVATE EOLC_IGL)
endif()

#-----------------------------------------------
# OS SPECIFIC SETTINGS
#-----------------------------------------------

if(WIN32)
    # Disable specific warnings
    target_compile_options(${PROJECT_NAME} PRIVATE /wd4996)
else()
    # Enable pedantic warnings
    target_compile_options(${PROJECT_NAME} PRIVATE -Wall -pedantic)
endif()
