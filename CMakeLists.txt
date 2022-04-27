# Disable MSVC++ warning C4996: 'may be unsafe/disable deprecation'
add_definitions(-D_CRT_SECURE_NO_WARNINGS)

# Add option to enable experimental build
option(OPENIW_STEAM_EMU_EXPERIMENTAL_BUILD "Enable goldberg emulator experimental build" OFF)

# Find the protobuf compiler and libraries
find_package(Protobuf CONFIG REQUIRED)

# Generate the .h and .cxx files for dll/net.proto
protobuf_generate_cpp(PROTO_SRCS PROTO_HDRS dll/net.proto)

# Print path to generated files
message(STATUS "PROTO_SRCS = ${PROTO_SRCS}")
message(STATUS "PROTO_HDRS = ${PROTO_HDRS}")
message(STATUS "PROTOBUF_INCLUDE_DIRS = ${PROTOBUF_INCLUDE_DIRS}")
message(STATUS "PROTOBUF_LIBRARIES = ${PROTOBUF_LIBRARIES}")
message(STATUS "PROTOBUF_PROTOC_EXECUTABLE = ${PROTOBUF_PROTOC_EXECUTABLE}")

# Setup the lib/exe names for the targets
set(LIB_STEAM_API steam_api)
set(LIB_STEAMCLIENT steamclient)
set(LIB_STEAMNETWORKINGSOCKETS steamnetworkingsockets)
set(BIN_LOBBY_CONNECT lobby_connect)
set(BIN_GENERATE_INTERFACES_FILE generate_interfaces_file)
set(LIB_STEAM_API64 steam_api64)
set(LIB_STEAMCLIENT64 steamclient64)
set(LIB_STEAMNETWORKINGSOCKETS64 steamnetworkingsockets64)
set(BIN_LOBBY_CONNECT64 lobby_connect64)
set(BIN_GENERATE_INTERFACES_FILE64 generate_interfaces_file64)

