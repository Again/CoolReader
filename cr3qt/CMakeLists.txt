FIND_PACKAGE( Qt4 REQUIRED )
INCLUDE(${QT_USE_FILE})

SET(CR3_UIS
  src/aboutdlg.ui        src/bookmarklistdlg.ui  src/mainwindow.ui  src/settings.ui
  src/addbookmarkdlg.ui  src/filepropsdlg.ui     src/recentdlg.ui   src/tocdlg.ui
  src/wolexportdlg.ui    src/exportprogressdlg.ui src/searchdlg.ui
)

SET(CR3_MOC_HDRS
  src/aboutdlg.h        
  src/bookmarklistdlg.h  
  src/mainwindow.h      
  src/settings.h
  src/addbookmarkdlg.h  
  src/cr3widget.h        
  src/filepropsdlg.h    
  src/recentdlg.h   
  src/wolexportdlg.h
  src/exportprogressdlg.h
  src/tocdlg.h
  src/searchdlg.h
)

SET (CR3_SOURCES 
  src/aboutdlg.cpp        
  src/bookmarklistdlg.cpp  
  src/crqtutil.cpp      
  src/main.cpp        
  src/recentdlg.cpp  
  src/tocdlg.cpp
  src/addbookmarkdlg.cpp  
  src/cr3widget.cpp        
  src/filepropsdlg.cpp  
  src/mainwindow.cpp  
  src/settings.cpp
  src/wolexportdlg.cpp
  src/exportprogressdlg.cpp
  src/searchdlg.cpp
)

IF (UNIX)
    ADD_DEFINITIONS(-DCR3_DATA_DIR="${CMAKE_INSTALL_PREFIX}/share/cr3/" 
          -DUSE_FONTCONFIG=1 
    )
ELSE()
    ADD_DEFINITIONS(-DCR3_DATA_DIR=""
          -DUSE_FONTCONFIG=0
    )
ENDIF(UNIX)

#IF (WIN32)
#IF(NOT ${CMAKE_BUILD_TYPE} STREQUALS Debug)
#SET(CMAKE_EXE_LINKER_FLAGS ${CMAKE_EXE_LINKER_FLAGS} /SUBSYSTEM:WINDOWS )
#ENDIF(NOT ${CMAKE_BUILD_TYPE} STREQUALS Debug)
#ENDIF (WIN32)



SET(LANGUAGES ru uk de es pl)
message( "Languages: ${LANGUAGES}" )
SET(LANGUAGE_TS_FILES)
FOREACH(LANGUAGE ${LANGUAGES})
    SET(TS_FILE "${CMAKE_CURRENT_SOURCE_DIR}/src/i18n/cr3_${LANGUAGE}.ts")
    SET(LANGUAGE_TS_FILES ${LANGUAGE_TS_FILES} ${TS_FILE})
    SET_SOURCE_FILES_PROPERTIES(${TS_FILE} PROPERTIES OUTPUT_LOCATION
        "${CMAKE_BINARY_DIR}${CMAKE_DATA_DIR}/i18n")
ENDFOREACH(LANGUAGE ${LANGUAGES})

QT4_ADD_RESOURCES(CR3_RCS src/cr3res.qrc)
QT4_WRAP_UI(CR3_UI_HDRS ${CR3_UIS})
QT4_WRAP_CPP(CR3_MOC_SRCS ${CR3_MOC_HDRS})
IF(LANGUAGE_TS_FILES)
    QT4_CREATE_TRANSLATION(QM_FILES ${CR3_SOURCES} ${CR3_UI_HDRS} ${LANGUAGE_TS_FILES})
ENDIF(LANGUAGE_TS_FILES)

INCLUDE_DIRECTORIES( ${CMAKE_CURRENT_SOURCE_DIR}/src ${CMAKE_BINARY_DIR}/cr3qt )

