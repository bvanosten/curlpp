project(curlpp) 

if(WIN32)
    # cmake 3.4 is required for CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS
    cmake_minimum_required(VERSION 3.4)
else()
    cmake_minimum_required(VERSION 3.0)
endif()

set(CMAKE_CXX_STANDARD 11) # C++11...
set(CMAKE_CXX_STANDARD_REQUIRED ON) #...is required...
set(CMAKE_CXX_EXTENSIONS OFF) #...without compiler extensions like gnu++11

set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/bin")

#########################################################################################
# Root folders
set(ProjectRootDir ${${PROJECT_NAME}_SOURCE_DIR}/)
message(STATUS "\tUsing Sources From: ${ProjectRootDir}")

set(ProjectBinaryDir ${${PROJECT_NAME}_BINARY_DIR}/)
message(STATUS "\tPutting all major output in: ${ProjectBinaryDir}")

# Source Subfolders
set(SrcFolder ${ProjectRootDir}src/)
message(STATUS "\tInputs of type .cpp at: ${SrcFolder}")

set(IncludeFolder ${ProjectRootDir}include/)
message(STATUS "\tInputs of type .h and .hpp at: ${IncludeFolder}")

set(ExamplesFolder ${ProjectRootDir}examples/)
message(STATUS "\tInputs to be built as standalone examples are at: ${ExamplesFolder}")

option(CURLPP_EXAMPLES "Enable Examples" ON)

#########################################################################################
# For more verbose displays
function(DisplayList Header ListToShow)
    message(STATUS "\t${Header}")
    foreach(ListItem ${ListToShow})
        message(STATUS "\t\t${ListItem}")
    endforeach(ListItem ${ListToShow})
endfunction(DisplayList)

#########################################################################################
# File Lists
set(HeaderFileList
  ${IncludeFolder}utilspp/EmptyType.hpp
  ${IncludeFolder}utilspp/NullType.hpp
  ${IncludeFolder}utilspp/ThreadingSingle.hpp
  ${IncludeFolder}utilspp/TypeTrait.hpp
  ${IncludeFolder}utilspp/clone_ptr.hpp
  ${IncludeFolder}utilspp/TypeList.hpp
  ${IncludeFolder}utilspp/NonCopyable.hpp
  ${IncludeFolder}utilspp/SmartPtr.hpp
  ${IncludeFolder}utilspp/ThreadingFactoryMutex.hpp
  ${IncludeFolder}curlpp/Exception.hpp
  ${IncludeFolder}curlpp/Option.hpp
  ${IncludeFolder}curlpp/Info.hpp
  ${IncludeFolder}curlpp/Easy.hpp
  ${IncludeFolder}curlpp/Form.hpp
  ${IncludeFolder}curlpp/Types.hpp
  ${IncludeFolder}curlpp/OptionBase.hpp
  ${IncludeFolder}curlpp/Infos.hpp
  ${IncludeFolder}curlpp/Options.hpp
  ${IncludeFolder}curlpp/Multi.hpp
  ${IncludeFolder}curlpp/internal/OptionContainerType.hpp
  ${IncludeFolder}curlpp/internal/OptionContainer.hpp
  ${IncludeFolder}curlpp/internal/SList.hpp
  ${IncludeFolder}curlpp/internal/CurlHandle.hpp
  ${IncludeFolder}curlpp/internal/OptionSetter.hpp
  ${IncludeFolder}curlpp/internal/OptionList.hpp
  ${IncludeFolder}curlpp/cURLpp.hpp
  ${SrcFolder}curlpp/autolink.h
)
DisplayList("Using Header Files:" "${HeaderFileList}")

