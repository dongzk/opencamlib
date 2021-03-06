project(OpenCamLib)

cmake_minimum_required(VERSION 2.4)

# some options to set what is built:
option(BUILD_CXX_LIB
  "Build/install the pure c++ libocl library? " ON)

option(BUILD_PY_LIB 
  "Build/install the python ocl library? " ON)

option(BUILD_DOC
  "Build/install the ocl documentation? " ON)

if (NOT BUILD_CXX_LIB)
    MESSAGE(STATUS " Note: will NOT build pure c++ library")
endif(NOT BUILD_CXX_LIB)

if (NOT BUILD_PY_LIB)
    MESSAGE(STATUS " Note: will NOT build python library")
endif(NOT BUILD_PY_LIB)

if (NOT BUILD_DOC)
    MESSAGE(STATUS " Note: will NOT build ocl documentation")
endif(NOT BUILD_DOC)

# figure out the gcc version
INCLUDE(gcc_version.cmake)

#
# Turn compiler warnings up to 11, at least with gcc.  I dont know how to
# do this with other compilers we might support and I'm leaving it up to
# the relevant platform maintainers...
# #include'ing the boost graph-library creates deprecated warnings
# thus we use now use -Wno-deprecated here.
#
if (CMAKE_BUILD_TOOL MATCHES "make")
    if(GCC_4_6)
        MESSAGE(STATUS "setting gcc options: -Wall -Werror -Wno-deprecated -pedantic-errors")
        add_definitions(-Wall  -Wno-deprecated -Werror -pedantic-errors)
    else(GCC_4_6)
        MESSAGE(STATUS "setting gcc options: -Wall  -Wno-deprecated -pedantic-errors")
        add_definitions(-Wall  -Wno-deprecated -pedantic-errors)
    endif(GCC_4_6)
endif (CMAKE_BUILD_TOOL MATCHES "make")

option(BUILD_TYPE
  "Build type: Release=ON/Debug=OFF  " ON)

if (BUILD_TYPE)
    MESSAGE(STATUS " CMAKE_BUILD_TYPE = Release")
    set(CMAKE_BUILD_TYPE Release)
endif(BUILD_TYPE)

if (NOT BUILD_TYPE)
    MESSAGE(STATUS " CMAKE_BUILD_TYPE = Debug")
    set(CMAKE_BUILD_TYPE Debug)
endif(NOT BUILD_TYPE)

#
# this figures out the Python include directories and adds them to the
# header file search path
#
execute_process(
    COMMAND python-config --includes
    COMMAND sed -r "s/-I//g; s/ +/;/g"
    COMMAND tr -d '\n'
    OUTPUT_VARIABLE Python_Includes
)
include_directories(${Python_Includes})

include_directories(${CMAKE_CURRENT_BINARY_DIR})

# find BOOST and boost-python
find_package( Boost COMPONENTS python REQUIRED)
if(Boost_FOUND)
    include_directories(${Boost_INCLUDE_DIRS})
    MESSAGE(STATUS "found Boost: " ${Boost_LIB_VERSION})
    MESSAGE(STATUS "boost-incude dirs are: " ${Boost_INCLUDE_DIRS})
    MESSAGE(STATUS "boost-python lib is: " ${Boost_PYTHON_LIBRARY})
    MESSAGE(STATUS "boost_LIBRARY_DIRS is: " ${Boost_LIBRARY_DIRS})
    MESSAGE(STATUS "Boost_LIBRARIES is: " ${Boost_LIBRARIES})    
endif()

# find OpenMP
find_package( OpenMP REQUIRED )

IF (OPENMP_FOUND)
    MESSAGE(STATUS "found OpenMP, compiling with flags: " ${OpenMP_CXX_FLAGS} )
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
ENDIF(OPENMP_FOUND)


