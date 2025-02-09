# Copyright (c) Martin Schweiger
# Licensed under the MIT License

# SAFESEH linker flag must be turned off because the DX7 libraries don't support it'
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -SAFESEH:NO")

# Sources for all Orbiter executable targets
set(common_src
# General source files
	Astro.cpp
	Camera.cpp
	Config.cpp
	ddeserver.cpp
	Element.cpp
	elevmgr.cpp
	Help.cpp
	Input.cpp
	Keymap.cpp
	LightEmitter.cpp
	Mesh.cpp
	Nav.cpp
	Orbiter.cpp
	PlaybackEd.cpp
	Psys.cpp
	Script.cpp
	Shadow.cpp
	State.cpp
	Vecmat.cpp
	VectorMap.cpp
# Launchpad
	Launchpad.cpp
	LpadTab.cpp
	TabAbout.cpp
	TabExtra.cpp
	TabJoystick.cpp
	TabModule.cpp
	TabParam.cpp
	TabScenario.cpp
	TabVideo.cpp
	TabVisual.cpp
# Dialogs
	CustomControls.cpp
	Dialogs.cpp
	DialogWin.cpp
	DlgCamera.cpp
	DlgCapture.cpp
	DlgFocus.cpp
	DlgFunction.cpp
	DlgHelp.cpp
	DlgInfo.cpp
	DlgMap.cpp
	DlgMenuCfg.cpp
	DlgMgr.cpp
	DlgRecorder.cpp
	DlgTacc.cpp
	DlgVishelper.cpp
	htmlctrl.c
	Select.cpp
# Body classes
	Body.cpp
	BodyIntegrator.cpp
	Celbody.cpp
	Planet.cpp
	Rigidbody.cpp
	Star.cpp
# Vessel classes
	FlightRecorder.cpp
	SuperVessel.cpp
	Vessel.cpp
	Vesselbase.cpp
	Vesselstatus.cpp
# Surface base classes
	Base.cpp
	Baseobj.cpp
# Cockpit classes
	Defpanel.cpp
	hud.cpp
	MenuInfoBar.cpp
	Pane.cpp
	Panel.cpp
	Panel2D.cpp
	VCockpit.cpp
# MFD classes
	Mfd.cpp
	MfdAlign.cpp
	MfdComms.cpp
	MfdDocking.cpp
	MfdHsi.cpp
	MfdLanding.cpp
	MfdMap.cpp
	MfdMap_old.cpp
	MfdOrbit.cpp
	MfdSurface.cpp
	MfdSync.cpp
	MfdTransfer.cpp
	MfdUser.cpp
# API implementations
	CamAPI.cpp
	DrawAPI.cpp
	GraphicsAPI.cpp
	MFDAPI.cpp
	ModuleAPI.cpp
	OrbiterAPI.cpp
# Graphics utils
	D3d7util.cpp
	D3dmath.cpp
	Di7frame.cpp
# Graphics interface base class for GDI clients
	${GDICLIENT_DIR}/GDIClient.cpp
# Utils
	Log.cpp
	Memstat.cpp
	Util.cpp
	ZTreeMgr.cpp
# Resources
	Orbiter.rc
	Orbiter.rc2
	Orbiter.ico
# Files built on the fly
	cryptstring.h
	Disclaimer.txt
	splash_cs.dat
)

# Sources for DX7 built-in graphics version only
set(graphics_src
# DX7 inline graphics
	cloudmgr2.cpp
	CSphereMgr.cpp
	cspheremgr2.cpp
	D3d7enum.cpp
	D3d7frame.cpp
	ExtraRender.cpp
	OGraphics.cpp
	Particle.cpp
	Scene.cpp
	ScreenNote.cpp
	Spherepatch.cpp
	surfmgr2.cpp
	Texture.cpp
	tilelabel.cpp
	TileMgr.cpp
	tilemgr2.cpp
	VBase.cpp
	Vobject.cpp
	VPlanet.cpp
	Vstar.cpp
	Vvessel.cpp
)

set(orbiter_includes
	${CMAKE_CURRENT_SOURCE_DIR}
	${CMAKE_SOURCE_DIR}/Orbitersdk/include
	${CMAKE_SOURCE_DIR}/OVP
	${CMAKE_CURRENT_BINARY_DIR}
	${ZLIB_INCLUDE_DIR}
	${DX7SDK_INCLUDE_DIR}
)