set(SourceFileList
  ${SrcFolder}curlpp/Form.cpp
  ${SrcFolder}curlpp/Exception.cpp
  ${SrcFolder}curlpp/Options.cpp
  ${SrcFolder}curlpp/Option.cpp
  ${SrcFolder}curlpp/Options.custom.ins
  ${SrcFolder}curlpp/Info.cpp
  ${SrcFolder}curlpp/Multi.cpp
  ${SrcFolder}curlpp/OptionBase.cpp
  ${SrcFolder}curlpp/internal/SList.cpp
  ${SrcFolder}curlpp/internal/OptionSetter.cpp
  ${SrcFolder}curlpp/internal/OptionList.cpp
  ${SrcFolder}curlpp/internal/CurlHandle.cpp
  ${SrcFolder}curlpp/Infos.cpp
  ${SrcFolder}curlpp/Easy.cpp
  ${SrcFolder}curlpp/cURLpp.cpp
)
DisplayList("Using Source Files:" "${SourceFileList}")

set(ExamplesFileList
  ${ExamplesFolder}example00.cpp
  ${ExamplesFolder}example01.cpp
  ${ExamplesFolder}example02.cpp
  ${ExamplesFolder}example03.cpp
  ${ExamplesFolder}example04.cpp
  ${ExamplesFolder}example05.cpp
  ${ExamplesFolder}example06.cpp
  ${ExamplesFolder}example07.cpp
  ${ExamplesFolder}example08.cpp
  ${ExamplesFolder}example09.cpp
  ${ExamplesFolder}example10.cpp
  ${ExamplesFolder}example11.cpp
  ${ExamplesFolder}example12.cpp
  ${ExamplesFolder}example13.cpp
  ${ExamplesFolder}example14.cpp
  ${ExamplesFolder}example16.cpp
  ${ExamplesFolder}example15.cpp
  ${ExamplesFolder}example17.cpp
  ${ExamplesFolder}example18.cpp
  ${ExamplesFolder}example19.cpp
  ${ExamplesFolder}example20.cpp
  ${ExamplesFolder}example21.cpp
  ${ExamplesFolder}example22.cpp
  ${ExamplesFolder}example23.cpp
  ${ExamplesFolder}example24.cpp
)
DisplayList("Using Example Files:" "${ExamplesFileList}")

#########################################################################################
# Look for dependencies

# Documented at https://cmake.org/cmake/help/v3.0/module/FindCURL.html?highlight=curlpp
# Seems simple.

message(STATUS "Looking for CURL")
include(FindCURL)
find_package(CURL REQUIRED)

if(${CURL_FOUND})
  message(STATUS "Found CURL version: ${CURL_VERSION_STRING}")
  message(STATUS "Using CURL include dir(s): ${CURL_INCLUDE_DIRS}")
  message(STATUS "Using CURL lib(s): ${CURL_LIBRARIES}")
else(${CURL_FOUND})
  message(FATAL_ERROR "Could not find CURL")
endif(${CURL_FOUND})

# All following targets should search these directories for headers
include_directories( 
  ${IncludeFolder}
  ${CURL_INCLUDE_DIRS}
)

#########################################################################################
# Connect all the found parts

# Define main library
message(STATUS 
  "Defining the shared library for ${PROJECT_NAME} including "
  "all source and header files not in examples."
)
add_library(${PROJECT_NAME} STATIC ${HeaderFileList} ${SourceFileList})
target_link_libraries(${PROJECT_NAME} ${CURL_LIBRARIES})

if(CURLPP_EXAMPLES)
  # Create a meta target to create examples
  add_custom_target(build_all_examples COMMENT A target that requires all the examples.)

  message(STATUS "Creating build rules for Examples:")
  foreach(Example ${ExamplesFileList})
    message(STATUS "\tCreating build rule for ${Example}")
    # Get the name without extension or directory
    get_filename_component(ExampleName ${Example} NAME_WE)
    # Define example executable
    add_executable(${ExampleName} ${Example})
    # Link example against curlpp
    target_link_libraries(${ExampleName} ${PROJECT_NAME})
    # make the meta target depend on this example.
    add_dependencies(build_all_examples ${ExampleName})
  endforeach(Example ${ExamplesFileList})
endif()

