cmake_minimum_required(VERSION 3.26)

cmake_policy(SET CMP0091 NEW)
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")

project(cpp-playground LANGUAGES CXX)

add_executable(cpp-playground source/main/src/main.cpp)

if(WIN32)
  target_sources(cpp-playground
    PRIVATE 
      cpp-playground.manifest
  )
endif()

set_target_properties(cpp-playground
  PROPERTIES
    C_STANDARD                 17
    C_EXTENSIONS              OFF
    C_STANDARD_REQUIRED        ON
    CXX_STANDARD               20
    CXX_EXTENSIONS            OFF
    CXX_STANDARD_REQUIRED      ON
    POSITION_INDEPENDENT_CODE  ON
    EXPORT_COMPILE_COMMANDS    ON
)

string(TOUPPER "${CMAKE_BUILD_TYPE}" BUILD_TYPE_ALL_CAPS)

if(WIN32)
  target_compile_definitions(cpp-playground
    PUBLIC
      -D_WIN32_IE=0x0A00          # _WIN32_IE_WIN10
      -D_WIN32_WINNT=0x0A00       # _WIN32_WINNT_WIN10
      -DNTDDI_VERSION=0x0A00000B  # NTDDI_WIN10_CO
      -DWINVER=0x0A00             # _WIN32_WINNT_WIN10
      -DWIN32_LEAN_AND_MEAN
      -DVC_EXTRALEAN
  )
endif()

if("${BUILD_TYPE_ALL_CAPS}" STREQUAL "DEBUG")
  target_compile_definitions(cpp-playground
    PUBLIC
      -DDEBUG
      -D_DEBUG
  )
  if(WIN32)
    target_compile_definitions(cpp-playground
      PUBLIC
        -D_ITERATOR_DEBUG_LEVEL=2
        -D_CRTDBG_MAP_ALLOC
  )
  elseif(LINUX)
    target_compile_definitions(cpp-playground
      PUBLIC
        -D_GLIBCXX_ASSERTIONS
    )
  endif()
elseif("${BUILD_TYPE_ALL_CAPS}" STREQUAL "RELEASE" OR
       "${BUILD_TYPE_ALL_CAPS}" STREQUAL "RELWITHDEBINFO" OR
       "${BUILD_TYPE_ALL_CAPS}" STREQUAL "MINSIZEREL")
  target_compile_definitions(cpp-playground
    PUBLIC
      -DNDEBUG
  )
endif()

if(MSVC)
  target_compile_options(cpp-playground
    PRIVATE
      /Zc:__cplusplus
      /Zc:enumTypes
      /Zc:externConstexpr
      /Zc:inline
      /Zc:preprocessor
      /Zc:templateScope
      /Zc:throwingNew
      /diagnostics:caret
      /utf-8
      /volatile:iso
  )
endif()

if(WIN32)
  target_link_options(cpp-playground
    PRIVATE
      /DEPENDENTLOADFLAG:0x800
  )
  if("${BUILD_TYPE_ALL_CAPS}" STREQUAL "DEBUG")
    target_link_options(cpp-playground
      PRIVATE
        /nodefaultlib:libucrtd.lib
        /defaultlib:ucrtd.lib
    )
  elseif("${BUILD_TYPE_ALL_CAPS}" STREQUAL "RELEASE" OR
         "${BUILD_TYPE_ALL_CAPS}" STREQUAL "RELWITHDEBINFO" OR
         "${BUILD_TYPE_ALL_CAPS}" STREQUAL "MINSIZEREL")
    target_link_options(cpp-playground
      PRIVATE
        /nodefaultlib:libucrt.lib
        /defaultlib:ucrt.lib
    )
  endif()
endif()

