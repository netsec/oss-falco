#
# Copyright (C) 2016-2018 Draios Inc dba Sysdig.
#
# This file is part of falco .
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
include_directories("${PROJECT_SOURCE_DIR}/../sysdig/userspace/libsinsp/third-party/jsoncpp")
include_directories("${PROJECT_SOURCE_DIR}/../sysdig/userspace/libscap")
include_directories("${PROJECT_SOURCE_DIR}/../sysdig/userspace/libsinsp")
include_directories("${PROJECT_BINARY_DIR}/userspace/engine")
include_directories("${LUAJIT_INCLUDE}")
include_directories("${NJSON_INCLUDE}")
include_directories("${CURL_INCLUDE_DIR}")
include_directories("${TBB_INCLUDE_DIR}")

list(APPEND falco_engine_files
	rules.cpp
	falco_common.cpp
	falco_engine.cpp
	json_evt.cpp
	ruleset.cpp
	token_bucket.cpp
	formats.cpp)

list(APPEND falco_engine_include_directories
	"${LUAJIT_INCLUDE}"
	"${NJSON_INCLUDE}"
	"${PROJECT_BINARY_DIR}/userspace/engine")

list(APPEND falco_engine_link_libraries
  "${LPEG_LIB}"
  "${LYAML_LIB}"
  "${LIBYAML_LIB}")

# Create the production library
add_library(falco_engine STATIC ${falco_engine_files})
target_include_directories(falco_engine PUBLIC ${falco_engine_include_directories})
target_link_libraries(falco_engine "${FALCO_SINSP_LIBRARY}" ${falco_engine_link_libraries})

if(BUILD_FOR_TEST)
	# Create the test library with the test version of sinsp
	add_library(falco_engine.test STATIC ${falco_engine_files})
	target_include_directories(falco_engine.test PUBLIC ${falco_engine_include_directories})
	target_link_libraries(falco_engine.test "${FALCO_SINSP_TEST_LIBRARY}" ${falco_engine_link_libraries})
endif()

configure_file(config_falco_engine.h.in config_falco_engine.h)

if(DEFINED FALCO_COMPONENT)
install(DIRECTORY lua
	DESTINATION "${FALCO_SHARE_DIR}"
	COMPONENT "${FALCO_COMPONENT}"
	FILES_MATCHING PATTERN *.lua)
else()
install(DIRECTORY lua
	DESTINATION "${FALCO_SHARE_DIR}"
	FILES_MATCHING PATTERN *.lua)
endif()
