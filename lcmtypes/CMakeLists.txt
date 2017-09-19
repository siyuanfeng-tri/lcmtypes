if(WIN32)
  set(CONFIG_INSTALL_DIR CMake)
else()
  set(CONFIG_INSTALL_DIR lib${LIB_SUFFIX}/${PROJECT_NAME}/cmake)
endif()

if(PYTHONINTERP_FOUND)
  set(python_args PYTHON_SOURCES python_install_sources)
endif()
if(JAVA_FOUND AND TARGET lcm-java)
  set(java_args JAVA_SOURCES java_sources)
endif()

lcm_wrap_types(
  C_EXPORT robotlocomotion_lcmtypes
  C_SOURCES c_sources
  C_HEADERS c_install_headers
  CPP_HEADERS cpp_install_headers
  CREATE_C_AGGREGATE_HEADER
  CREATE_CPP_AGGREGATE_HEADER
  ${python_args}
  ${java_args}
  header_t.lcm
  image_array_t.lcm
  image_t.lcm
  grasp_transition_state_t.lcm
  plan_control_t.lcm
  plan_status_t.lcm
  point_t.lcm
  pose_t.lcm
  pose_stamped_t.lcm
  quaternion_t.lcm
  residual_observer_state_t.lcm
  robot_plan_t.lcm
  robot_plan_with_supports_t.lcm
  robot_plan_w_keyframes_t.lcm
  support_body_t.lcm
  support_element_t.lcm
  support_sequence_t.lcm
  viewer_command_t.lcm
  viewer_draw_t.lcm
  viewer_geometry_data_t.lcm
  viewer_link_data_t.lcm
  viewer_load_robot_t.lcm
  viewer2_comms_t.lcm
  hardware/iiwa_command.lcm
  hardware/iiwa_status.lcm
  hardware/schunk_wsg_command.lcm
  hardware/schunk_wsg_status.lcm
  )


lcm_add_library(robotlocomotion-lcmtypes C ${c_sources} ${c_install_headers})
generate_export_header(robotlocomotion-lcmtypes
  BASE_NAME robotlocomotion_lcmtypes)
target_include_directories(robotlocomotion-lcmtypes INTERFACE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>)
target_link_libraries(robotlocomotion-lcmtypes PUBLIC lcmtypes_bot2-core)

lcm_add_library(robotlocomotion-lcmtypes-cpp CPP ${cpp_install_headers})
target_include_directories(robotlocomotion-lcmtypes-cpp INTERFACE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>)
target_link_libraries(robotlocomotion-lcmtypes-cpp INTERFACE lcmtypes_bot2-core-cpp)

lcm_install_headers(
  ${CMAKE_CURRENT_BINARY_DIR}/robotlocomotion_lcmtypes_export.h
  ${c_install_headers}
  ${cpp_install_headers}
  DESTINATION include/lcmtypes
)

install(TARGETS robotlocomotion-lcmtypes robotlocomotion-lcmtypes-cpp
  EXPORT ${PROJECT_NAME}Targets
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib${LIB_SUFFIX}
  ARCHIVE DESTINATION lib${LIB_SUFFIX}
  INCLUDES DESTINATION include/lcmtypes
)

if(PYTHONINTERP_FOUND)
  lcm_install_python(${python_install_sources})
endif()

if(JAVA_FOUND)
  if(NOT TARGET lcm-java)
    message(STATUS "lcm-java not found: not building Java LCM type bindings")
  else()
    include(UseJava)
    add_jar(robotlocomotion-lcmtypes-jar
      OUTPUT_NAME lcmtypes_robotlocomotion
      INCLUDE_JARS lcmtypes_bot2-core-jar lcm-java
      SOURCES ${java_sources}
    )

    install_jar(robotlocomotion-lcmtypes-jar share/java)

    set(ROBOTLOCOMOTION-LCMTYPES_INCLUDE_JAVA
      "include(\${CMAKE_CURRENT_LIST_DIR}/${PROJECT_NAME}JavaTargets.cmake)"
    )
    export_jars(
      FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}JavaTargets.cmake
      TARGETS robotlocomotion-lcmtypes-jar
    )
    install_jar_exports(
      FILE ${PROJECT_NAME}JavaTargets.cmake
      DESTINATION ${CONFIG_INSTALL_DIR}
      TARGETS robotlocomotion-lcmtypes-jar
    )
  endif()
endif()

configure_package_config_file(
  ${PROJECT_SOURCE_DIR}/${PROJECT_NAME}Config.cmake.in
  ${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
  INSTALL_DESTINATION ${CONFIG_INSTALL_DIR}
)

# Exported targets for build directory
export(
  TARGETS robotlocomotion-lcmtypes robotlocomotion-lcmtypes-cpp
  NAMESPACE ${PROJECT_NAME}::
  FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}Targets.cmake
)

# Exported targets for installation
install(EXPORT ${PROJECT_NAME}Targets
  NAMESPACE ${PROJECT_NAME}::
  DESTINATION ${CONFIG_INSTALL_DIR}
  FILE ${PROJECT_NAME}Targets.cmake
)

# Install package configuration files
install(FILES
  ${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
  DESTINATION ${CONFIG_INSTALL_DIR}
)