if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
  # gcc 5
  list(APPEND COMPILER_WARNINGS
    -Wall
    -Warray-bounds=2
    -Wcast-qual
    -Wconversion
    -Wctor-dtor-privacy
    -Wdeprecated-declarations
    -Wdisabled-optimization
    -Wdouble-promotion
    -Wextra
    -Wformat=2
    -Wlogical-op
    -Wmissing-include-dirs
    -Wnoexcept
    -Wnon-virtual-dtor
    -Wold-style-cast
    -Woverloaded-virtual
    -Wpedantic
    -Wpointer-arith
    -Wredundant-decls
    -Wshadow
    -Wsign-conversion
    -Wsized-deallocation
    -Wtrampolines
    -Wundef
    -Wunused
    -Wunused-parameter
    -Wuseless-cast
    -Wvector-operation-performance
    -Wwrite-strings
    -pedantic-errors
  )

  # gcc 6
  if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 6.1)
    list(APPEND COMPILER_WARNINGS
      -Wduplicated-cond
      -Wmisleading-indentation
      -Wnull-dereference
      -Wshift-overflow=2
    )
  endif()

  # gcc 7
  if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 7.1)
    list(APPEND COMPILER_WARNINGS
      -Wduplicated-branches
      -Wno-psabi
    )
  endif()

  # gcc 9
  if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 9.1)
    list(APPEND COMPILER_WARNINGS
      -Wzero-as-null-pointer-constant
    )
  endif()

  # gcc 10
  if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 10.1)
    list(APPEND COMPILER_WARNINGS
      -Wredundant-tags
    )
  endif()

  # gcc >=10.2
  if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 10.2)
    list(APPEND COMPILER_WARNINGS
      -Wmismatched-tags
    )
  endif()

  # gcc 11
  if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 11.1)
    list(APPEND COMPILER_WARNINGS
      -Wenum-conversion
    )
  endif()

elseif("${CMAKE_CXX_COMPILER_ID}" MATCHES "(Apple)?[Cc]lang")
  list(APPEND COMPILER_WARNINGS
    -Weverything
    -Wno-c++98-compat
    -Wno-c++98-compat-pedantic
    -Wno-padded
  )

  # AppleClang 12 == clang 10
  if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "AppleClang")
    if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 12.0)
      list(APPEND COMPILER_WARNINGS
        -Wno-poison-system-directories
        )
    endif()
  # Regular clang
  else()
    if(CMAKE_CXX_COMPILER_VERSION VERSION_LESS 13.0)
      list(APPEND COMPILER_WARNINGS
        -Wno-return-std-move-in-c++11
      )
    endif()
    if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 9.0)
      list(APPEND COMPILER_WARNINGS
        -Wno-ctad-maybe-unsupported
      )
    endif()
    if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 10.0)
      list(APPEND COMPILER_WARNINGS
        -Wno-poison-system-directories
      )
    endif()
  endif()

else()
  list(APPEND COMPILER_WARNINGS
    /Wall
    /wd4577 # 'noexcept' used with no exception handling mode specified
    /wd4710 # function not inlined despite inline
    /wd4711 # function inlined despite no inline
  )
endif()

list(APPEND CXX_ONLY_COMPILER_WARNINGS
  -Wctor-dtor-privacy
  -Wmismatched-tags
  -Wnoexcept
  -Wnon-virtual-dtor
  -Wold-style-cast
  -Woverloaded-virtual
  -Wredundant-tags
  -Wsized-deallocation
  -Wuseless-cast
  -Wzero-as-null-pointer-constant
)

foreach(COMPILER_WARNING ${COMPILER_WARNINGS})
  if(NOT ${COMPILER_WARNING} IN_LIST CXX_ONLY_COMPILER_WARNINGS)
    list(APPEND C_COMPILER_WARNINGS ${COMPILER_WARNING})
  endif()
endforeach()

string(REPLACE ";" " " C_COMPILER_WARNINGS_STR "${C_COMPILER_WARNINGS}")
string(REPLACE ";" " " CXX_COMPILER_WARNINGS_STR "${COMPILER_WARNINGS}")

get_target_property(cpp-playground_SOURCES cpp-playground SOURCES)

foreach(cpp-playground_SOURCE ${cpp-playground_SOURCES})
  get_source_file_property(${cpp-playground_SOURCE}_LANG ${cpp-playground_SOURCE} LANGUAGE)
  if("${${cpp-playground_SOURCE}_LANG}" STREQUAL "C")
    set_source_files_properties(${cpp-playground_SOURCE} PROPERTIES COMPILE_FLAGS "${C_COMPILER_WARNINGS_STR}")
  elseif("${${cpp-playground_SOURCE}_LANG}" STREQUAL "CXX")
    set_source_files_properties(${cpp-playground_SOURCE} PROPERTIES COMPILE_FLAGS "${CXX_COMPILER_WARNINGS_STR}")
  endif()
endforeach()