################ create version_string.hpp, http://stackoverflow.com/questions/3780667
# include the output directory, where the version_string.hpp file is generated
include_directories(${CMAKE_CURRENT_BINARY_DIR})
INCLUDE(version_string.cmake)
# now parse the git commit id:
STRING(REGEX REPLACE "([0-9][0-9]).*" "\\1" GIT_MAJOR_VERSION "${GIT_COMMIT_ID}" )
STRING(REGEX REPLACE "[0-9][0-9].([0-9][0-9])-.*" "\\1" GIT_MINOR_VERSION "${GIT_COMMIT_ID}" )
STRING(REGEX REPLACE "[0-9][0-9].[0-9][0-9]-(.*)-.*" "\\1" GIT_PATCH_VERSION "${GIT_COMMIT_ID}" )
SET(MY_VERSION "${GIT_MAJOR_VERSION}.${GIT_MINOR_VERSION}.${GIT_PATCH_VERSION}" CACHE STRING "name")

# this defines the source-files
set(OCL_GEO_SRC
    ${OpenCamLib_SOURCE_DIR}/geo/arc.cpp
    ${OpenCamLib_SOURCE_DIR}/geo/bbox.cpp
    ${OpenCamLib_SOURCE_DIR}/geo/ccpoint.cpp
    ${OpenCamLib_SOURCE_DIR}/geo/clpoint.cpp
    ${OpenCamLib_SOURCE_DIR}/geo/line.cpp
    ${OpenCamLib_SOURCE_DIR}/geo/path.cpp
    ${OpenCamLib_SOURCE_DIR}/geo/point.cpp
    ${OpenCamLib_SOURCE_DIR}/geo/stlreader.cpp
    ${OpenCamLib_SOURCE_DIR}/geo/stlsurf.cpp
    ${OpenCamLib_SOURCE_DIR}/geo/triangle.cpp
)

set(OCL_CUTTER_SRC
    ${OpenCamLib_SOURCE_DIR}/cutters/ballcutter.cpp
    ${OpenCamLib_SOURCE_DIR}/cutters/bullcutter.cpp
    ${OpenCamLib_SOURCE_DIR}/cutters/compositecutter.cpp
    ${OpenCamLib_SOURCE_DIR}/cutters/conecutter.cpp
    ${OpenCamLib_SOURCE_DIR}/cutters/millingcutter.cpp
    ${OpenCamLib_SOURCE_DIR}/cutters/cylcutter.cpp
    ${OpenCamLib_SOURCE_DIR}/cutters/ellipse.cpp
    ${OpenCamLib_SOURCE_DIR}/cutters/ellipseposition.cpp
)

set(OCL_DROPCUTTER_SRC
    ${OpenCamLib_SOURCE_DIR}/dropcutter/batchdropcutter.cpp
    ${OpenCamLib_SOURCE_DIR}/dropcutter/pointdropcutter.cpp
    ${OpenCamLib_SOURCE_DIR}/dropcutter/pathdropcutter.cpp
    ${OpenCamLib_SOURCE_DIR}/dropcutter/adaptivepathdropcutter.cpp
)

set(OCL_ALGO_SRC
    ${OpenCamLib_SOURCE_DIR}/algo/batchpushcutter.cpp
    ${OpenCamLib_SOURCE_DIR}/algo/fiberpushcutter.cpp
    ${OpenCamLib_SOURCE_DIR}/algo/interval.cpp
    ${OpenCamLib_SOURCE_DIR}/algo/fiber.cpp
    ${OpenCamLib_SOURCE_DIR}/algo/waterline.cpp
    ${OpenCamLib_SOURCE_DIR}/algo/adaptivewaterline.cpp
    ${OpenCamLib_SOURCE_DIR}/algo/weave.cpp
    ${OpenCamLib_SOURCE_DIR}/algo/simple_weave.cpp
    ${OpenCamLib_SOURCE_DIR}/algo/smart_weave.cpp
)


set(OCL_COMMON_SRC
    ${OpenCamLib_SOURCE_DIR}/common/numeric.cpp
    ${OpenCamLib_SOURCE_DIR}/common/lineclfilter.cpp
)

