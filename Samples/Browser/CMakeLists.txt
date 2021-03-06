#-------------------------------------------------------------------
# This file is part of the CMake build system for OGRE
#     (Object-oriented Graphics Rendering Engine)
# For the latest info, see http://www.ogre3d.org/
#
# The contents of this file are placed in the public domain. Feel
# free to make use of it in any way you like.
#-------------------------------------------------------------------

# Configure Browser demo build

set (HEADER_FILES
	include/SampleBrowser.h
	${OGRE_SOURCE_DIR}/Samples/Common/include/Sample.h
	${OGRE_SOURCE_DIR}/Samples/Common/include/SampleContext.h
	${OGRE_SOURCE_DIR}/Samples/Common/include/SamplePlugin.h
	${OGRE_SOURCE_DIR}/Samples/Common/include/SdkSample.h
)

if (WINDOWS_STORE OR WINDOWS_PHONE)
	set(SOURCE_FILES src/SampleBrowser_WinRT.cpp)
else()
	set(SOURCE_FILES src/SampleBrowser.cpp)
endif()

# Get the list of configured samples
get_property(OGRE_SAMPLES_LIST GLOBAL PROPERTY "OGRE_SAMPLES_LIST")

if (APPLE_IOS)
  set_source_files_properties(${CMAKE_CURRENT_SOURCE_DIR}/src/SampleBrowser.cpp PROPERTIES COMPILE_FLAGS "-x objective-c++")

  if(OGRE_BUILD_RENDERSYSTEM_GLES AND NOT OGRE_BUILD_RENDERSYSTEM_GLES2)
    list(REMOVE_ITEM OGRE_SAMPLES_LIST Sample_BSP Sample_Compositor Sample_CelShading Sample_CubeMapping Sample_Dot3Bump Sample_Fresnel Sample_Instancing Sample_Lighting Sample_ShaderSystem Sample_ShaderSystemTexturedFog Sample_ShaderSystemMultiLight Sample_VolumeTex Sample_Water)
  endif()

  if(NOT OGRE_BUILD_COMPONENT_TERRAIN AND NOT OGRE_SDK_BUILD)
    list(REMOVE_ITEM OGRE_SAMPLES_LIST Sample_Terrain)
    list(REMOVE_ITEM OGRE_SAMPLES_LIST Sample_EndlessWorld)
  endif()

  if(NOT OGRE_BUILD_COMPONENT_RTSHADERSYSTEM AND NOT OGRE_SDK_BUILD)
    list(REMOVE_ITEM OGRE_SAMPLES_LIST Sample_ShaderSystem)
    list(REMOVE_ITEM OGRE_SAMPLES_LIST Sample_ShaderSystemMultiLight)
    list(REMOVE_ITEM OGRE_SAMPLES_LIST Sample_ShaderSystemTexturedFog)
  endif()
endif ()

if (APPLE AND APPLE_IOS)
  set(HEADER_FILES ${HEADER_FILES}
    include/SampleBrowser_iOS.h
  )
elseif (OGRE_BUILD_PLATFORM_NACL)
  set(HEADER_FILES ${HEADER_FILES}
    include/SampleBrowser_NaCl.h
  )
elseif (WINDOWS_STORE OR WINDOWS_PHONE)
  set(HEADER_FILES ${HEADER_FILES}
    include/SampleBrowser_WinRT.h
    include/InputManagerWinRT.h
  )
elseif (APPLE)
  set(HEADER_FILES ${HEADER_FILES}
    include/SampleBrowser_OSX.h
	${OGRE_SOURCE_DIR}/Samples/Common/misc/SampleBrowser_OSX.icns
  )
elseif (ANDROID)
  set(HEADER_FILES ${HEADER_FILES}
    include/SampleBrowser_Android.h)
endif (APPLE AND APPLE_IOS)


