cmake_minimum_required(VERSION 2.8)

set(CMAKE_DISABLE_SOURCE_CHANGES ON)
set(CMAKE_DISABLE_IN_SOURCE_BUILD ON)

project(tfs)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

include(cotire)

add_compile_options(-Wall -Werror -pipe -fvisibility=hidden)
add_definitions(-DBOOST_STACKTRACE_USE_BACKTRACE)

set(CMAKE_CXX_FLAGS_PERFORMANCE "${CMAKE_CXX_FLAGS_RELEASE} -march=native")
set(CMAKE_BUILD_TYPE Debug)

if (CMAKE_COMPILER_IS_GNUCXX)
    add_compile_options(-fno-strict-aliasing)
endif()

include(FindCXX11)
include(FindLTO)

# Find packages.
find_package(Crypto++ REQUIRED)
find_package(PugiXML REQUIRED)
find_package(MySQL)
find_package(Threads)

# Selects LuaJIT if user defines or auto-detected
if(DEFINED USE_LUAJIT AND NOT USE_LUAJIT)
    set(FORCE_LUAJIT ${USE_LUAJIT})
else()
    find_package(LuaJIT)
    set(FORCE_LUAJIT ${LuaJIT_FOUND})
endif()
option(USE_LUAJIT "Use LuaJIT" ${FORCE_LUAJIT})

if(FORCE_LUAJIT)
    if(APPLE)
      set(CMAKE_EXE_LINKER_FLAGS "-pagezero_size 10000 -image_base 100000000")
    endif()
else()
    find_package(Lua REQUIRED)
endif()

set(_Boost_STACKTRACE_BASIC_HEADERS     "boost/stacktrace.hpp")
set(_Boost_STACKTRACE_BACKTRACE_HEADERS "boost/stacktrace.hpp")
set(_Boost_STACKTRACE_ADDR2LINE_HEADERS "boost/stacktrace.hpp")
set(_Boost_STACKTRACE_NOOP_HEADERS      "boost/stacktrace.hpp")
set(Boost_USE_STATIC_LIBS ON)

find_package(Boost 1.65.0 COMPONENTS system filesystem iostreams stacktrace_basic stacktrace_backtrace stacktrace_addr2line stacktrace_noop log log_setup REQUIRED)

add_subdirectory(src)
add_executable(tfs ${tfs_SRC})

include_directories(${MYSQL_INCLUDE_DIR} ${LUA_INCLUDE_DIR} ${Boost_INCLUDE_DIRS} ${PUGIXML_INCLUDE_DIR} ${Crypto++_INCLUDE_DIR})
target_link_libraries(tfs ${MYSQL_CLIENT_LIBS} ${LUA_LIBRARIES} ${Boost_LIBRARIES} ${Boost_LOG_LIBRARY} ${Boost_LOG_SETUP_LIBRARY} ${Boost_FILESYSTEM_LIBRARY} ${PUGIXML_LIBRARIES} ${Crypto++_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT} ${CMAKE_DL_LIBS})

set_target_properties(tfs PROPERTIES COTIRE_CXX_PREFIX_HEADER_INIT "src/otpch.h")
set_target_properties(tfs PROPERTIES COTIRE_ADD_UNITY_BUILD FALSE)
cotire(tfs)