set( OCL_INCLUDE_FILES  
    ${OpenCamLib_SOURCE_DIR}/geo/arc.hpp
    ${OpenCamLib_SOURCE_DIR}/geo/bbox.hpp
    ${OpenCamLib_SOURCE_DIR}/geo/ccpoint.hpp
    ${OpenCamLib_SOURCE_DIR}/geo/clpoint.hpp
    ${OpenCamLib_SOURCE_DIR}/geo/line.hpp
    ${OpenCamLib_SOURCE_DIR}/geo/path.hpp
    ${OpenCamLib_SOURCE_DIR}/geo/stlreader.hpp
    ${OpenCamLib_SOURCE_DIR}/geo/stlsurf.hpp
    ${OpenCamLib_SOURCE_DIR}/geo/triangle.hpp
    ${OpenCamLib_SOURCE_DIR}/geo/point.hpp
    
    ${OpenCamLib_SOURCE_DIR}/cutters/ballcutter.hpp
    ${OpenCamLib_SOURCE_DIR}/cutters/bullcutter.hpp
    ${OpenCamLib_SOURCE_DIR}/cutters/compositecutter.hpp
    ${OpenCamLib_SOURCE_DIR}/cutters/conecutter.hpp
    ${OpenCamLib_SOURCE_DIR}/cutters/cylcutter.hpp
    ${OpenCamLib_SOURCE_DIR}/cutters/ellipseposition.hpp
    ${OpenCamLib_SOURCE_DIR}/cutters/millingcutter.hpp
    ${OpenCamLib_SOURCE_DIR}/cutters/ellipse.hpp
    
    ${OpenCamLib_SOURCE_DIR}/dropcutter/adaptivepathdropcutter.hpp
    ${OpenCamLib_SOURCE_DIR}/dropcutter/pathdropcutter.hpp
    ${OpenCamLib_SOURCE_DIR}/dropcutter/batchdropcutter.hpp
    ${OpenCamLib_SOURCE_DIR}/dropcutter/pointdropcutter.hpp
    
    ${OpenCamLib_SOURCE_DIR}/common/brent_zero.hpp
    ${OpenCamLib_SOURCE_DIR}/common/kdnode.hpp
    ${OpenCamLib_SOURCE_DIR}/common/kdtree.hpp
    ${OpenCamLib_SOURCE_DIR}/common/numeric.hpp
    ${OpenCamLib_SOURCE_DIR}/common/lineclfilter.hpp
    ${OpenCamLib_SOURCE_DIR}/common/clfilter.hpp
    ${OpenCamLib_SOURCE_DIR}/common/halfedgediagram.hpp

    
    ${OpenCamLib_SOURCE_DIR}/algo/operation.hpp
    ${OpenCamLib_SOURCE_DIR}/algo/batchpushcutter.hpp
    ${OpenCamLib_SOURCE_DIR}/algo/fiberpushcutter.hpp
    ${OpenCamLib_SOURCE_DIR}/algo/fiber.hpp
    ${OpenCamLib_SOURCE_DIR}/algo/interval.hpp
    ${OpenCamLib_SOURCE_DIR}/algo/waterline.hpp
    ${OpenCamLib_SOURCE_DIR}/algo/adaptivewaterline.hpp
    ${OpenCamLib_SOURCE_DIR}/algo/weave.hpp
    ${OpenCamLib_SOURCE_DIR}/algo/simple_weave.hpp
    ${OpenCamLib_SOURCE_DIR}/algo/smart_weave.hpp
    ${OpenCamLib_SOURCE_DIR}/algo/weave_typedef.hpp
    ${OpenCamLib_SOURCE_DIR}/algo/tsp.hpp
    
    ${CMAKE_CURRENT_BINARY_DIR}/version_string.hpp

)


# this branches into the dirs and compiles stuff there
ADD_SUBDIRECTORY( ${OpenCamLib_SOURCE_DIR}/cutters  )
ADD_SUBDIRECTORY( ${OpenCamLib_SOURCE_DIR}/geo  )
ADD_SUBDIRECTORY( ${OpenCamLib_SOURCE_DIR}/algo  ) 
ADD_SUBDIRECTORY( ${OpenCamLib_SOURCE_DIR}/dropcutter  ) 
ADD_SUBDIRECTORY( ${OpenCamLib_SOURCE_DIR}/common  ) 

