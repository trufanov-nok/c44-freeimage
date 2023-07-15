IF(UNIX)
    set(CPACK_PACKAGE_VERSION ${CMAKE_PROJECT_VERSION})
    set(CPACK_GENERATOR "DEB")
    set(CPACK_PACKAGE_CONTACT "Alexander Trufanov <trufanovan@gmail.com>")

    set(CPACK_PACKAGING_INSTALL_PREFIX ${CMAKE_INSTALL_PREFIX})

    set(CPACK_PACKAGE_NAME "c44-freeimage")

    set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "FreeImage version of c44 encoder.
    Fork of c44 encoder from DjVuLibre that supports most
    of the common image formats with help of FreeImage library.")

    if(CPACK_GENERATOR MATCHES "DEB")
        # Создание DEB пакета
        set(CPACK_DEBIAN_PACKAGE_SECTION "graphics")
        set(CPACK_DEBIAN_PACKAGE_SHLIBDEPS ON)

        include(GNUInstallDirs)

        add_custom_command(
            OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/changelog.gz"
            COMMAND gzip -cn9 "${CMAKE_CURRENT_SOURCE_DIR}/debian/changelog" > "${CMAKE_CURRENT_BINARY_DIR}/changelog.gz"
            WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
            DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/debian/changelog"
            COMMENT "Compressing changelog"
        )

        add_custom_target(changelog ALL DEPENDS "${CMAKE_CURRENT_BINARY_DIR}/changelog.gz")

        install(FILES "${CMAKE_CURRENT_BINARY_DIR}/changelog.gz" DESTINATION "${CMAKE_INSTALL_DOCDIR}")
        install(FILES "${CMAKE_CURRENT_SOURCE_DIR}/debian/copyright" DESTINATION "${CMAKE_INSTALL_DOCDIR}")

        include(CPack)
    endif(CPACK_GENERATOR MATCHES "DEB")


endif(UNIX)
