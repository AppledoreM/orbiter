# Copyright (c) Martin Schweiger
# Licensed under the MIT License

add_subdirectory(Date)
add_subdirectory(fchecksum)
add_subdirectory(meshc)
add_subdirectory(Pltex)
add_subdirectory(Shipedit)
add_subdirectory(scramble)
add_subdirectory(texpack)

# We do this as an external project to invoke x64 toolchain
ExternalProject_Add(plsplit
	PREFIX ${CMAKE_CURRENT_BINARY_DIR}/plsplit
	BINARY_DIR ${CMAKE_CURRENT_BINARY_DIR}/plsplit
	SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/plsplit
	INSTALL_DIR ${ORBITER_INSTALL_UTILS_DIR}
	CMAKE_GENERATOR "Visual Studio 16 2019" 
	CMAKE_GENERATOR_PLATFORM "x64"
	CMAKE_ARGS -DCMAKE_INSTALL_PREFIX:PATH=${ORBITER_INSTALL_UTILS_DIR}
)
