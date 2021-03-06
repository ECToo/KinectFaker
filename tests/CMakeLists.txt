cmake_minimum_required( VERSION 3.1 )
project( Tests )

find_package( KinectSDK QUIET )
if (NOT KinectSDK_FOUND)
    find_path(KinectSDK_INCLUDE_DIRS
        NAMES "NuiApi.h"
        DOC "Kinect10 Header directory"
    )
endif()

#search catch
find_path(
    CATCH_INCLUDE_DIR 
    NAMES catch.hpp
    DOC "Catch include dir"
)

if (NOT CATCH_INCLUDE_DIR)
    message(STATUS "Could not find Catch. Create download target.")
    include(ExternalProject)
    find_package(Git REQUIRED)
    ExternalProject_Add(
        catch
        PREFIX ${CMAKE_BINARY_DIR}/catch
        GIT_REPOSITORY https://github.com/catchorg/Catch2.git
        UPDATE_COMMAND ${GIT_EXECUTABLE} pull
        CONFIGURE_COMMAND ""
        BUILD_COMMAND ""
        INSTALL_COMMAND ""
        LOG_DOWNLOAD ON
    )
    
    # Expose required variable (CATCH_INCLUDE_DIR) to parent scope
    ExternalProject_Get_Property(catch source_dir)
    set(CATCH_INCLUDE_DIR ${source_dir}/single_include/catch2)
endif()



set (SOURCES
    "main.cpp"
    "basic_functionality.cpp"
    "fake_kinect.config"
    )

enable_testing()	
add_executable( tests "${SOURCES}" )
add_test(NAME basic_functionality
    COMMAND tests -b
    WORKING_DIRECTORY $<TARGET_FILE_DIR:tests>
    )
target_include_directories(tests 
PUBLIC 
    ${KinectSDK_INCLUDE_DIRS}
    ${CATCH_INCLUDE_DIR}
)

target_link_libraries(tests PUBLIC
    Kinect10
    )
    
set_target_properties(tests PROPERTIES
    #VS_DEBUGGER_WORKING_DIRECTORY
    #"$(TargetDir)" #idk but the debugger doesnt start with this parameter
    RUNTIME_OUTPUT_DIRECTORY $<TARGET_FILE_DIR:Kinect10>
    )
    


add_custom_command(TARGET tests POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
    "${CMAKE_CURRENT_SOURCE_DIR}/test_file.kif"
    "${CMAKE_CURRENT_SOURCE_DIR}/fake_kinect.config"
    
    "$<TARGET_FILE_DIR:tests>")
    
add_dependencies(tests Kinect10)
