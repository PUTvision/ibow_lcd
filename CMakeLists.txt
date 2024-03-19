cmake_minimum_required(VERSION 3.1 FATAL_ERROR)
project(ibow_lcd LANGUAGES CXX)

message(STATUS "Build type: " ${CMAKE_BUILD_TYPE})

# Setting the flags for profiling information or not
if(CMAKE_BUILD_TYPE MATCHES Release)
  message(STATUS "Setting Release options")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -O3 -march=native")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -O3 -march=native")
elseif(CMAKE_BUILD_TYPE MATCHES Debug)
  message(STATUS "Setting Debug options")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -O1 -pg -march=native")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -O1 -pg -march=native")
endif()

# Set the C++ standard
set(CMAKE_CXX_STANDARD 14)

if (NOT YOCTO_BUILD)
  set(YOCTO_BUILD true)
endif()

# Printing the compiling flags
message(STATUS "Compiler flags: ${CMAKE_CXX_FLAGS}")

# Other packages
find_package(OpenCV REQUIRED) # OpenCV
find_package(Boost REQUIRED COMPONENTS system filesystem)
find_package(obindex2 REQUIRED) # obindex2
find_package(OpenMP REQUIRED) # OpenMP
if (OPENMP_FOUND)
  set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
  set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
endif()


include_directories(include
                    external
                    ${OpenCV_INCLUDE_DIRS}
                    ${Boost_INCLUDE_DIRS}
                    ${obindex2_INCLUDE_DIRS}
                    )

### Targets ###
# Library
add_library(lcdetector
            SHARED
            include/ibow_lcd/island.h
            include/ibow_lcd/lcdetector.h
            src/lcdetector.cc)

target_link_libraries(lcdetector
                      ${OpenCV_LIBRARIES}
                      ${Boost_LIBRARIES}
                      obindex2::obindex2
                      )

set_target_properties(lcdetector
        PROPERTIES PUBLIC_HEADER
        "include/ibow_lcd/island.h;include/ibow_lcd/lcdetector.h"
)

if (NOT ${YOCTO_BUILD})
  target_include_directories(obindex2 PUBLIC
    $<BUILD_INTERFACE:${CMAKE_INSTALL_LIBDIR}>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_LIBDIR}>
  )
endif()

install(TARGETS lcdetector
        EXPORT lcdetectorConfig
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/ibow_lcd
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

export(TARGETS
        lcdetector
        NAMESPACE lcdetector::
        FILE "${CMAKE_CURRENT_BINARY_DIR}/lcdetectorConfig.cmake"
)

install(EXPORT
        lcdetectorConfig
        DESTINATION "${CMAKE_INSTALL_LIBDIR}/lcdetector/cmake"
        NAMESPACE lcdetector::
)