IF (UNIX)
FIND_PROGRAM(GZIP_TOOL
  NAMES gzip
  PATHS /bin
    /usr/bin
    /usr/local/bin
)

IF(NOT GZIP_TOOL)
  MESSAGE(FATAL_ERROR "Unable to find 'gzip' program")
ENDIF(NOT GZIP_TOOL)

SET (man ${CMAKE_CURRENT_SOURCE_DIR}/data/docs/cr3.1)
SET (man_gz ${CMAKE_BINARY_DIR}/cr3.1.gz)

ADD_CUSTOM_COMMAND(OUTPUT ${man_gz}
   COMMAND ${GZIP_TOOL} -c ${man} > ${man_gz}
   DEPENDS ${man}
   COMMENT "Building ${man_gz}")

SET (CR3_MAN_PAGES ${man_gz})

ELSE()
SET (CR3_MAN_PAGES)

ENDIF(UNIX)


ADD_EXECUTABLE(cr3 ${CR3_MAN_PAGES} ${CR3_SOURCES} ${CR3_RCS} ${CR3_UI_HDRS} ${CR3_MOC_SRCS} ${QM_FILES})

if(WIN32)
if(CMAKE_GENERATOR MATCHES "Visual Studio*")
set_target_properties(cr3 PROPERTIES
LINK_FLAGS_DEBUG "/SUBSYSTEM:CONSOLE")
set_target_properties(cr3 PROPERTIES
RELWITHDEBINFO "/SUBSYSTEM:CONSOLE")
set_target_properties(cr3 PROPERTIES
LINK_FLAGS_RELEASE "/SUBSYSTEM:WINDOWS")
set_target_properties(cr3 PROPERTIES
MINSIZEREL "/SUBSYSTEM:WINDOWS")
endif(CMAKE_GENERATOR MATCHES "Visual Studio*")
endif(WIN32)



IF (UNIX)
    SET (EXTRA_LIBS ${QT_LIBRARIES} fontconfig ${STD_LIBS} )
ELSE()
    SET (EXTRA_LIBS ${QT_LIBRARIES} ${STD_LIBS} )
ENDIF(UNIX)
TARGET_LINK_LIBRARIES(cr3 crengine tinydict ${EXTRA_LIBS})


IF (UNIX)


INSTALL(FILES ${man_gz}
   DESTINATION "share/man/man1"
   PERMISSIONS OWNER_READ GROUP_READ WORLD_READ)

INSTALL( TARGETS cr3 RUNTIME DESTINATION bin )
INSTALL( DIRECTORY data/ DESTINATION share/cr3 
   FILES_MATCHING PATTERN "*.css" )
INSTALL( DIRECTORY data/hyph DESTINATION share/cr3 
   FILES_MATCHING PATTERN "*.pdb" )
INSTALL( DIRECTORY data/hyph DESTINATION share/cr3 
   FILES_MATCHING PATTERN "*.pattern" )
INSTALL( DIRECTORY data/skins DESTINATION share/cr3/skins )
INSTALL( FILES ${QM_FILES} DESTINATION share/cr3/i18n )
INSTALL( FILES src/desktop/cr3.desktop DESTINATION share/applications )
INSTALL( FILES src/desktop/cr3.png DESTINATION share/pixmaps )
INSTALL( FILES src/desktop/cr3.xpm DESTINATION share/pixmaps )

ELSE()

INSTALL( TARGETS cr3 RUNTIME DESTINATION . )
INSTALL( DIRECTORY data/ DESTINATION . 
   FILES_MATCHING PATTERN "*.css" )
INSTALL( DIRECTORY data/hyph DESTINATION . 
   FILES_MATCHING PATTERN "*.pdb" )
INSTALL( DIRECTORY data/hyph DESTINATION . 
   FILES_MATCHING PATTERN "*.pattern" )
INSTALL( DIRECTORY data/skins DESTINATION . )
INSTALL( FILES ${QM_FILES} DESTINATION i18n )

ENDIF(UNIX)
