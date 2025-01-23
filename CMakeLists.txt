cmake_minimum_required(VERSION 3.10)

project(generic_conan_recipe)

# Get the target library name from a CMake variable
set(LIBRARIES bzip2 CACHE STRING "Target library name")

# create header for default file if it doesn't exists
if(NOT EXISTS default)
  file(WRITE default "[replace_requires]")
endif()

foreach(LIBRARY ${LIBRARIES})
  find_package(${LIBRARY} MODULE)
  if(NOT ${LIBRARY}_FOUND)
  find_package(${LIBRARY}) 

# If not found, try pkg-config
    if(NOT ${LIBRARY}_FOUND)
      include(FindPkgConfig)
        pkg_check_modules(${LIBRARY} ${LIBRARY})
        if(NOT ${LIBRARY}_FOUND)
            message(FATAL_ERROR "Could not find ${LIBRARY} library. Please install it or provide appropriate CMake/pkg-config definitions.")
        endif()
        if(${LIBRARY}_CONFIG)
          include(${LIBRARY}_CONFIG)
        endif()
    endif()
  endif()

  get_cmake_property(_variableNames VARIABLES)
  list (SORT _variableNames)
  foreach (_variableName ${_variableNames})
      message(DEBUG "${_variableName}=${${_variableName}}")
  endforeach()

  set(CLASS_NAME "Class_${LIBRARY}")
  set(LIBRARY_MODULE_NAME "${${LIBRARY}_MODULE_NAME}")
  set(LIBRARY_VERSION "${${LIBRARY}_VERSION}")
  set(LIBRARY_INCLUDE_DIRS "${${LIBRARY}_INCLUDE_DIRS}")
  set(LIBRARY_LIBRARIES "${${LIBRARY}_LIBRARIES}")
  set(LD_FLAGS "${${LIBRARY}_LDFLAGS}")

  set(OPTIONSTR "")
  set(OPTIONS "shared" CACHE STRING "Options for building")
  if(OPTIONS)
    string(APPEND OPTIONSTR "    options = {\n")
    foreach(OPT IN LISTS OPTIONS)
      string(APPEND OPTIONSTR "        \"${OPT}\": [True, False],\n")
    endforeach()
    string(APPEND OPTIONSTR "    }\n    default_options = {\n")
    foreach(OPT IN LISTS OPTIONS)
      string(APPEND OPTIONSTR "        \"${OPT}\": True,\n")
    endforeach()
    string(APPEND OPTIONSTR "    }\n")
  endif()

  configure_file("${CMAKE_CURRENT_BINARY_DIR}/conanfile.py.in" 
    "${CMAKE_CURRENT_BINARY_DIR}/${LIBRARY}/conanfile.py")
  file(APPEND default "${LIBRARY}/*: ${LIBRARY}/${LIBRARY_VERSION}@abuild/obs\n")
endforeach()

