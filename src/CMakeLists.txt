# Target name
SET(TARGET_LIBRARYNAME OsgOculus)
SET(TARGET_TARGETNAME OsgOculusViewer)

# Target source files
SET(TARGET_SRC
	oculusdevice.cpp
	hmdcamera.cpp
)

# Target headerfiles
SET(TARGET_H
	oculusdevice.h
	hmdcamera.h
)

#####################################################################
# Create library
#####################################################################

ADD_LIBRARY(${TARGET_LIBRARYNAME} ${TARGET_SRC} ${TARGET_H})

#####################################################################
# Linker options
#####################################################################

# Link to open gl libs
TARGET_LINK_LIBRARIES(${TARGET_LIBRARYNAME}  ${OPENGL_LIBRARIES} )

# Link to OpenSceneGraph libs
TARGET_LINK_LIBRARIES(${TARGET_LIBRARYNAME}  ${OPENSCENEGRAPH_LIBRARIES} )

# Link to Oculus libs
TARGET_LINK_LIBRARIES(${TARGET_LIBRARYNAME} 	${OCULUS_SDK_LIBRARIES}	)

# Link to libraries needed for Oculus on Windows
IF(WIN32)
	TARGET_LINK_LIBRARIES(${TARGET_LIBRARYNAME} winmm.lib )
ENDIF(WIN32)

# Link to libraries needed for Oculus on Linux
IF(UNIX)
	TARGET_LINK_LIBRARIES(${TARGET_LIBRARYNAME} pthread udev X11 Xinerama)
ENDIF(UNIX)

#####################################################################
# Create executable
#####################################################################

ADD_EXECUTABLE(${TARGET_TARGETNAME} main.cpp)
TARGET_LINK_LIBRARIES(${TARGET_TARGETNAME} ${TARGET_LIBRARYNAME})

####################################################################
# Create user file for correct environment string
#####################################################################
IF(WIN32)
    IF(MSVC)
		IF (OPENSCENEGRAPH_FOUND)
			# Build path
			SET(USERFILE_ENVIRONMENT_DEBUG "PATH=")
			SET(USERFILE_ENVIRONMENT_RELEASE "PATH=")

			# Add OpenSceneGraph to path
			STRING(REPLACE "\\" "/" OSG_DIR $ENV{OSG_DIR})
			IF (EXISTS "${OSG_DIR}/bin")
				SET(OSG_BIN_DIR "${OSG_DIR}/bin" CACHE PATH "Path where to find OpenSceneGraph binaries")
			ELSE (EXISTS "${OSG_DIR}/bin")
				SET (OSG_BIN_DIR CACHE PATH "Path where to find OpenSceneGraph binaries")
			ENDIF(EXISTS "${OSG_DIR}/bin")
			SET(USERFILE_ENVIRONMENT_DEBUG "${USERFILE_ENVIRONMENT_DEBUG}${OSG_BIN_DIR}")
			SET(USERFILE_ENVIRONMENT_RELEASE "${USERFILE_ENVIRONMENT_RELEASE}${OSG_BIN_DIR}")

			IF (OSG_THIRD_PARTY_DIR)
				# Add third party to path
				SET(USERFILE_ENVIRONMENT_DEBUG "${USERFILE_ENVIRONMENT_DEBUG};${OSG_THIRD_PARTY_DIR}/bin")
				SET(USERFILE_ENVIRONMENT_RELEASE "${USERFILE_ENVIRONMENT_RELEASE};${OSG_THIRD_PARTY_DIR}/bin")
			ELSE (OSG_THIRD_PARTY_DIR)
				MESSAGE("Warning: OpenSceneGraph 3rd Party Directory not found.")
			ENDIF(OSG_THIRD_PARTY_DIR)

			# For visual studio 10
			IF(MSVC10)
				CONFIGURE_FILE(osgoculusviewer.vcxproj.template ${TARGET_TARGETNAME}.vcxproj.user @ONLY)
			ENDIF(MSVC10)

		ENDIF (OPENSCENEGRAPH_FOUND)
    ENDIF(MSVC)
ENDIF(WIN32)
