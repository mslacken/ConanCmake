cmake_minimum_required(VERSION 3.10)

project(generic_conan_recipe)

# Get the target library name from a CMake variable
set(LIBRARY bzip2 CACHE STRING "Target library name")

# Try to find the library using find_package (optional)
find_package(${LIBRARY} NO_MODULE)

# If not found, try pkg-config
if(NOT ${LIBRARY}_FOUND)
	include(FindPkgConfig)
    pkg_check_modules(${LIBRARY} ${LIBRARY})
    if(NOT ${LIBRARY}_FOUND)
        message(FATAL_ERROR "Could not find ${LIBRARY} library. Please install it or provide appropriate CMake/pkg-config definitions.")
    endif()
endif()

# Replace placeholders in the template with actual library information
file(READ "${CMAKE_CURRENT_BINARY_DIR}/conanfile.py.in" conan_recipe)
string(REPLACE "@LIBRARY_INCLUDE_DIRS@" "${${LIBRARY}_INCLUDE_DIRS}" conan_recipe "${conan_recipe}")
string(REPLACE "@LIBRARY_LIBRARIES@" "${${LIBRARY}_LIBRARIES}" conan_recipe "${conan_recipe}")
string(REPLACE "@LIBRARY_VERSION@" "${${LIBRARY}_VERSION}" conan_recipe "${conan_recipe}")
string(REPLACE "@LIBRARY_NAME@" "${LIBRARY}" conan_recipe "${conan_recipe}")
string(REPLACE "@LD_FLAGS@" "${${LIBRARY}_LDFLAGS}" conan_recipe "${conan_recipe}")

set(OPTIONS "shared")
if(OPTIONS)
	string(REPLACE "@OPTIONS@" "    options = {\n@OPTIONS@" conan_recipe "${conan_recipe}")
	foreach(OPT IN LISTS OPTIONS)
		string(REPLACE "@OPTIONS@" "        \"${OPT}\": [True, False],\n@OPTIONS@" conan_recipe "${conan_recipe}")
	endforeach()
	string(REPLACE "@OPTIONS@" "    }\n    default_options = {\n@OPTIONS@" conan_recipe "${conan_recipe}")
	foreach(OPT IN LISTS OPTIONS)
		string(REPLACE "@OPTIONS@" "        \"${OPT}\": True,\n@OPTIONS@" conan_recipe "${conan_recipe}")
	endforeach()
	string(REPLACE "@OPTIONS@" "    }\n" conan_recipe "${conan_recipe}")
else()
	string(REPLACE "@OPTIONS@" "" conan_recipe "${conan_recipe}")
endif()


file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/${LIBRARY}/conanfile.py" "${conan_recipe}")

#get_cmake_property(_variableNames VARIABLES)
#list (SORT _variableNames)
#foreach (_variableName ${_variableNames})
#    message(STATUS "${_variableName}=${${_variableName}}")
#endforeach()