# Gather the files that are shared between multiple targets
file(GLOB DLL_SRC_SHARED
  dll/*.h
  dll/*.cpp
)

file(GLOB DETOURS_SRC_SHARED
  detours/*.cpp
)

file(GLOB OVERLAY_EXPERIMENTAL_SRC_SHARED
  overlay_experimental/*.cpp
  overlay_experimental/windows/*.cpp
  ImGui/*.cpp
  ImGui/impls/*.cpp
  ImGui/impls/windows/*.cpp
  glew/glew.c
)

###################################################
# Setup for the steam_api(64).dll / libsteam_api.so
###################################################

# Setup the target
add_library(${LIB_STEAM_API}
  SHARED
  $<$<BOOL:${EMU_EXPERIMENTAL_BUILD}>:${DETOURS_SRC_SHARED}>
  $<$<AND:$<BOOL:${EMU_EXPERIMENTAL_BUILD}>,$<BOOL:${EMU_OVERLAY}>>:${OVERLAY_EXPERIMENTAL_SRC_SHARED}>
  ${DLL_SRC_SHARED}
  ${PROTO_SRCS}
  ${PROTO_HDRS}
)

# Include the required directories
target_include_directories(${LIB_STEAM_API}
  PRIVATE
  ${PROTOBUF_INCLUDE_DIRS}
  ${CMAKE_CURRENT_BINARY_DIR}
  ${CMAKE_CURRENT_SOURCE_DIR}/ImGui
  ${CMAKE_CURRENT_SOURCE_DIR}/glew/include
)

# Link the required libraries
target_link_libraries(${LIB_STEAM_API}
  PRIVATE
  protobuf::libprotobuf
  $<$<BOOL:${WIN32}>:ws2_32>
  $<$<BOOL:${WIN32}>:iphlpapi>
  $<$<AND:$<BOOL:${WIN32}>,$<BOOL:${EMU_EXPERIMENTAL_BUILD}>,$<BOOL:${EMU_OVERLAY}>>:opengl32.lib>
  $<$<AND:$<BOOL:${WIN32}>,$<BOOL:${EMU_EXPERIMENTAL_BUILD}>,$<BOOL:${EMU_OVERLAY}>>:Winmm.lib>
)

# Add target compile definitions
target_compile_definitions(${LIB_STEAM_API}
  PRIVATE
  $<$<CONFIG:>:EMU_RELEASE_BUILD>
  $<$<CONFIG:Release>:EMU_RELEASE_BUILD>
  $<$<CONFIG:RelWithDebInfo>:EMU_RELEASE_BUILD>
  $<$<CONFIG:MinSizeRel>:EMU_RELEASE_BUILD>
  $<$<BOOL:${EMU_EXPERIMENTAL_BUILD}>:EMU_EXPERIMENTAL_BUILD>
  $<$<BOOL:${EMU_OVERLAY}>:EMU_OVERLAY>
  $<$<AND:$<BOOL:${WIN32}>,$<BOOL:${EMU_EXPERIMENTAL_BUILD}>,$<BOOL:${EMU_OVERLAY}>>:GLEW_STATIC>
)

# Setup the target (64)
add_library(${LIB_STEAM_API64}
  SHARED
  $<$<BOOL:${EMU_EXPERIMENTAL_BUILD}>:${DETOURS_SRC_SHARED}>
  $<$<AND:$<BOOL:${EMU_EXPERIMENTAL_BUILD}>,$<BOOL:${EMU_OVERLAY}>>:${OVERLAY_EXPERIMENTAL_SRC_SHARED}>
  ${DLL_SRC_SHARED}
  ${PROTO_SRCS}
  ${PROTO_HDRS}
)

# Include the required directories
target_include_directories(${LIB_STEAM_API64}
  PRIVATE
  ${PROTOBUF_INCLUDE_DIRS}
  ${CMAKE_CURRENT_BINARY_DIR}
  ${CMAKE_CURRENT_SOURCE_DIR}/ImGui
  ${CMAKE_CURRENT_SOURCE_DIR}/glew/include
)

# Link the required libraries
target_link_libraries(${LIB_STEAM_API64}
  PRIVATE
  protobuf::libprotobuf
  $<$<BOOL:${WIN32}>:ws2_32>
  $<$<BOOL:${WIN32}>:iphlpapi>
  $<$<AND:$<BOOL:${WIN32}>,$<BOOL:${EMU_EXPERIMENTAL_BUILD}>,$<BOOL:${EMU_OVERLAY}>>:opengl32.lib>
  $<$<AND:$<BOOL:${WIN32}>,$<BOOL:${EMU_EXPERIMENTAL_BUILD}>,$<BOOL:${EMU_OVERLAY}>>:Winmm.lib>
)

# Add target compile definitions
target_compile_definitions(${LIB_STEAM_API64}
  PRIVATE
  $<$<CONFIG:>:EMU_RELEASE_BUILD>
  $<$<CONFIG:Release>:EMU_RELEASE_BUILD>
  $<$<CONFIG:RelWithDebInfo>:EMU_RELEASE_BUILD>
  $<$<CONFIG:MinSizeRel>:EMU_RELEASE_BUILD>
  $<$<BOOL:${EMU_EXPERIMENTAL_BUILD}>:EMU_EXPERIMENTAL_BUILD>
  $<$<BOOL:${EMU_OVERLAY}>:EMU_OVERLAY>
  $<$<AND:$<BOOL:${WIN32}>,$<BOOL:${EMU_EXPERIMENTAL_BUILD}>,$<BOOL:${EMU_OVERLAY}>>:GLEW_STATIC>
)

set_target_properties(${LIB_STEAM_API64} PROPERTIES
LINK_FLAGS
  /MACHINE:X64
)

if(OPENIW_STEAM_EMU_EXPERIMENTAL_BUILD)

########################################################
# Setup for the steamclient(64).dll / libsteamclient.so?
########################################################

# Setup the target
add_library(${LIB_STEAMCLIENT}
  SHARED
  steamclient.cpp
)

# Add target compile definitions
target_compile_definitions(${LIB_STEAMCLIENT}
  PRIVATE
  $<$<CONFIG:>:EMU_RELEASE_BUILD>
  $<$<CONFIG:Release>:EMU_RELEASE_BUILD>
  $<$<CONFIG:RelWithDebInfo>:EMU_RELEASE_BUILD>
  $<$<CONFIG:MinSizeRel>:EMU_RELEASE_BUILD>
  $<$<BOOL:${EMU_EXPERIMENTAL_BUILD}>:EMU_EXPERIMENTAL_BUILD>
)

# Setup the target (64)
add_library(${LIB_STEAMCLIENT64}
  SHARED
  steamclient.cpp
)

# Add target compile definitions
target_compile_definitions(${LIB_STEAMCLIENT64}
  PRIVATE
  $<$<CONFIG:>:EMU_RELEASE_BUILD>
  $<$<CONFIG:Release>:EMU_RELEASE_BUILD>
  $<$<CONFIG:RelWithDebInfo>:EMU_RELEASE_BUILD>
  $<$<CONFIG:MinSizeRel>:EMU_RELEASE_BUILD>
  $<$<BOOL:${EMU_EXPERIMENTAL_BUILD}>:EMU_EXPERIMENTAL_BUILD>
)

set_target_properties(${LIB_STEAMCLIENT64} PROPERTIES
LINK_FLAGS
  /MACHINE:X64
)

##############################################################################
# Setup for the steamnetworkingsockets(64).dll / libsteamnetworkingsockets.so?
##############################################################################

# # Setup the target
# add_library(${LIB_STEAMNETWORKINGSOCKETS}
# 	SHARED
# 	steamnetworkingsockets.cpp
# )

# # Add target compile definitions
# target_compile_definitions(${LIB_STEAMNETWORKINGSOCKETS}
# 	PRIVATE
# 	$<$<CONFIG:>:EMU_RELEASE_BUILD>
#   $<$<CONFIG:Release>:EMU_RELEASE_BUILD>
# 	$<$<CONFIG:RelWithDebInfo>:EMU_RELEASE_BUILD>
# 	$<$<CONFIG:MinSizeRel>:EMU_RELEASE_BUILD>
# 	$<$<BOOL:${EMU_EXPERIMENTAL_BUILD}>:EMU_EXPERIMENTAL_BUILD>
# )

# # Setup install rules for the target
# if(WIN32)
#   install(TARGETS
#     ${LIB_STEAMNETWORKINGSOCKETS}
#     RUNTIME DESTINATION ./
#   )
# else()
# 	install(TARGETS
#     ${LIB_STEAMNETWORKINGSOCKETS}
#     LIBRARY DESTINATION ./
#   )
# endif()

# if(NOT WIN32)
#   message(STATUS "Target library 'steamnetworkingsockets' is only supported for windows at this time... Disabling Build ALL inclusion for this target")
# 	set_target_properties(${LIB_STEAMNETWORKINGSOCKETS} PROPERTIES EXCLUDE_FROM_ALL 1 EXCLUDE_FROM_DEFAULT_BUILD 1)
# endif()

###########################################################################
# Setup for the lobby_connect(64).exe / lobby_connect
###########################################################################

# Setup the target
add_executable(${BIN_LOBBY_CONNECT}
  lobby_connect.cpp
  $<$<BOOL:${EMU_EXPERIMENTAL_BUILD}>:${DETOURS_SRC_SHARED}>
  ${DLL_SRC_SHARED}
  ${PROTO_SRCS}
  ${PROTO_HDRS}
)

target_include_directories(${BIN_LOBBY_CONNECT}
  PRIVATE
  ${PROTOBUF_INCLUDE_DIRS}
  ${CMAKE_CURRENT_BINARY_DIR}
)

# Link the required libraries
target_link_libraries(${BIN_LOBBY_CONNECT}
  PRIVATE
  protobuf::libprotobuf
  $<$<BOOL:${WIN32}>:ws2_32>
  $<$<BOOL:${WIN32}>:iphlpapi>
  $<$<BOOL:${WIN32}>:comdlg32>
  ${CMAKE_DL_LIBS}
  -debug:none
)

# Add target compile definitions
target_compile_definitions(${BIN_LOBBY_CONNECT}
  PRIVATE
  NO_DISK_WRITES
  LOBBY_CONNECT
  $<$<CONFIG:>:EMU_RELEASE_BUILD>
  $<$<CONFIG:Release>:EMU_RELEASE_BUILD>
  $<$<CONFIG:RelWithDebInfo>:EMU_RELEASE_BUILD>
  $<$<CONFIG:MinSizeRel>:EMU_RELEASE_BUILD>
  $<$<BOOL:${EMU_EXPERIMENTAL_BUILD}>:EMU_EXPERIMENTAL_BUILD>
)

# Setup the target
add_executable(${BIN_LOBBY_CONNECT64}
  lobby_connect.cpp
  $<$<BOOL:${EMU_EXPERIMENTAL_BUILD}>:${DETOURS_SRC_SHARED}>
  ${DLL_SRC_SHARED}
  ${PROTO_SRCS}
  ${PROTO_HDRS}
)

target_include_directories(${BIN_LOBBY_CONNECT64}
  PRIVATE
  ${PROTOBUF_INCLUDE_DIRS}
  ${CMAKE_CURRENT_BINARY_DIR}
)

# Link the required libraries
target_link_libraries(${BIN_LOBBY_CONNECT64}
  PRIVATE
  protobuf::libprotobuf
  $<$<BOOL:${WIN32}>:ws2_32>
  $<$<BOOL:${WIN32}>:iphlpapi>
  $<$<BOOL:${WIN32}>:comdlg32>
  ${CMAKE_DL_LIBS}
  -debug:none
)

# Add target compile definitions
target_compile_definitions(${BIN_LOBBY_CONNECT64}
  PRIVATE
  NO_DISK_WRITES
  LOBBY_CONNECT
  $<$<CONFIG:>:EMU_RELEASE_BUILD>
  $<$<CONFIG:Release>:EMU_RELEASE_BUILD>
  $<$<CONFIG:RelWithDebInfo>:EMU_RELEASE_BUILD>
  $<$<CONFIG:MinSizeRel>:EMU_RELEASE_BUILD>
  $<$<BOOL:${EMU_EXPERIMENTAL_BUILD}>:EMU_EXPERIMENTAL_BUILD>
)

set_target_properties(${BIN_LOBBY_CONNECT64} PROPERTIES
LINK_FLAGS
  /MACHINE:X64
)

###########################################################################
# Setup for the generate_interfaces_file(64).exe / generate_interfaces_file
###########################################################################

# Setup the target
add_executable(${BIN_GENERATE_INTERFACES_FILE}
  generate_interfaces_file.cpp
)

# Link the required libraries
target_link_libraries(${BIN_GENERATE_INTERFACES_FILE}
  PRIVATE
  -debug:none
)

# Setup the target
add_executable(${BIN_GENERATE_INTERFACES_FILE64}
  generate_interfaces_file.cpp
)

# Link the required libraries
target_link_libraries(${BIN_GENERATE_INTERFACES_FILE64}
  PRIVATE
  -debug:none
)

set_target_properties(${BIN_GENERATE_INTERFACES_FILE64} PROPERTIES
LINK_FLAGS
  /MACHINE:X64
)

endif()