set(orbiter_libs
	Winmm.dll
	comctl32.lib
	UxTheme.lib
	${DX7SDK_LIB_DIR}/dxguid.lib
	${DX7SDK_LIB_DIR}/d3dim.lib
	${DX7SDK_LIB_DIR}/ddraw.lib
	${DX7SDK_LIB_DIR}/dinput.lib
	${HTML_HELP_LIBRARY}
	${ZLIB_LIBRARIES}
	$<TARGET_FILE:Orbitersdk>
	$<TARGET_FILE:DlgCtrl>
)

set(orbiter_depends
	scramble
	fchecksum
	Orbitersdk
	DlgCtrl
)

# Orbiter executable (DX7 inline graphics version)
if(BUILD_ORBITER_DX7)

	add_executable(Orbiter
		${common_src}
		${graphics_src}
	)

	set_target_properties(Orbiter
		PROPERTIES
		LINK_FLAGS "/SUBSYSTEM:WINDOWS"
		RUNTIME_OUTPUT_DIRECTORY ${ORBITER_BINARY_ROOT_DIR}
		COMPILE_DEFINITIONS INLINEGRAPHICS
		ENABLE_EXPORTS 1
		FOLDER Core
	)

	target_include_directories(Orbiter PUBLIC
		${orbiter_includes}
	)
	
	target_link_libraries(Orbiter
		${orbiter_libs}
	)

	add_dependencies(Orbiter
		${orbiter_depends}
	)

	# Installation: Orbiter executable (standalone graphics version)
	install(TARGETS
		Orbiter
		RUNTIME
		DESTINATION ${ORBITER_INSTALL_ROOT_DIR}
	)

	# Installation: Orbiter SDK interface library to SDK folder to be included by addons
	install(FILES $<TARGET_LINKER_FILE:Orbiter>
		DESTINATION ${ORBITER_INSTALL_SDK_DIR}/lib
	)
	
endif()

# Orbiter executable (graphics server version)
if(BUILD_ORBITER_SERVER)

	add_executable(Orbiter_server
		${common_src}
	)

	set_target_properties(Orbiter_server
		PROPERTIES
		LINK_FLAGS "/SUBSYSTEM:WINDOWS"
		RUNTIME_OUTPUT_DIRECTORY ${ORBITER_BINARY_MODULE_DIR}/Server
		RUNTIME_OUTPUT_NAME Orbiter
		COMPILE_DEFINITIONS NOGRAPHICS
		ENABLE_EXPORTS 1
		FOLDER Core
	)

	target_include_directories(Orbiter_server PUBLIC
		${orbiter_includes}
	)

	target_link_libraries(Orbiter_server
		${orbiter_libs}
	)

	add_dependencies(Orbiter_server
		${orbiter_depends}
	)

	# Installation: Orbiter executable (graphics-server versions)
	install(TARGETS
		Orbiter_server
		RUNTIME
		DESTINATION ${ORBITER_INSTALL_MODULE_DIR}/Server
	)

	# If the standard Orbiter executable is not built, we have to use the NG version to export the API links
	if(NOT BUILD_ORBITER_DX7)
		install(FILES $<TARGET_LINKER_FILE:Orbiter_server>
			DESTINATION ${ORBITER_INSTALL_SDK_DIR}/lib
		)
	endif()
	
endif()

add_custom_command(
	OUTPUT cryptstring.h
	DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/cryptstring.h.in
	COMMAND $<TARGET_FILE:scramble> ${CMAKE_CURRENT_SOURCE_DIR}/cryptstring.h.in cryptstring.h
)

add_custom_command(
	OUTPUT Disclaimer.txt
	DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/Disclaimer.txt.in
	COMMAND $<TARGET_FILE:scramble> ${CMAKE_CURRENT_SOURCE_DIR}/Disclaimer.txt.in Disclaimer.txt
)

add_custom_command(
	OUTPUT splash_cs.dat
	DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/Bitmaps/Splash.jpg
	COMMAND $<TARGET_FILE:fchecksum> ${CMAKE_CURRENT_SOURCE_DIR}/Bitmaps/Splash.jpg splash_cs.dat
)
