PROJECT( OsmPlugin )

INCLUDE_DIRECTORIES(
 ${CMAKE_SOURCE_DIR}/src/3rdparty/o5mreader
 ${CMAKE_CURRENT_SOURCE_DIR}
 ${CMAKE_CURRENT_SOURCE_DIR}/writers
 ${CMAKE_CURRENT_SOURCE_DIR}/translators
 ${CMAKE_CURRENT_BINARY_DIR}
 ${Protobuf_INCLUDE_DIRS}
 ${ZLIB_INCLUDE_DIRS}
)

if (Protobuf_FOUND AND Protobuf_PROTOC_EXECUTABLE)
    PROTOBUF_GENERATE_CPP(pbf_srcs pbf_hdrs
        ${CMAKE_SOURCE_DIR}/tools/osm-addresses/pbf/fileformat.proto
        ${CMAKE_SOURCE_DIR}/tools/osm-addresses/pbf/osmformat.proto
    )
    set(EXTRA_LIBS ${Protobuf_LIBRARIES} ${ZLIB_LIBRARIES})
    add_definitions(-DHAVE_PROTOBUF)

	if(MSVC)
	add_definitions(-DPROTOBUF_USE_DLLS)
	endif()
endif()

set( osm_writers_SRCS
        writers/OsmObjectAttributeWriter.cpp
        writers/OsmNodeTagWriter.cpp
        writers/OsmWayTagWriter.cpp
        writers/OsmRelationTagWriter.cpp
        writers/OsmTagWriter.cpp
        writers/OsmTagTagWriter.cpp
   )
set( osm_translators_SRCS
        translators/OsmDocumentTagTranslator.cpp
        translators/O5mWriter.cpp
        translators/OsmConverter.cpp
   )

set( osm_SRCS
  OsmParser.cpp
  OsmPlugin.cpp
  OsmRunner.cpp
  OsmNode.cpp
  OsmWay.cpp
  OsmRelation.cpp
  OsmElementDictionary.cpp
  OsmPbfParser.cpp
  ${pbf_srcs}
)

marble_add_plugin( OsmPlugin ${osm_SRCS} ${osm_writers_SRCS} ${osm_translators_SRCS} )
target_link_libraries(OsmPlugin o5mreader ${EXTRA_LIBS})

find_package(ECM ${REQUIRED_ECM_VERSION} QUIET)
if(NOT ECM_FOUND)
    return()
endif()

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${ECM_MODULE_PATH})

include(KDEInstallDirs)

macro_optional_find_package(KF5 ${REQUIRED_KF5_MIN_VERSION} QUIET COMPONENTS KIO)
if(NOT KF5_FOUND)
    return()
endif()

# register marble part as handler
# install again once part is fixed to load and show the passed file
# install(FILES marble_part_osm.desktop DESTINATION ${KDE_INSTALL_KSERVICES5DIR})

# register thumbnail plugin as handler
install(FILES marble_thumbnail_osm.desktop DESTINATION ${KDE_INSTALL_KSERVICES5DIR})

