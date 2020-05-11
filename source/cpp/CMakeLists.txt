cmake_minimum_required(VERSION 3.5 FATAL_ERROR)

project(CppExtraSamples)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake)
include(CompilerOptions)

set(ZIVID_VERSION 1.8.1)

set(SAMPLES
	Camera/Basic/CaptureViaGenICam
	Camera/Basic/CaptureViaZivid)


set(Halcon_DEPENDING CaptureViaGenICam CaptureViaZivid)


if(DEFINED ENV{HALCONROOT} AND NOT $ENV{HALCONROOT} STREQUAL "")
  message("Using HALCONROOT " $ENV{HALCONROOT})
  STRING(REGEX REPLACE "\\\\" "/" HALCONROOT $ENV{HALCONROOT}) 
endif()
find_package(Halcon)
find_package(HDevEngine)


find_package(Zivid ${ZIVID_VERSION} COMPONENTS Core REQUIRED)

if(WIN32)
    add_custom_target(
        CopyDlls
        COMMAND ${CMAKE_COMMAND} -E copy_directory
                $ENV{ZIVID_INSTALL_FOLDER}/$<$<CONFIG:Debug>:bin_debug>$<$<CONFIG:Release>:bin>/plugins/
                ${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_CFG_INTDIR}/plugins
    )
endif()

message(STATUS "All samples: ${SAMPLES}")

foreach(SAMPLE ${SAMPLES})
    get_filename_component(SAMPLE_NAME ${SAMPLE} NAME)

    add_executable(${SAMPLE_NAME} ${SAMPLE}/${SAMPLE_NAME}.cpp)

    target_link_libraries(${SAMPLE_NAME} Zivid::Core)
						
    if(${SAMPLE_NAME} IN_LIST Halcon_DEPENDING)
        target_link_libraries(
						${SAMPLE_NAME}
						${HALCON_LIBS}
						${HDEVENGINE_LIBS})
		target_include_directories(
			${SAMPLE_NAME}
			PUBLIC 
			${HALCON_INC_DIRS}
			${HDEVENGINE_INC_DIRS}
			include
		)
    endif()

    if(WIN32)
        add_dependencies(${SAMPLE_NAME} CopyDlls)
    endif()
endforeach()



