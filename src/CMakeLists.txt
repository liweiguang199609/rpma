#
# SPDX-License-Identifier: BSD-3-Clause
# Copyright 2020, Intel Corporation
#

add_cstyle(src
	${CMAKE_CURRENT_SOURCE_DIR}/*.[ch]
	${CMAKE_CURRENT_SOURCE_DIR}/include/*.h)

add_check_whitespace(src
	${CMAKE_CURRENT_SOURCE_DIR}/*.[ch]
	${CMAKE_CURRENT_SOURCE_DIR}/include/*.h)

set(SOURCES
	conn.c
	conn_req.c
	ep.c
	flush.c
	info.c
	librpma.c
	log.c
	log_default.c
	mr.c
	peer.c
	private_data.c
	rpma_err.c
	rpma.c)

add_library(rpma SHARED ${SOURCES})

target_include_directories(rpma PRIVATE . include)

if(TESTS_USE_FAULT_INJECTION)
	target_compile_definitions(rpma PUBLIC FAULT_INJECTION=1)
endif()

target_link_libraries(rpma PRIVATE
	${LIBIBVERBS_LIBRARIES}
	${LIBRDMACM_LIBRARIES}
	-Wl,--version-script=${CMAKE_SOURCE_DIR}/src/librpma.map)

set_target_properties(rpma PROPERTIES
	SOVERSION 0
	PUBLIC_HEADER "include/librpma.h;include/librpma_log.h")

target_compile_definitions(rpma PRIVATE SRCVERSION="${SRCVERSION}")

if(VALGRIND_FOUND)
	target_include_directories(rpma PRIVATE src/valgrind)
endif()

install(TARGETS rpma
	PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
	LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
	RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
