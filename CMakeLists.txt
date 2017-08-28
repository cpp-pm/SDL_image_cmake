CMAKE_MINIMUM_REQUIRED(VERSION 3.0)

include("cmake/HunterGate.cmake")
HunterGate(
    URL "https://github.com/ruslo/hunter/archive/v0.19.84.tar.gz"
    SHA1 "9086d4b3c32784effdd6f83f45125b55cf95aae8"
)

PROJECT(SDL_image VERSION 2.0.1)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/")

hunter_add_package(SDL2)
find_package(SDL2 CONFIG REQUIRED)

set(ENABLE_DEFS "")
set(LINK_LIBS SDL2::SDL2)
set(INCLUDE_DIRS "")

option(ENABLE_BMP "ENABLE_BMP" ON)
option(ENABLE_GIF "ENABLE_GIF" ON)
option(ENABLE_JPG "ENABLE_JPG" ON)
option(ENABLE_LBM "ENABLE_LBM" ON)
option(ENABLE_PCX "ENABLE_PCX" ON)
option(ENABLE_PNG "ENABLE_PNG" ON)
option(ENABLE_PNM "ENABLE_PNM" ON)
option(ENABLE_TGA "ENABLE_TGA" ON)
option(ENABLE_TIF "ENABLE_TIF" ON)
option(ENABLE_WEBP "ENABLE_WEBP" OFF) # BROKEN
option(ENABLE_XCF "ENABLE_XCF" ON)
option(ENABLE_XPM "ENABLE_XPM" ON)
option(ENABLE_XV "ENABLE_XV" ON)

if(ENABLE_BMP)
    set(ENABLE_DEFS "${ENABLE_DEFS}" LOAD_BMP)
endif()

if(ENABLE_GIF)
    set(ENABLE_DEFS "${ENABLE_DEFS}" LOAD_GIF)
endif()

if(ENABLE_JPG)
    set(ENABLE_DEFS "${ENABLE_DEFS}" LOAD_JPG)
    hunter_add_package(Jpeg)
    find_package(JPEG CONFIG REQUIRED)
    set(LINK_LIBS "${LINK_LIBS}" JPEG::jpeg)
endif()

if(ENABLE_LBM)
    set(ENABLE_DEFS "${ENABLE_DEFS}" LOAD_LBM)
endif()

if(ENABLE_PCX)
    set(ENABLE_DEFS "${ENABLE_DEFS}" LOAD_PCX)
endif()

if(ENABLE_PNG)
    set(ENABLE_DEFS "${ENABLE_DEFS}" LOAD_PNG)
    hunter_add_package(PNG)
    find_package(PNG CONFIG REQUIRED)
    find_package(PNG REQUIRED)
    set(LINK_LIBS "${LINK_LIBS}" PNG::png)
endif()

if(ENABLE_PNM)
    set(ENABLE_DEFS "${ENABLE_DEFS}" LOAD_PNM)
endif()

if(ENABLE_TGA)
    set(ENABLE_DEFS "${ENABLE_DEFS}" LOAD_TGA)
endif()

if(ENABLE_TIF)
    set(ENABLE_DEFS "${ENABLE_DEFS}" LOAD_TIF)
    hunter_add_package(TIFF)
    find_package(TIFF CONFIG REQUIRED)
    set(LINK_LIBS "${LINK_LIBS}" TIFF::libtiff)
endif()

if(ENABLE_WEBP)
    set(ENABLE_DEFS "${ENABLE_DEFS}" LOAD_WEBP)
    find_package(WebP REQUIRED) # BROKEN, debian doesn't install the cmake config files, so find_package will fail
    set(LINK_LIBS "${LINK_LIBS}" "${WebP_LIBRARIES}")
    set(INCLUDE_DIRS "${INCLUDE_DIRS}" "${WebP_INCLUDE_DIRS}")
endif()

if(ENABLE_XCF)
    set(ENABLE_DEFS "${ENABLE_DEFS}" LOAD_XCF)
endif()

if(ENABLE_XV)
    set(ENABLE_DEFS "${ENABLE_DEFS}" LOAD_XV)
endif()

set(SRC_C_FILES
    IMG.c
    IMG_bmp.c
    IMG_gif.c
    IMG_jpg.c
    IMG_lbm.c
    IMG_pcx.c
    IMG_png.c
    IMG_pnm.c
    IMG_tga.c
    IMG_tif.c
    IMG_webp.c
    IMG_xcf.c
    IMG_xpm.c
    IMG_xv.c
    IMG_xxx.c
)

set(SRC_H_FILES  
    SDL_image.h
    miniz.h
)

set(SRC_FILES ${SRC_C_FILES} ${SRC_H_FILES})

add_library(SDL_image ${SRC_FILES})


target_link_libraries(SDL_image ${LINK_LIBS})
include_directories(${INCLUDE_DIRS})

target_compile_definitions(SDL_image PUBLIC "${ENABLE_DEFS}" SDL_IMAGE_USE_COMMON_BACKEND)

#######################################
#       cmake config-mode stuff       #
#######################################

set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(include_install_dir "include")

set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated/cmake")

set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)

# Note: PROJECT_VERSION is used as a VERSION
write_basic_package_version_file(
    "${version_config}" COMPATIBILITY SameMajorVersion
)

# Note: variable 'targets_export_name' used
configure_file("Config.cmake.in" "${project_config}" @ONLY)


install(
    TARGETS SDL_image
    EXPORT "${targets_export_name}"
    LIBRARY DESTINATION "lib"
    ARCHIVE DESTINATION "lib"
    RUNTIME DESTINATION "bin"
    INCLUDES DESTINATION "${include_install_dir}" "${include_install_dir}/SDL2"
)

install(
    FILES ${SRC_H_FILES}
    DESTINATION "${include_install_dir}/SDL2"
)


install(
    FILES "${project_config}" "${version_config}"
    DESTINATION "${config_install_dir}"
)


install(
    EXPORT "${targets_export_name}"
    NAMESPACE "${namespace}"
    DESTINATION "${config_install_dir}"
)
