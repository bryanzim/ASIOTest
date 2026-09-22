cmake_minimum_required(VERSION 3.11)
project(ASIOTest DESCRIPTION "Generic ASIO Test Suite")

find_package(cxxopts CONFIG REQUIRED)
find_package(dechamps_cpputil CONFIG REQUIRED)
find_package(dechamps_cpplog CONFIG REQUIRED)
find_package(dechamps_ASIOUtil CONFIG REQUIRED)
find_package(sndfile CONFIG REQUIRED)

set(CMAKE_CXX_STANDARD 20)
# clang-cl does not implement MSVC /analyze.
# /wd4458 and /wd4456 suppress shadowing warnings from lambdas that intentionally
# reuse the name of the variable they initialize.
set(_warning_options /WX /W4 /permissive- /wd4458 /wd4456)
if(NOT (CMAKE_CXX_COMPILER_ID STREQUAL "Clang" AND CMAKE_CXX_SIMULATE_ID STREQUAL "MSVC"))
	list(APPEND _warning_options /analyze)
endif()
add_compile_options(${_warning_options})

add_library(ASIOTest SHARED test.cpp dll.def)
target_link_libraries(ASIOTest
	PRIVATE dechamps_ASIOUtil::asiosdk_ginclude
	PRIVATE dechamps_ASIOUtil::asiosdk_asio
	PRIVATE dechamps_ASIOUtil::asio
	PRIVATE dechamps_cpplog::log
	PRIVATE dechamps_cpputil::endian
	PRIVATE dechamps_cpputil::find
	PRIVATE dechamps_cpputil::string
	PRIVATE cxxopts::cxxopts
	PRIVATE SndFile::sndfile
)

target_include_directories(ASIOTest INTERFACE "$<INSTALL_INTERFACE:include>")

install(FILES test.h DESTINATION include/ASIOTest)
install(TARGETS ASIOTest EXPORT ASIOTest-config RUNTIME DESTINATION bin ARCHIVE DESTINATION lib)
install(EXPORT ASIOTest-config NAMESPACE ASIOTest:: DESTINATION lib/cmake/ASIOTest)
