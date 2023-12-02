# build libprotobuf

# TODO(igork) - see if we have to have this (YUBA-15)
cmake_policy(SET CMP0079 NEW)
# end TODO(igork)

set(INCLUDE_LIBPROTOBUF 1)

set(protobuf_WITH_ZLIB OFF CACHE INTERNAL "")
set(protobuf_BUILD_TESTS OFF CACHE INTERNAL "")
if (ANDROID OR IOS)
  set(BUILD_PROTOC_EXE OFF CACHE INTERNAL "")
endif()
# TODO(igork) - to avid conflicts with gRPC; is this the best way?
# add_subdirectory(${at_media_deps_dir}/protobuf/cmake ${CMAKE_BINARY_DIR}/protobuf/src/cmake)
add_subdirectory_once(${at_media_deps_dir}/protobuf/cmake ${CMAKE_BINARY_DIR}/protobuf/src/cmake)
# end TODO(igork)

# On Clang 9 (Starting Android NDK r21b) this is nesserary to compile.
# (Error later seen on Xcode 12 too).
target_compile_options(libprotobuf PRIVATE -Wno-enum-compare-switch)

# On iOS and Android we need to make sure we use the host version of protoc to generate files
set(PROTOBUF_PROTOC_EXECUTABLE "")
if (IOS)
  set(PROTOBUF_PROTOC_EXECUTABLE 
"${CMAKE_BINARY_DIR}/../xcode-osx/protobuf/src/cmake/Release/protoc")
elseif(ANDROID)
  set(PROTOBUF_PROTOC_EXECUTABLE 
"${CMAKE_BINARY_DIR}/../${at_host_os}-${at_host_arch}-release/protobuf/src/cmake/protoc")
else()
  set(PROTOBUF_PROTOC_EXECUTABLE $<TARGET_FILE:protoc>)
endif()

# We don't build protoc for android or ios, but we call 
# set_target_properties on
# the protoc target below, so we need to create a stub target
if (NOT TARGET protoc)
  add_custom_target(protoc)
endif()

if (PROTOBUF_PROTOC_EXECUTABLE)
  set_target_properties(protoc PROPERTIES TARGET_FILE 
${PROTOBUF_PROTOC_EXECUTABLE})
endif()

set(protoc_source_dir ${CMAKE_CURRENT_SOURCE_DIR}/src)

# Copy the protoc binary to the protobuf src directory since
# 'mvn package' expects it there
# add_custom_target(protoc-symlink
#   COMMAND ${CMAKE_COMMAND} -E create_symlink
#   ${PROTOBUF_PROTOC_EXECUTABLE}
#   ${protoc_source_dir}/src/protoc
# )
# TODO(igork) - to avid conflicts with gRPC; is this the best way?
# (commented out above is the original code)
if (NOT TARGET protoc-symlink)
  add_custom_target(protoc-symlink
    COMMAND ${CMAKE_COMMAND} -E create_symlink
    ${PROTOBUF_PROTOC_EXECUTABLE}
    ${protoc_source_dir}/src/protoc
  )
endif()
# end TODO(igork)

set(protoc_java_dir ${protoc_source_dir}/java)

# add_custom_target(protoc-jar
#   COMMAND mvn clean
#   COMMAND mvn package
#   COMMAND ${CMAKE_COMMAND} -E copy_if_different
#           ${protoc_java_dir}/core/target/protobuf-java*.jar
#           ${CMAKE_BINARY_DIR}/protobuf
#   WORKING_DIRECTORY ${protoc_java_dir}
#   DEPENDS protoc-symlink
# )
# TODO(igork) - to avid conflicts with gRPC; is this the best way?
# (commented out above is the original code)
if (NOT TARGET protoc-jar)
  add_custom_target(protoc-jar
    COMMAND mvn clean
    COMMAND mvn package
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
            ${protoc_java_dir}/core/target/protobuf-java*.jar
            ${CMAKE_BINARY_DIR}/protobuf
    WORKING_DIRECTORY ${protoc_java_dir}
    DEPENDS protoc-symlink
  )
endif()
# end TODO(igork)
