add_subdirectory(unittests)

if(WIN32)
	set(win_SRCS winmain.c dblib.def dbopen.c)
endif()

# TODO add dbopen.c if necessary
add_library(sybdb SHARED
	dblib.c dbutil.c rpc.c bcp.c xact.c dbpivot.c buffering.h
	${win_SRCS}
)
target_compile_definitions(sybdb PUBLIC DLL_EXPORT=1)
add_dependencies(sybdb encodings_h)
target_link_libraries(sybdb tds replacements tdsutils ${lib_NETWORK} ${lib_BASE})

add_library(db-lib STATIC
	dblib.c dbutil.c rpc.c bcp.c xact.c dbpivot.c buffering.h
)
add_dependencies(db-lib encodings_h)
target_link_libraries(db-lib tds replacements tdsutils ${lib_NETWORK} ${lib_BASE})

if(NOT WIN32)
	set_target_properties(sybdb PROPERTIES SOVERSION "5.1.0")
endif()

if(MINGW OR CYGWIN)
	set_target_properties(sybdb PROPERTIES LINK_FLAGS "--static")
endif(MINGW OR CYGWIN)

#generate the .pc file
set(prefix "${CMAKE_INSTALL_PREFIX}")
set(version, CMAKE_PROJECT_VERSION)
string(REPLACE ";" " -l" DBLIBS_PRIVATE "${lib_NETWORK}")
set(libs_private "-l${DBLIBS_PRIVATE}")
configure_file(dblib.pc.in dblib.pc @ONLY) 

INSTALL(TARGETS sybdb db-lib
	PUBLIC_HEADER DESTINATION include
	RUNTIME DESTINATION bin
	LIBRARY DESTINATION lib
	ARCHIVE DESTINATION lib
)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/dblib.pc DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)