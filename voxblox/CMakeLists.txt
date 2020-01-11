cmake_minimum_required(VERSION 2.8.12)
project(voxblox)

find_package(catkin_simple REQUIRED )
catkin_simple(ALL_DEPS_REQUIRED)

set(CMAKE_MACOSX_RPATH 0)
add_definitions(-std=c++11 -Wall -Wextra)

############
# PROTOBUF #
############

find_package(Protobuf REQUIRED)
protobuf_generate_cpp(PROTO_SRCS PROTO_HDRS
    proto/voxblox/Block.proto
    proto/voxblox/Layer.proto)

set(BLOCK_IN_HEADER "${CMAKE_CURRENT_BINARY_DIR}/Block.pb.h")
set(BLOCK_OUT_HEADER "${CMAKE_CURRENT_SOURCE_DIR}/include/voxblox/Block.pb.h")
set(LAYER_IN_HEADER "${CMAKE_CURRENT_BINARY_DIR}/Layer.pb.h")
set(LAYER_OUT_HEADER "${CMAKE_CURRENT_SOURCE_DIR}/include/voxblox/Layer.pb.h")
add_custom_command(OUTPUT ${BLOCK_OUT_HEADER} ${LAYER_OUT_HEADER}
    COMMAND ${CMAKE_COMMAND} -E copy_if_different ${BLOCK_IN_HEADER} ${BLOCK_OUT_HEADER}
    COMMAND ${CMAKE_COMMAND} -E copy_if_different ${LAYER_IN_HEADER} ${LAYER_OUT_HEADER}
    DEPENDS ${BLOCK_IN_HEADER} ${LAYER_IN_HEADER}
)
set(DEVEL_PROTO_HEADERS
    ${BLOCK_OUT_HEADER}
    ${LAYER_OUT_HEADER})

####################
# SET SOURCE FILES #
####################

set("${PROJECT_NAME}_SRCS"
  src/alignment/icp.cc
  src/core/block.cc
  src/core/esdf_map.cc
  src/core/tsdf_map.cc
  src/integrator/esdf_integrator.cc
  src/integrator/esdf_occ_integrator.cc
  src/integrator/integrator_utils.cc
  src/integrator/intensity_integrator.cc
  src/integrator/tsdf_integrator.cc
  src/io/mesh_ply.cc
  src/io/sdf_ply.cc
  src/mesh/marching_cubes.cc
  src/simulation/objects.cc
  src/simulation/simulation_world.cc
  src/utils/camera_model.cc
  src/utils/evaluation_utils.cc
  src/utils/layer_utils.cc
  src/utils/neighbor_tools.cc
  src/utils/protobuf_utils.cc
  src/utils/timing.cc
  src/utils/voxel_utils.cc
)

#############
# LIBRARIES #
#############
cs_add_library(${PROJECT_NAME}_proto
    ${PROTO_SRCS} ${DEVEL_PROTO_HEADERS}
)

cs_add_library(${PROJECT_NAME}
  ${${PROJECT_NAME}_SRCS}
)
target_link_libraries(${PROJECT_NAME} ${PROJECT_NAME}_proto ${Protobuf_LIBRARIES})

############
# BINARIES #
############

add_executable(tsdf_to_esdf
  test/tsdf_to_esdf.cc
)
target_link_libraries(tsdf_to_esdf ${PROJECT_NAME})

add_executable(test_load_esdf
  test/test_load_esdf.cc
)
target_link_libraries(test_load_esdf ${PROJECT_NAME})

#########
# TESTS #
#########
add_custom_target(test_data)
add_custom_command(TARGET test_data
                  COMMAND rm -rf test_data
                  COMMAND mkdir -p test_data
                  COMMAND cp -r ${CMAKE_SOURCE_DIR}/test/test_data/*
                                test_data/ || :)

#add_definitions(-DVISUALIZE_UNIT_TEST_RESULTS)

catkin_add_gtest(test_approx_hash_array
  test/test_approx_hash_array.cc
)
target_link_libraries(test_approx_hash_array ${PROJECT_NAME})

catkin_add_gtest(test_tsdf_map
  test/test_tsdf_map.cc
)
target_link_libraries(test_tsdf_map ${PROJECT_NAME})

catkin_add_gtest(test_protobuf
  test/test_protobuf.cc
)
target_link_libraries(test_protobuf ${PROJECT_NAME})

catkin_add_gtest(test_tsdf_interpolator
  test/test_tsdf_interpolator.cc
)
target_link_libraries(test_tsdf_interpolator ${PROJECT_NAME})

catkin_add_gtest(test_layer
  test/test_layer.cc
)
target_link_libraries(test_layer ${PROJECT_NAME})

catkin_add_gtest(test_merge_integration
  test/test_merge_integration.cc
)
target_link_libraries(test_merge_integration ${PROJECT_NAME})

catkin_add_gtest(test_layer_utils
  test/test_layer_utils.cc
)
target_link_libraries(test_layer_utils ${PROJECT_NAME})

catkin_add_gtest(test_sdf_integrators
  test/test_sdf_integrators.cc
)
target_link_libraries(test_sdf_integrators ${PROJECT_NAME})

catkin_add_gtest(test_bucket_queue
  test/test_bucket_queue.cc
)
target_link_libraries(test_bucket_queue ${PROJECT_NAME})

catkin_add_gtest(test_clear_spheres
  test/test_clear_spheres.cc
)
target_link_libraries(test_clear_spheres ${PROJECT_NAME})

##########
# EXPORT #
##########
cs_install()
cs_export()
