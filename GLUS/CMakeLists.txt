#
# GLUS CMake file
#
# Note: Set OpenGL=ES, OpenGL=ES31 or OpenGL=ES2 for Windows OpenGL ES 3.0, 3.1 or 2.0 simulation.
#       Raspberry Pi and i.MX6 is default OpenGL ES 2.0.
#		Set SoC=iMX6 for i.MX6.
#
# (c) Norbert Nopper
# 

cmake_minimum_required(VERSION 2.6)

project(GLUS)

IF(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
	# Mac OS X

	# Support only desktop
	
    SET(Processor "x64")
    SET(OperatingSystem "Darwin")
    SET(Compiler "GCC")

	set(ENV_DIR ${Processor}/${OperatingSystem}/${Compiler})
	
	include_directories(${GLUS_SOURCE_DIR}/../External/${ENV_DIR}/include)

ELSEIF(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
	# Linux

	IF(${CMAKE_SYSTEM_PROCESSOR} MATCHES "arm")
		# Raspberry Pi and i.MX6
		
		SET(OpenGL "ES2")
		
		add_definitions(-DGLUS_ES2)

		IF(${SoC} MATCHES "iMX6")
			# i.MX6

			# Everything is in /usr/include

		ELSE()
			# Raspberry Pi

			include_directories(/opt/vc/include /opt/vc/include/interface/vcos/pthreads /opt/vc/include/interface/vmcs_host/linux)		

		ENDIF()
		
	ELSE()
		# Support only desktop
	
		SET(Processor "x64")
		SET(OperatingSystem "Linux")
		SET(Compiler "GCC")
		
		set(ENV_DIR ${Processor}/${OperatingSystem}/${Compiler})

		# External directories
		include_directories(${GLUS_SOURCE_DIR}/../External/${ENV_DIR}/include)		
		
	ENDIF()
	
ELSEIF(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
	# Windows
	
	add_definitions(-D_CRT_SECURE_NO_WARNINGS)
	
	IF(${OpenGL} MATCHES "ES31")
		# Desktop OpenGL ES 3.1

		add_definitions(-DGLUS_ES31)
	
		include_directories(C:\\Imagination\\PowerVR_Graphics\\PowerVR_SDK\\SDK_3.4\\Builds\\Include)	

	ELSEIF(${OpenGL} MATCHES "ES2")
		# Desktop OpenGL ES 2.0

		add_definitions(-DGLUS_ES2)
	
		include_directories(C:\\Imagination\\PowerVR_Graphics\\PowerVR_SDK\\SDK_3.4\\Builds\\Include)	

	ELSEIF(${OpenGL} MATCHES "ES")
		# Desktop OpenGL ES 3.0

		add_definitions(-DGLUS_ES)
	
		include_directories(C:\\Imagination\\PowerVR_Graphics\\PowerVR_SDK\\SDK_3.4\\Builds\\Include)	

	ELSE()
		# Desktop

		SET(Processor "x86")
		SET(OperatingSystem "Windows")
		SET(Compiler "MSVC")
		
		set(ENV_DIR ${Processor}/${OperatingSystem}/${Compiler})
		
		include_directories(${GLUS_SOURCE_DIR}/../External/${ENV_DIR}/include)	
	
	ENDIF()
		
ENDIF()

# Files only valid for desktop OpenGL
list(APPEND ONLY_GL_C_FILES		${GLUS_SOURCE_DIR}/src/glus_window_glfw.c
								${GLUS_SOURCE_DIR}/src/glus_programpipeline.c
								${GLUS_SOURCE_DIR}/src/glus_program.c
								${GLUS_SOURCE_DIR}/src/glus_shape_adjacency.c
)

# Files currently not used
list(APPEND NOT_USED_C_FILES	${GLUS_SOURCE_DIR}/src/glus_memory_nodm.c
)

# Source files
file(GLOB C_FILES ${GLUS_SOURCE_DIR}/src/*.c)
file(GLOB ES_C_FILES ${GLUS_SOURCE_DIR}/src/*_es.c)
file(GLOB ES2_C_FILES ${GLUS_SOURCE_DIR}/src/*_es2.c)
file(GLOB ES31_C_FILES ${GLUS_SOURCE_DIR}/src/*_es31.c)
file(GLOB VG_C_FILES ${GLUS_SOURCE_DIR}/src/*_vg.c)

# Header files
file(GLOB_RECURSE H_FILES ${GLUS_SOURCE_DIR}/src/*.h)

list(REMOVE_ITEM C_FILES ${NOT_USED_C_FILES})

IF(${OpenGL} MATCHES "ES")

	list(REMOVE_ITEM C_FILES ${VG_C_FILES})

	IF(${OpenGL} MATCHES "ES31")
		# OpenGL ES 3.1
		
		list(REMOVE_ITEM C_FILES ${ONLY_GL_C_FILES} ${ES2_C_FILES})

	ELSEIF(${OpenGL} MATCHES "ES2")
		# OpenGL ES 2.0
		
		list(REMOVE_ITEM C_FILES ${ONLY_GL_C_FILES} ${ES31_C_FILES})

	ELSE()		
		# OpenGL ES 3.0
		
		list(REMOVE_ITEM C_FILES ${ONLY_GL_C_FILES} ${ES31_C_FILES} ${ES2_C_FILES})
	
	ENDIF()
	
	IF(${CMAKE_SYSTEM_PROCESSOR} MATCHES "arm")
		# Raspberry Pi and i.MX6

		list(REMOVE_ITEM C_FILES ${GLUS_SOURCE_DIR}/src/glus_os_windows_es.c)
		list(REMOVE_ITEM C_FILES ${GLUS_SOURCE_DIR}/src/glus_os_linux_es.c)
		
		IF(${SoC} MATCHES "iMX6")			
			# i.MX6
			
			list(REMOVE_ITEM C_FILES ${GLUS_SOURCE_DIR}/src/glus_os_linux_raspberrypi_es.c)
			
		ELSE()
			# Raspberry Pi
		
			list(REMOVE_ITEM C_FILES ${GLUS_SOURCE_DIR}/src/glus_os_linux_imx6_es.c)
			
		ENDIF()
		
	ELSE()
		# Windows simulation
	
		list(REMOVE_ITEM C_FILES ${GLUS_SOURCE_DIR}/src/glus_os_linux_raspberrypi_es.c)		
		list(REMOVE_ITEM C_FILES ${GLUS_SOURCE_DIR}/src/glus_os_linux_es.c)
		list(REMOVE_ITEM C_FILES ${GLUS_SOURCE_DIR}/src/glus_os_linux_imx6_es.c)
		
	ENDIF()	
	
ELSE()
	# OpenGL 3 and 4
	
	list(REMOVE_ITEM C_FILES ${ES_C_FILES} ${ES2_C_FILES} ${ES31_C_FILES} ${VG_C_FILES})
	
ENDIF()

add_library(GLUS ${C_FILES} ${H_FILES})