# include dirs
include_directories( ${OpenCamLib_SOURCE_DIR}/cutters )
include_directories( ${OpenCamLib_SOURCE_DIR}/geo )
include_directories( ${OpenCamLib_SOURCE_DIR}/algo )
include_directories( ${OpenCamLib_SOURCE_DIR}/dropcutter )
include_directories( ${OpenCamLib_SOURCE_DIR}/common )
include_directories( ${OpenCamLib_SOURCE_DIR} )

#
# this figures out where to install the Python modules
#

execute_process(
    COMMAND python -c "from distutils.sysconfig import get_python_lib; print get_python_lib(0,0,\"/usr/local\")"
    OUTPUT_VARIABLE Python_site_packages
    OUTPUT_STRIP_TRAILING_WHITESPACE
) # on Ubuntu 11.10 this outputs: /usr/local/lib/python2.7/dist-packages


# strip away /usr/local/  because that is what CMAKE_INSTALL_PREFIX is set to
# also, since there is no leading "/", it makes ${Python_site_packages} a relative path.
STRING(REGEX REPLACE "/usr/local/(.*)$" "\\1" Python_site_packages "${Python_site_packages}" )

MESSAGE(STATUS "CMAKE_INSTALL_PREFIX is : " ${CMAKE_INSTALL_PREFIX})
MESSAGE(STATUS "Python libraries will be installed to: " ${Python_site_packages})

if (BUILD_PY_LIB)
    # this makes the ocl Python module
    add_library(
        ocl 
        MODULE
        ocl_cutters.cpp
        ocl_geometry.cpp
        ocl_algo.cpp
        ocl_dropcutter.cpp
        ocl.cpp
        
    )

    MESSAGE(STATUS "linking python binary ocl.so with boost: " ${Boost_PYTHON_LIBRARY})
    target_link_libraries(ocl ocl_common ocl_dropcutter ocl_cutters  ocl_geo ocl_algo ${Boost_LIBRARIES} )
    # 
    # this makes the lib name ocl.so and not libocl.so
    set_target_properties(ocl PROPERTIES PREFIX "") 
    set_target_properties(ocl PROPERTIES VERSION ${MY_VERSION}) 
    
    install(
        TARGETS ocl
        LIBRARY DESTINATION ${Python_site_packages}
    )
    # these are the python helper lib-files such as camvtk.py 
    install(
        DIRECTORY lib/
        DESTINATION ${Python_site_packages}
    #    PATTERN .svn EXCLUDE
    )
endif (BUILD_PY_LIB)


# this is the C++ shared library
if (BUILD_CXX_LIB)
    add_library(
        libocl
        SHARED
        ${OCL_GEO_SRC}
        ${OCL_CUTTER_SRC}
        ${OCL_ALGO_SRC}
        ${OCL_DROPCUTTER_SRC}
        ${OCL_COMMON_SRC}
    )
    set_target_properties(libocl PROPERTIES PREFIX "") 
    set_target_properties(libocl PROPERTIES VERSION ${MY_VERSION}) 
    install(
        TARGETS libocl
        LIBRARY 
        DESTINATION lib/opencamlib
        ARCHIVE DESTINATION lib/opencamlib
        PERMISSIONS OWNER_READ OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE
    )

    # this installs the c++ include headers
    install(
        FILES ${OCL_INCLUDE_FILES}
        DESTINATION include/opencamlib
        PERMISSIONS OWNER_READ GROUP_READ WORLD_READ
    )
endif (BUILD_CXX_LIB)


#
# this installs the examples
#

# install(
#     DIRECTORY ../scripts/
#    DESTINATION share/doc/python-opencam/examples
#    PATTERN .svn EXCLUDE
# )

# the example STL files
# install(
#     DIRECTORY ../stl/
#     DESTINATION share/doc/python-opencam/examples/stl
#     PATTERN .svn EXCLUDE
# )


#
# build & install documentation (if Doxygen is available)
#

if (BUILD_DOC)

