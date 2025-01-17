cmake_minimum_required(VERSION 3.16)
project(CustomAsioStreams)

set(CMAKE_CXX_STANDARD 20)
MESSAGE(STATUS "Compiler version: ${CMAKE_CXX_COMPILER_VERSION}")

set(FETCHCONTENT_QUIET OFF)
set(CPM_USE_NAMED_CACHE_DIRECTORIES ON)
include(cmake/CPM.cmake)

# set CPM_SOURCE_CACHE env var
CPMAddPackage(
  NAME Boost
  VERSION 1.79.0
  GITHUB_REPOSITORY "boostorg/boost"
  GIT_TAG "boost-1.79.0"
)
CPMAddPackage(
  NAME fmt
  VERSION 8.1.1
  GITHUB_REPOSITORY "fmtlib/fmt"
  GIT_TAG "8.1.1"
)
find_package (Threads REQUIRED)

function(makeExe target sources)
    add_executable(${target} ${sources})
    target_include_directories(${target} PUBLIC src/)
    target_compile_definitions(${target} PRIVATE BOOST_ASIO_NO_DEPRECATED)
    target_link_libraries(${target} PRIVATE Threads::Threads)
    target_link_libraries(${target} PRIVATE Boost::beast)
    target_link_libraries(${target} PRIVATE fmt)
    if(WIN32)
        target_link_libraries(${target} PRIVATE wsock32 ws2_32)
    endif()
endfunction()

makeExe(CAS_coro_context_switching "examples/coro_context_switching.cpp")
makeExe(CAS_coro_basic_io_object "examples/coro_basic_io_object.cpp")
makeExe(CAS_coro_advcd_io_object "examples/coro_advcd_io_object.cpp")
makeExe(CAS_async_calls "examples/async_calls.cpp")
makeExe(CAS_work_guards "examples/work_guards.cpp")

makeExe(CAS_coro_outcome_return "examples/coro_outcome_return.cpp")
target_link_libraries(CAS_coro_outcome_return PRIVATE Boost::outcome)

add_executable(CAS_iterator_concepts "examples/iterator_concepts.cpp")

add_compile_options(--enable-tls --enable-threads) # for thread local storage
IF(CMAKE_BUILD_TYPE MATCHES DEBUG)
    add_compile_options(-fno-omit-frame-pointer -ggdb3 -O0) # clang -fcoroutines-ts -stdlib=libstdc++
    if(NOT WIN32)
        add_compile_options(-fsanitize=address)
        target_link_libraries(${target} asan)
        add_link_options(-fsanitize=address)
    endif()
ENDIF(CMAKE_BUILD_TYPE MATCHES DEBUG)

IF(WIN32)
  add_compile_definitions(_WIN32_WINNT=0x0A00) # https://docs.microsoft.com/en-us/windows/win32/winprog/using-the-windows-headers?redirectedfrom=MSDN#macros-for-conditional-declarations
ENDIF(WIN32)
