find_package( KPMcore 3.3 )

set( _partition_defs "" )

if ( KPMcore_FOUND )
    find_package( Qt5 REQUIRED DBus )  # Needed for KPMCore
    find_package( KF5 REQUIRED I18n WidgetsAddons )  # Needed for KPMCore

    include_directories( ${KPMCORE_INCLUDE_DIR} ${CMAKE_SOURCE_DIR}/src/modules/partition )

    if ( KPMcore_VERSION VERSION_GREATER "3.3.0")
        list( APPEND _partition_defs WITH_KPMCORE331API) # kpmcore > 3.3.0 with deprecations
    endif()
    if ( KPMcore_VERSION VERSION_GREATER "3.90")
        list( APPEND _partition_defs WITH_KPMCORE4API) # kpmcore 4 with new API
    endif()

    # The PartitionIterator is a small class, and it's easiest -- but also a
    # gross hack -- to just compile it again from the partition module tree.
    calamares_add_plugin( fsresizer
        TYPE job
        EXPORT_MACRO PLUGINDLLEXPORT_PRO
        SOURCES
            ResizeFSJob.cpp
            ${PROJECT_SOURCE_DIR}/src/modules/partition/core/PartitionIterator.cpp
        LINK_PRIVATE_LIBRARIES
            kpmcore
            calamares
        COMPILE_DEFINITIONS ${_partition_defs}
        SHARED_LIB
    )

    if( ECM_FOUND AND BUILD_TESTING )
        ecm_add_test(
                Tests.cpp
            TEST_NAME
                fsresizertest
            LINK_LIBRARIES
                ${CALAMARES_LIBRARIES}
                calamares
                calamares_job_fsresizer  # From above
                ${YAMLCPP_LIBRARY}
                Qt5::Core
                Qt5::Test
        )
        set_target_properties( fsresizertest PROPERTIES AUTOMOC TRUE )
        target_include_directories( fsresizertest PRIVATE /usr/local/include )
        target_compile_definitions( fsresizertest PRIVATE ${_partition_defs} )
    endif()
else()
    calamares_skip_module( "fsresizer (missing suitable KPMcore)" )
endif()