FIND_PACKAGE(Doxygen)
IF (DOXYGEN_FOUND)
    MESSAGE(STATUS "Found doxygen. Documentation can be built with 'make doc' ")
    
    # this works around a bug in cmake 2.4 (Ubuntu Hardy)
    execute_process(
        COMMAND mkdir -p doc/html doc/latex
    )

    FIND_PACKAGE(LATEX)
    IF    (NOT LATEX_COMPILER)
        MESSAGE(STATUS "latex command LATEX_COMPILER not found but usually required. You will probably get warnings and user inetraction on doxy run.")
    ENDIF (NOT LATEX_COMPILER)
    IF    (NOT MAKEINDEX_COMPILER)
        MESSAGE(STATUS "makeindex command MAKEINDEX_COMPILER not found but usually required.")
    ENDIF (NOT MAKEINDEX_COMPILER)
    IF    (NOT DVIPS_CONVERTER)
        MESSAGE(STATUS "dvips command DVIPS_CONVERTER not found but usually required.")
    ENDIF (NOT DVIPS_CONVERTER)
    
    #if (EXISTS Doxyfile)
        set(DOXY_CONFIG ${OpenCamLib_SOURCE_DIR}/Doxyfile)
    #endif (EXISTS Doxyfile)

    add_custom_command(
        OUTPUT
            doc/latex/index.tex
            doc/html/index.html
        COMMAND ${DOXYGEN_EXECUTABLE} ${DOXY_CONFIG}
        COMMENT building LaTex & HTML docs
    )

    add_custom_target(
        doc
        DEPENDS doc/latex/index.tex
    )


    IF (EXISTS ${PDFLATEX_COMPILER})
        add_custom_command(
            OUTPUT doc/latex/refman.pdf
            DEPENDS doc/latex/index.tex
            WORKING_DIRECTORY doc/latex
            COMMAND make pdf
            COMMENT building PDF docs
            COMMAND mv refman.pdf ../ocl-manual.pdf
        )

        add_custom_target(
            doc-pdf
            DEPENDS doc/latex/refman.pdf
        )

        add_dependencies(doc doc-pdf)
    ELSE (EXISTS ${PDFLATEX_COMPILER}) 
       message(STATUS "pdflatex compiler not found, PDF docs will not be built")
   ENDIF (EXISTS ${PDFLATEX_COMPILER})


    add_custom_target(
        doc-latex
        DEPENDS doc/latex/index.tex
    )

    #install(
    #    DIRECTORY doc/latex/
    #    DESTINATION share/doc/python-opencam/pdf
    #    FILES_MATCHING PATTERN *.pdf
    #)

    #install(
    #    FILES doc/ocl-manual.pdf
    #    DESTINATION share/doc/python-opencam/pdf
    #)

    #install(
    #    DIRECTORY doc/html
    #    DESTINATION share/doc/python-opencam/
    #)

ENDIF(DOXYGEN_FOUND)
endif (BUILD_DOC)

include(${CMAKE_SOURCE_DIR}/deb/package_details.cmake)

# "make spackage"
add_custom_target(spackage 
              ${CMAKE_COMMAND} 
              -D SRC_DIR:STRING=${CMAKE_SOURCE_DIR} 
              -D MY_VERSION:STRING=${MY_VERSION} 
              -C ${CMAKE_SOURCE_DIR}/deb/package_details.cmake
              -P ${CMAKE_CURRENT_SOURCE_DIR}/deb/DebSourcePPA.cmake 
            ) 
#add_custom_target(spackage-oneiric
#              ${CMAKE_COMMAND} 
#              -D SRC_DIR:STRING=${CMAKE_SOURCE_DIR} 
#              -D MY_VERSION:STRING=${MY_VERSION} 
#              -D MY_DISTRIBUTION_RELEASES:STRING="oneiric"
#              -C ${CMAKE_SOURCE_DIR}/package_details.cmake
#              -P ${CMAKE_CURRENT_SOURCE_DIR}/deb/DebSourcePPA.cmake 
#            )

message(STATUS "type:")
message(STATUS " 'make' for a normal build")
message(STATUS " 'make -j8' to build faster (if you have many cpus)")
message(STATUS " 'make install' to install")
message(STATUS " 'make package' to build a binary deb-packate")
message(STATUS " 'make spackage' to build debian source-packages")
#message(STATUS " 'make spackage-oneiric' to build a debian source-package only for oneiric")
message(STATUS " 'make test' to run the tests")

# "make package"
include(CPack) # this should be last