if (OGRE_STATIC)
  # Find all samples headers, and copy them to a single include folder
  # We do this instead of including every sample in the include_directories directive, 
  # because this creates a list that is too long for the MSVC command line sometimes.
  file (GLOB SAMPLES_HEADERS ${OGRE_SOURCE_DIR}/Samples/*/include/*.h)
  foreach(SAMPLE_HEADER_FULLPATH ${SAMPLES_HEADERS})
	get_filename_component(SAMPLE_HEADER_NAME ${SAMPLE_HEADER_FULLPATH} NAME)
	configure_file(${SAMPLE_HEADER_FULLPATH} ${OGRE_BINARY_DIR}/Samples/include/${SAMPLE_HEADER_NAME} COPYONLY)
  endforeach(SAMPLE_HEADER_FULLPATH)
  include_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}/include
	${OGRE_BINARY_DIR}/Samples/include)

  # Fix for static build with MinGW
  if (OGRE_BUILD_RENDERSYSTEM_D3D9 AND OGRE_STATIC)
    include_directories(${DirectX9_INCLUDE_DIR})
  endif() 
  
	ogre_add_component_include_dir(Paging)
	ogre_add_component_include_dir(Terrain)
	ogre_add_component_include_dir(Volume)
	ogre_add_component_include_dir(MeshLodGenerator)
else()
  include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)  
endif()


if (WINDOWS_STORE OR WINDOWS_PHONE)

    # Inspired by http://cmakems.codeplex.com/SourceControl/latest#Tests/VSWinStorePhone/CMakeLists.txt

    # Generate manifest.
    set(EXE_NAME SampleBrowser)
    set(SHORT_NAME ${EXE_NAME})
    set(PACKAGE_GUID "ef31484f-4f23-4b9f-8748-af5482696ae5")
    set(APP_MANIFEST_NAME Package.appxmanifest)
    if(WINDOWS_PHONE AND "${CMAKE_SYSTEM_VERSION}" STREQUAL "8.0")
        set(APP_MANIFEST_NAME WMAppManifest.xml)
    endif()
    configure_file(
        ${OGRE_TEMPLATES_DIR}/Package_vc${MSVC_VERSION}.${CMAKE_SYSTEM_NAME}.appxmanifest.in
        ${CMAKE_CURRENT_BINARY_DIR}/${APP_MANIFEST_NAME}
        @ONLY)

    # Make symlink to Media folder
    if( NOT EXISTS "${CMAKE_CURRENT_BINARY_DIR}/Media")
        # Get OS dependent path to use in `execute_process`
        file(TO_NATIVE_PATH "${CMAKE_CURRENT_BINARY_DIR}/Media" link)
        file(TO_NATIVE_PATH "${CMAKE_SOURCE_DIR}/Samples/Media" target)
        execute_process( COMMAND cmd.exe /c mklink /J ${link} ${target})
    endif()

    # Copy other resources
    file(COPY
        ${OGRE_BINARY_DIR}/inst/bin/debug/resources_d.cfg
        ${OGRE_BINARY_DIR}/inst/bin/debug/plugins_d.cfg
        ${OGRE_BINARY_DIR}/inst/bin/debug/quakemap_d.cfg
        ${OGRE_BINARY_DIR}/inst/bin/debug/samples_d.cfg
        ${OGRE_BINARY_DIR}/inst/bin/debug/tests_d.cfg

        ${OGRE_BINARY_DIR}/inst/bin/release/resources.cfg
        ${OGRE_BINARY_DIR}/inst/bin/release/plugins.cfg
        ${OGRE_BINARY_DIR}/inst/bin/release/quakemap.cfg
        ${OGRE_BINARY_DIR}/inst/bin/release/samples.cfg
        ${OGRE_BINARY_DIR}/inst/bin/release/tests.cfg

        ${CMAKE_SOURCE_DIR}/Samples/Common/misc.winrt/Logo.png
        ${CMAKE_SOURCE_DIR}/Samples/Common/misc.winrt/SmallLogo.png
        ${CMAKE_SOURCE_DIR}/Samples/Common/misc.winrt/StoreLogo.png
        ${CMAKE_SOURCE_DIR}/Samples/Common/misc.winrt/SplashScreen.png

        DESTINATION ${CMAKE_CURRENT_BINARY_DIR}
    )

    # Mark resources as deployment content, so that it would be packaged
    set(CONTENT_FILES ${CONTENT_FILES}
        ${CMAKE_CURRENT_BINARY_DIR}/${APP_MANIFEST_NAME}
        ${CMAKE_CURRENT_BINARY_DIR}/Media
        ${CMAKE_CURRENT_BINARY_DIR}/Logo.png
        ${CMAKE_CURRENT_BINARY_DIR}/SmallLogo.png
        ${CMAKE_CURRENT_BINARY_DIR}/StoreLogo.png
        ${CMAKE_CURRENT_BINARY_DIR}/SplashScreen.png
    )
    set(DEBUG_CONTENT_FILES ${DEBUG_CONTENT_FILES}
        ${CMAKE_CURRENT_BINARY_DIR}/resources_d.cfg
        ${CMAKE_CURRENT_BINARY_DIR}/plugins_d.cfg
        ${CMAKE_CURRENT_BINARY_DIR}/quakemap_d.cfg
        ${CMAKE_CURRENT_BINARY_DIR}/samples_d.cfg
        ${CMAKE_CURRENT_BINARY_DIR}/tests_d.cfg
    )
    set(RELEASE_CONTENT_FILES ${RELEASE_CONTENT_FILES}
        ${CMAKE_CURRENT_BINARY_DIR}/resources.cfg
        ${CMAKE_CURRENT_BINARY_DIR}/plugins.cfg
        ${CMAKE_CURRENT_BINARY_DIR}/quakemap.cfg
        ${CMAKE_CURRENT_BINARY_DIR}/samples.cfg
        ${CMAKE_CURRENT_BINARY_DIR}/tests.cfg
    )
    set_property(SOURCE ${CONTENT_FILES} PROPERTY VS_DEPLOYMENT_CONTENT 1)
    set_property(SOURCE ${DEBUG_CONTENT_FILES} PROPERTY VS_DEPLOYMENT_CONTENT $<CONFIG:Debug>)
    set_property(SOURCE ${RELEASE_CONTENT_FILES} PROPERTY VS_DEPLOYMENT_CONTENT $<OR:$<CONFIG:Release>,$<CONFIG:RelWithDebInfo>,$<CONFIG:MinSizeRel>>)

    set(RESOURCE_FILES ${CONTENT_FILES} ${DEBUG_CONTENT_FILES} ${RELEASE_CONTENT_FILES})
    source_group("Resource Files" FILES ${RESOURCE_FILES})

endif()


if(ANDROID)
    include(AndroidMacros)
    
    SET(ANDROID_MOD_NAME "OgreSampleBrowser")
    SET(JNI_PATH "${CMAKE_SOURCE_DIR}/Samples/Browser/src")
    SET(JNI_SRC_FILES "SampleBrowser.cpp")
    
    SET(NDKOUT "${CMAKE_BINARY_DIR}/SampleBrowserNDK")
    
    
    SET(PKG_NAME "org.ogre.browser")
    # Set this variable to false if no java code will be present (google android:hasCode for more info)
    SET(HAS_CODE "false")
    
    SET(MAIN_ACTIVITY "android.app.NativeActivity")
    SET(HEADERS "")
    SET(SAMPLE_LDLIBS "")
    
    add_static_libs("${OGRE_BINARY_DIR}/lib" "Sample_Water" "Sample_Ocean" "Sample_NewInstancing" "Sample_ShaderSystem" "Sample_DeferredShading" "Sample_EndlessWorld" "Sample_VolumeCSG" "Sample_VolumeTerrain" "Sample_Terrain" "Sample_ShaderSystemMultiLight" "Sample_ShaderSystemTexturedFog")
	add_static_libs("${OGRE_BINARY_DIR}/lib" "Sample_FacialAnimation" "Sample_Fresnel" "Sample_Grass" "Sample_ParticleFX" "Sample_MeshLod" "Sample_SkyBox" "Sample_SkyPlane" "Sample_SkyDome" "Sample_Smoke")
	add_static_libs("${OGRE_BINARY_DIR}/lib" "Sample_CameraTrack" "Sample_CelShading" "Sample_Character" "Sample_Compositor" "Sample_Lighting" "Sample_BezierPatch" "Sample_CubeMapping" "Sample_DynTex")
	add_static_libs("${OGRE_BINARY_DIR}/lib" "Sample_HLMS" "Sample_VolumeTex")

    if(OGRE_BUILD_TESTS)
        set(OGRE_ANDROID_CFLAGS -DSAMPLES_INCLUDE_PLAYPEN)
        set(OGRE_ANDROID_INCLUDES ${OGRE_SOURCE_DIR}/Tests/VisualTests/PlayPen/include)
        add_static_libs("${OGRE_BINARY_DIR}/lib" PlayPenTests)
    endif()

    configure_file("${OGRE_TEMPLATES_DIR}/Android_resources.cfg.in" "${NDKOUT}/assets/resources.cfg" @ONLY)
    configure_file("${OGRE_TEMPLATES_DIR}/samples.cfg.in" "${NDKOUT}/assets/samples.cfg" @ONLY)
    
    file(COPY "${CMAKE_SOURCE_DIR}/Samples/Media/models" DESTINATION "${NDKOUT}/assets")
    file(COPY "${CMAKE_SOURCE_DIR}/Samples/Media/particle" DESTINATION "${NDKOUT}/assets")
    file(COPY "${CMAKE_SOURCE_DIR}/Samples/Media/RTShaderLib" DESTINATION "${NDKOUT}/assets")
    file(COPY "${CMAKE_SOURCE_DIR}/Samples/Media/thumbnails" DESTINATION "${NDKOUT}/assets")
    file(COPY "${CMAKE_SOURCE_DIR}/Samples/Media/packs" DESTINATION "${NDKOUT}/assets")
    file(COPY "${CMAKE_SOURCE_DIR}/Samples/Media/materials" DESTINATION "${NDKOUT}/assets")
	file(COPY "${CMAKE_SOURCE_DIR}/Samples/Media/HLMS" DESTINATION "${NDKOUT}/assets")
    
	file(COPY "${CMAKE_SOURCE_DIR}/SDK/Android/drawable-hdpi" DESTINATION "${NDKOUT}/res")
	file(COPY "${CMAKE_SOURCE_DIR}/SDK/Android/drawable-ldpi" DESTINATION "${NDKOUT}/res")
	file(COPY "${CMAKE_SOURCE_DIR}/SDK/Android/drawable-mdpi" DESTINATION "${NDKOUT}/res")
	file(COPY "${CMAKE_SOURCE_DIR}/SDK/Android/drawable-xhdpi" DESTINATION "${NDKOUT}/res")
    
    file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/dummy.cpp "int x = 0;")
    ogre_add_library_to_folder(Samples SampleBrowserDummy MODULE ${CMAKE_CURRENT_BINARY_DIR}/dummy.cpp)
    target_link_libraries(SampleBrowserDummy ${SAMPLE_LIBRARIES})
    add_dependencies(SampleBrowserDummy ${OGRE_SAMPLES_LIST})
    
    create_android_proj(SampleBrowserDummy)

else()

if (OGRE_BUILD_TESTS)
	add_definitions(-DSAMPLES_INCLUDE_PLAYPEN)
	include_directories(${OGRE_SOURCE_DIR}/Tests/PlayPen/include
	${OGRE_SOURCE_DIR}/Tests/VisualTests/PlayPen/include)
endif()

# Fix for static build with MinGW
if (OGRE_BUILD_RENDERSYSTEM_D3D9 AND OGRE_STATIC)
    link_directories(${DirectX9_LIBRARY_DIR})
endif()

ogre_add_executable(SampleBrowser WIN32 ${HEADER_FILES} ${SOURCE_FILES} ${RESOURCE_FILES})

# Fix for static build with MinGW
if (OGRE_BUILD_RENDERSYSTEM_D3D9 AND OGRE_STATIC)
   target_link_libraries(SampleBrowser ${DirectX9_LIBRARIES})
endif()

target_link_libraries(SampleBrowser ${OGRE_LIBRARIES} ${OGRE_PLUGIN_LIBRARIES} ${SDL2_LIBRARY} ${SAMPLE_LIBRARIES} OgreBites)

# Add samples as dependencies
add_dependencies(SampleBrowser ${OGRE_SAMPLES_LIST})

if (OGRE_STATIC)
	# Link to samples too
	target_link_libraries(SampleBrowser ${OGRE_SAMPLES_LIST})
endif()

if (SAMPLE_DEPENDENCIES)
  add_dependencies(SampleBrowser ${SAMPLE_DEPENDENCIES})
endif ()


# configure SampleBrowser build

ogre_config_common(SampleBrowser)

# append _d for debug builds
if (NOT (APPLE OR WINDOWS_STORE OR WINDOWS_PHONE))
	set_property(TARGET SampleBrowser APPEND PROPERTY DEBUG_POSTFIX "_d")
endif ()

# set install RPATH for Unix systems
if (UNIX AND OGRE_FULL_RPATH)
	set_property(TARGET SampleBrowser APPEND PROPERTY
		INSTALL_RPATH ${CMAKE_INSTALL_PREFIX}/${OGRE_LIB_DIRECTORY})
	set_property(TARGET SampleBrowser PROPERTY INSTALL_RPATH_USE_LINK_PATH TRUE)
endif ()

if (WINDOWS_STORE OR WINDOWS_PHONE)
	set_target_properties(SampleBrowser PROPERTIES VS_WINRT_COMPONENT "true")
endif()

if (APPLE)
	set_source_files_properties(${CMAKE_CURRENT_SOURCE_DIR}/src/SampleBrowser.cpp PROPERTIES COMPILE_FLAGS "-x objective-c++")

	# On OS X, create .app bundle
	set_property(TARGET SampleBrowser PROPERTY MACOSX_BUNDLE TRUE)
	set_property(TARGET SampleBrowser PROPERTY MACOSX_BUNDLE_ICON_FILE SampleBrowser_OSX.icns)
	set_property(TARGET SampleBrowser PROPERTY MACOSX_BUNDLE_INFO_PLIST ${OGRE_SOURCE_DIR}/Samples/Common/misc/Info.plist)
	set(MACOSX_BUNDLE_GUI_IDENTIFIER "org.ogre3d.\${PRODUCT_NAME:rfc1034identifier}")
    set_target_properties(SampleBrowser PROPERTIES XCODE_ATTRIBUTE_ONLY_ACTIVE_ARCH "NO")

  if (APPLE_IOS)
    set_target_properties(SampleBrowser PROPERTIES XCODE_ATTRIBUTE_TARGETED_DEVICE_FAMILY "1,2")
    set(OGRE_SAMPLE_CONTENTS_PATH ${OGRE_BINARY_DIR}/bin/$(CONFIGURATION)/SampleBrowser.app)
    add_custom_command(TARGET SampleBrowser POST_BUILD
      COMMAND ditto ${OGRE_SOURCE_DIR}/Samples/Common/misc/*.png ${OGRE_SAMPLE_CONTENTS_PATH}/
      COMMAND ditto ${OGRE_BINARY_DIR}/bin/*.cfg ${OGRE_SAMPLE_CONTENTS_PATH}/
    )
    if(OGRE_SDK_BUILD)
	    add_custom_command(TARGET SampleBrowser POST_BUILD
          COMMAND mkdir ARGS -p ${OGRE_BINARY_DIR}/lib/$(CONFIGURATION)/
	      COMMAND ditto ${OGRE_BINARY_DIR}/Media ${OGRE_SAMPLE_CONTENTS_PATH}/Media
	    )
    else()
        add_custom_command(TARGET SampleBrowser POST_BUILD
          COMMAND mkdir ARGS -p ${OGRE_BINARY_DIR}/lib/$(CONFIGURATION)/
          COMMAND ditto ${OGRE_SOURCE_DIR}/Samples/Media ${OGRE_SAMPLE_CONTENTS_PATH}/Media
        )
    endif(OGRE_SDK_BUILD)
  else()

    include_directories(
        ${OGRE_SOURCE_DIR}/RenderSystems/GL/include
        ${OGRE_SOURCE_DIR}/RenderSystems/GL/include/OSX
        )

	target_link_libraries(SampleBrowser ${OPENGL_LIBRARIES} ${CoreVideo_LIBRARIES})

   set_target_properties(SampleBrowser PROPERTIES
        LINK_FLAGS "-framework IOKit -framework Cocoa -framework Carbon -framework CoreVideo -framework ApplicationServices")

    # Add the path where the Ogre framework was found, if it has been found somewhere
    if(NOT ${OGRE_FRAMEWORK_PATH} STREQUAL "" AND OGRE_SDK_BUILD)
      set_target_properties(SampleBrowser PROPERTIES
        COMPILE_FLAGS "-F${OGRE_FRAMEWORK_PATH}"
        LINK_FLAGS "-F${OGRE_FRAMEWORK_PATH} -framework IOKit -framework Cocoa -framework Carbon -framework OpenGL -framework CoreVideo"
      )
    endif()
	# also, symlink frameworks so .app is standalone
	# NOTE: $(CONFIGURATION) is not resolvable at CMake run time, it's only 
	# valid at build time (hence parenthesis rather than braces)
	
    set(OGRE_OSX_BUILD_CONFIGURATION "$(PLATFORM_NAME)/$(CONFIGURATION)")
    set(OGRE_BUILT_FRAMEWORK "$(PLATFORM_NAME)/$(CONFIGURATION)")
	set (OGRE_SAMPLE_CONTENTS_PATH 
		${OGRE_BINARY_DIR}/bin/$(CONFIGURATION)/SampleBrowser.app/Contents)
	add_custom_command(TARGET SampleBrowser POST_BUILD
		COMMAND mkdir ARGS -p ${OGRE_SAMPLE_CONTENTS_PATH}/Frameworks
		COMMAND ln ARGS -s -f ${OGRE_BINARY_DIR}/lib/${OGRE_BUILT_FRAMEWORK}/Ogre.framework ${OGRE_SAMPLE_CONTENTS_PATH}/Frameworks/
		COMMAND ln ARGS -s -f ${Cg_LIBRARY_REL}                                             ${OGRE_SAMPLE_CONTENTS_PATH}/Frameworks/
		)
	# now cfg files
	if(OGRE_SDK_BUILD)
	add_custom_command(TARGET SampleBrowser POST_BUILD
		COMMAND mkdir ARGS -p ${OGRE_SAMPLE_CONTENTS_PATH}/Resources
		COMMAND ln ARGS -s -f ${OGRE_BINARY_DIR}/bin/macosx/plugins.cfg   ${OGRE_SAMPLE_CONTENTS_PATH}/Resources/
		COMMAND ln ARGS -s -f ${OGRE_BINARY_DIR}/bin/macosx/resources.cfg ${OGRE_SAMPLE_CONTENTS_PATH}/Resources/
		COMMAND ln ARGS -s -f ${OGRE_BINARY_DIR}/bin/macosx/samples.cfg   ${OGRE_SAMPLE_CONTENTS_PATH}/Resources/
		COMMAND ln ARGS -s -f ${OGRE_BINARY_DIR}/bin/macosx/quakemap.cfg  ${OGRE_SAMPLE_CONTENTS_PATH}/Resources/
		COMMAND ln ARGS -s -f ${OGRE_SOURCE_DIR}/Media                    ${OGRE_SAMPLE_CONTENTS_PATH}/Resources/
		COMMAND ditto ${OGRE_SOURCE_DIR}/Samples/Common/misc/SampleBrowser_OSX.icns ${OGRE_SAMPLE_CONTENTS_PATH}/Resources
		)
	else()
	add_custom_command(TARGET SampleBrowser POST_BUILD
		COMMAND mkdir ARGS -p ${OGRE_SAMPLE_CONTENTS_PATH}/Resources
		COMMAND ln ARGS -s -f ${OGRE_BINARY_DIR}/bin/plugins.cfg 		${OGRE_SAMPLE_CONTENTS_PATH}/Resources/
		COMMAND ln ARGS -s -f ${OGRE_BINARY_DIR}/bin/resources.cfg 		${OGRE_SAMPLE_CONTENTS_PATH}/Resources/
		COMMAND ln ARGS -s -f ${OGRE_BINARY_DIR}/bin/samples.cfg   		${OGRE_SAMPLE_CONTENTS_PATH}/Resources/
		COMMAND ln ARGS -s -f ${OGRE_BINARY_DIR}/bin/quakemap.cfg  		${OGRE_SAMPLE_CONTENTS_PATH}/Resources/
		COMMAND ln ARGS -s -f ${OGRE_SOURCE_DIR}/Samples/Media     		${OGRE_SAMPLE_CONTENTS_PATH}/Resources/
		COMMAND ditto ${OGRE_SOURCE_DIR}/Samples/Common/misc/SampleBrowser_OSX.icns ${OGRE_SAMPLE_CONTENTS_PATH}/Resources
		)
	endif()

	# now plugins
	add_custom_command(TARGET SampleBrowser POST_BUILD
		COMMAND mkdir ARGS -p ${OGRE_SAMPLE_CONTENTS_PATH}/Frameworks
		COMMAND mkdir ARGS -p ${OGRE_SAMPLE_CONTENTS_PATH}/Plugins)

    set(FRAMEWORKS OgreOverlay
		OgreBites
        OgrePaging
        OgreProperty
        OgreRTShaderSystem
        OgreTerrain
        OgreVolume
        Plugin_BSPSceneManager
        Plugin_CgProgramManager
        Plugin_OctreeSceneManager
        Plugin_OctreeZone
        Plugin_ParticleFX
        Plugin_PCZSceneManager
        RenderSystem_GL
        RenderSystem_GL3Plus
    )

    foreach(FWK ${FRAMEWORKS})
	add_custom_command(TARGET SampleBrowser POST_BUILD
        COMMAND ARGS if [ -d ${OGRE_BINARY_DIR}/lib/${OGRE_OSX_BUILD_CONFIGURATION}/${FWK}.framework ]\; then ln -s -f ${OGRE_BINARY_DIR}/lib/${OGRE_OSX_BUILD_CONFIGURATION}/${FWK}.framework ${OGRE_SAMPLE_CONTENTS_PATH}/Frameworks/${FWK}.framework\; fi
		)
    endforeach(FWK)

	# Add links for the sample plugins
    foreach(SAMPLE_PLUGIN ${OGRE_SAMPLES_LIST})
	add_custom_command(TARGET SampleBrowser POST_BUILD
		COMMAND ln ARGS -s -f ${OGRE_BINARY_DIR}/lib/${OGRE_OSX_BUILD_CONFIGURATION}/${SAMPLE_PLUGIN}.dylib
		${OGRE_SAMPLE_CONTENTS_PATH}/Plugins/
		)
    endforeach(SAMPLE_PLUGIN)

	if (OGRE_BUILD_TESTS)
	  add_custom_command(TARGET SampleBrowser POST_BUILD
		COMMAND ln ARGS -s -f ${OGRE_BINARY_DIR}/lib/${OGRE_OSX_BUILD_CONFIGURATION}/PlayPen.dylib 
		${OGRE_SAMPLE_CONTENTS_PATH}/Plugins/
		)
	endif()
  endif()

endif (APPLE)

if (OGRE_INSTALL_SAMPLES)
	ogre_install_target(SampleBrowser "" FALSE)
	if (OGRE_INSTALL_PDB)
		# install debug pdb files
		install(FILES ${OGRE_BINARY_DIR}/bin${OGRE_DEBUG_PATH}/SampleBrowser_d.pdb
			DESTINATION bin${OGRE_DEBUG_PATH} CONFIGURATIONS Debug
			)
		install(FILES ${OGRE_BINARY_DIR}/bin${OGRE_RELWDBG_PATH}/SampleBrowser.pdb
			DESTINATION bin${OGRE_RELWDBG_PATH} CONFIGURATIONS RelWithDebInfo
			)
	endif ()
endif ()
endif(ANDROID)
