#
#   This file is part of Magnum.
#
#   Copyright © 2010, 2011, 2012, 2013, 2014
#             Vladimír Vondruš <mosra@centrum.cz>
#
#   Permission is hereby granted, free of charge, to any person obtaining a
#   copy of this software and associated documentation files (the "Software"),
#   to deal in the Software without restriction, including without limitation
#   the rights to use, copy, modify, merge, publish, distribute, sublicense,
#   and/or sell copies of the Software, and to permit persons to whom the
#   Software is furnished to do so, subject to the following conditions:
#
#   The above copyright notice and this permission notice shall be included
#   in all copies or substantial portions of the Software.
#
#   THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
#   IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
#   FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL
#   THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
#   LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
#   FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER
#   DEALINGS IN THE SOFTWARE.
#

find_package(Magnum REQUIRED Text)
find_package(Freetype REQUIRED)
find_package(HarfBuzz REQUIRED)

include_directories(${FREETYPE_INCLUDE_DIRS} ${HARFBUZZ_INCLUDE_DIRS})

set(HarfBuzzFont_SRCS
    HarfBuzzFont.cpp)

set(HarfBuzzFont_HEADERS
    HarfBuzzFont.h)

add_library(HarfBuzzFontObjects OBJECT
    ${HarfBuzzFont_SRCS}
    ${HarfBuzzFont_HEADERS})
set_target_properties(HarfBuzzFontObjects PROPERTIES COMPILE_FLAGS "${CMAKE_SHARED_LIBRARY_CXX_FLAGS}")

add_plugin(HarfBuzzFont ${MAGNUM_PLUGINS_FONT_DEBUG_INSTALL_DIR} ${MAGNUM_PLUGINS_FONT_RELEASE_INSTALL_DIR}
    HarfBuzzFont.conf
    $<TARGET_OBJECTS:HarfBuzzFontObjects>
    pluginRegistration.cpp)
target_link_libraries(HarfBuzzFont
    ${MAGNUM_LIBRARIES}
    ${MAGNUM_TEXT_LIBRARIES}
    ${FREETYPE_LIBRARIES}
    ${HARFBUZZ_LIBRARIES})

if(CORRADE_TARGET_WINDOWS)
    target_link_libraries(HarfBuzzFont FreeTypeFont)
endif()

install(FILES ${HarfBuzzFont_HEADERS} DESTINATION ${MAGNUM_PLUGINS_INCLUDE_INSTALL_DIR}/HarfBuzzFont)

if(BUILD_GL_TESTS)
    add_library(MagnumHarfBuzzFontTestLib STATIC $<TARGET_OBJECTS:HarfBuzzFontObjects>)
    set_target_properties(MagnumHarfBuzzFontTestLib PROPERTIES DEBUG_POSTFIX "-d")
    target_link_libraries(MagnumHarfBuzzFontTestLib
        ${MAGNUM_LIBRARIES}
        ${MAGNUM_TEXT_LIBRARIES}
        ${FREETYPE_LIBRARIES}
        ${HARFBUZZ_LIBRARIES}
        MagnumFreeTypeFontTestLib)

    # On Windows we need to install first and then run the tests to avoid "DLL
    # not found" hell, thus we need to install this too
    if(CORRADE_TARGET_WINDOWS AND NOT CMAKE_CROSSCOMPILING)
        install(TARGETS MagnumHarfBuzzFontTestLib
            RUNTIME DESTINATION ${MAGNUM_BINARY_INSTALL_DIR}
            LIBRARY DESTINATION ${MAGNUM_LIBRARY_INSTALL_DIR}
            ARCHIVE DESTINATION ${MAGNUM_LIBRARY_INSTALL_DIR})
    endif()

    add_subdirectory(Test)
endif()
