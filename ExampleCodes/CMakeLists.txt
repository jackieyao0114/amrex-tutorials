cmake_minimum_required(VERSION 3.14)

project( AMReX-Tutorials
   DESCRIPTION "Tutorials for the AMReX adaptive mesh refinement framework"
   #VERSION  ${AMREX_PKG_VERSION}
   # Check that the line below points to the correct repo
   HOMEPAGE_URL "https://amrex-codes.github.io/amrex/tutorials_html/index.html"
   )

set( CMAKE_MODULE_PATH ${CMAKE_CURRENT_LIST_DIR}/cmake )

#
# Fetch and compile AMReX or link to an existing CMake build install of libamrex.
# If AMReX_DIR is passed then we link to the library found there. Otherwise
# the sources are fetched and compiled from the repo pointed to by AMReX_GIT_REPO
#
set ( AMReX_GIT_REPO "https://github.com/AMReX-Codes/amrex.git/"
    CACHE STRING "The URL identifying the repo to fetchg AMReX from" )

#
# Fetch and compile AMReX or link to an existing CMake build install of libamrex.
# If AMReX_DIR is passed then we link to the library found there. Otherwise
# the sources are fetched and compiled from the repo pointed to by AMReX_GIT_REPO
#
set ( SUNDIALS_GIT_REPO "https://github.com/LLNL/sundials.git"
    CACHE STRING "The URL identifying the repo to fetch SUNDIALS from" )

if( NOT DEFINED AMReX_DIR )
   #
   # Fetch amrex repo
   #
   message(STATUS "Fetching from ${AMReX_GIT_REPO} branch ${AMReX_GIT_BRANCH}" )

   set(AMReX_GIT_BRANCH "development" CACHE STRING "The AMReX branch to checkout")
   set(AMReX_INSTALL  "NO" CACHE INTERNAL "Disable install target for amrex")

   include(FetchContent)
   set(FETCHCONTENT_QUIET OFF)  # Verbose ON

   FetchContent_Declare( amrex
       GIT_REPOSITORY ${AMReX_GIT_REPO}
       GIT_TAG        ${AMReX_GIT_BRANCH}
       )

   if(NOT ${amrex}_POPULATED)
       FetchContent_Populate(amrex)

       list(APPEND CMAKE_MODULE_PATH ${amrex_SOURCE_DIR}/Tools/CMake)

       # Load amrex options here so that they are
       # available to the entire project
       include(AMReXOptions)

if( AMReX_SUNDIALS )
if( NOT DEFINED SUNDIALS_DIR )
   #
   # Fetch sundials repo
   #
   message(STATUS "Fetching from ${SUNDIALS_GIT_REPO} branch ${AMReX_GIT_BRANCH}" )

   set(SUNDIALS_GIT_BRANCH "develop" CACHE STRING "The SUNDIALS branch to checkout")

   include(FetchContent)
   set(FETCHCONTENT_QUIET OFF)  # Verbose ON

   FetchContent_Declare( sundials
       GIT_REPOSITORY ${SUNDIALS_GIT_REPO}
       GIT_TAG        ${SUNDIALS_GIT_BRANCH}
       )

   if(NOT ${sundials}_POPULATED)
       FetchContent_Populate(sundials)
#       set(SUNDIALS_DIR ${sundials_SOURCE_DIR})
   # Set build options for subproject
   set(EXAMPLES_ENABLE_C            OFF                        CACHE INTERNAL "" )
   set(EXAMPLES_ENABLE_CXX          OFF                        CACHE INTERNAL "" )
   set(EXAMPLES_ENABLE_CUDA         OFF                        CACHE INTERNAL "" )
   set(EXAMPLES_ENABLE_F77          OFF                        CACHE INTERNAL "" )
   set(EXAMPLES_ENABLE_F90          OFF                        CACHE INTERNAL "" )
   set(EXAMPLES_ENABLE_F2003        OFF                        CACHE INTERNAL "" )
   set(EXAMPLES_INSTALL             OFF                        CACHE INTERNAL "" )
   set(ENABLE_MPI                   OFF                        CACHE INTERNAL "" )
   set(ENABLE_OPENMP                ${ERF_ENABLE_OMP}                 CACHE INTERNAL "" )
   if (AMReX_GPU_BACKEND STREQUAL CUDA)
     set(ENABLE_CUDA                  ON                      CACHE INTERNAL "" )
     set(SUNDIALS_INDEX_SIZE          32                      CACHE INTERNAL "" )
     set(SUNDIALS_BUILD_PACKAGE_FUSED_KERNELS     ON          CACHE INTERNAL "" )
    else ()
      set(ENABLE_CUDA                  OFF                     CACHE INTERNAL "" )
      if (AMReX_GPU_BACKEND STREQUAL HIP)
    set(ENABLE_HIP                   ON                      CACHE INTERNAL "" )
    set(AMDGPU_TARGETS               ${AMReX_AMD_ARCH}       CACHE INTERNAL "" )
#        set(SUNDIALS_BUILD_PACKAGE_FUSED_KERNELS     ON          CACHE INTERNAL "" )
      endif ()
      if (AMReX_GPU_BACKEND STREQUAL SYCL)
    set(ENABLE_SYCL                  ON                      CACHE INTERNAL "" )
    set(CMAKE_CXX_STANDARD           17                      CACHE INTERNAL "" )
      endif ()
   endif ()
   set(BUILD_ARKODE                 ON                         CACHE INTERNAL "" )
   set(BUILD_KINSOL                 OFF                        CACHE INTERNAL "" )
   set(BUILD_IDA                    OFF                        CACHE INTERNAL "" )
   set(BUILD_IDAS                   OFF                        CACHE INTERNAL "" )
   set(BUILD_CVODES                 OFF                        CACHE INTERNAL "" )
endif ()
   list(APPEND CMAKE_MODULE_PATH ${sundials_SOURCE_DIR})

#  Add  SUNDIALS sources to the build
   add_subdirectory(${sundials_SOURCE_DIR})

   # This is to use the same target name uses by the sundials exported targets
   add_library(SUNDIALS::cvode      ALIAS sundials_cvode_static)
   add_library(SUNDIALS::arkode     ALIAS sundials_arkode_static)
   add_library(SUNDIALS::nvecmanyvector ALIAS sundials_nvecmanyvector_static)

   if (AMReX_GPU_BACKEND STREQUAL CUDA)
      add_library(SUNDIALS::nveccuda ALIAS sundials_nveccuda_static)
   elseif (AMReX_GPU_BACKEND STREQUAL HIP)
      add_library(SUNDIALS::nvechip ALIAS sundials_nvechip_static)
   elseif (AMReX_GPU_BACKEND STREQUAL SYCL)
      add_library(SUNDIALS::nvecsycl ALIAS sundials_nvecsycl_static)
   else ()
     add_library(SUNDIALS::nvecserial ALIAS sundials_nvecserial_static)
   endif ()

   set(SUNDIALS_FOUND TRUE)

else ()
   # This is to use the same target name uses by the sundials exported targets
   add_library(SUNDIALS::cvode      ALIAS sundials_cvode_static)
   add_library(SUNDIALS::arkode     ALIAS sundials_arkode_static)
   add_library(SUNDIALS::nvecmanyvector ALIAS sundials_nvecmanyvector_static)

   if (AMReX_GPU_BACKEND STREQUAL CUDA)
      add_library(SUNDIALS::nveccuda ALIAS sundials_nveccuda_static)
   elseif (AMReX_GPU_BACKEND STREQUAL HIP)
      add_library(SUNDIALS::nvechip ALIAS sundials_nvechip_static)
   elseif (AMReX_GPU_BACKEND STREQUAL SYCL)
      add_library(SUNDIALS::nvecsycl ALIAS sundials_nvecsycl_static)
   else ()
     add_library(SUNDIALS::nvecserial ALIAS sundials_nvecserial_static)
   endif ()

   set(SUNDIALS_FOUND TRUE)
endif ()
endif ()
       if(AMReX_FORTRAN)
          enable_language(Fortran)
       endif ()

       if(AMReX_GPU_BACKEND STREQUAL "CUDA")
          enable_language(CUDA)
          # AMReX 21.06+ supports CUDA_ARCHITECTURES
          if(CMAKE_VERSION VERSION_LESS 3.20)
             include(AMReX_SetupCUDA)
          endif()
       endif()

       # Bring the populated content into the build
       add_subdirectory(${amrex_SOURCE_DIR} ${amrex_BINARY_DIR})
   endif()
else()
   message(STATUS "Using existing AMReX library")

   enable_language(Fortran)
   find_package(AMReX REQUIRED)

   if(AMReX_GPU_BACKEND STREQUAL "CUDA")
      enable_language(CUDA)
      # AMReX 21.06+ supports CUDA_ARCHITECTURES
      if(CMAKE_VERSION VERSION_LESS 3.20)
         include(AMReX_SetupCUDA)
      endif()
   endif()
endif()

#
# List of subdirectories to search for CMakeLists.
# For now, we do not include MUI, SDC, SWFFT
#
set( AMREX_TUTORIALS_SUBDIRS Amr Basic ForkJoin)

if (AMReX_SENSEI)
   list(APPEND AMREX_TUTORIALS_SUBDIRS SENSEI)
endif()

if (AMReX_ASCENT)
   list(APPEND AMREX_TUTORIALS_SUBDIRS Blueprint)
endif ()

if (AMReX_EB)
   list(APPEND AMREX_TUTORIALS_SUBDIRS EB)
endif ()

if (AMReX_LINEAR_SOLVERS)
   list(APPEND AMREX_TUTORIALS_SUBDIRS LinearSolvers)
endif ()

if (AMReX_PARTICLES)
   list(APPEND AMREX_TUTORIALS_SUBDIRS Particles)
endif ()

if (AMReX_FORTRAN_INTERFACES)
   list(APPEND AMREX_TUTORIALS_SUBDIRS FortranInterface)
endif ()

if (NOT (AMReX_GPU_BACKEND STREQUAL "NONE") )
   list(APPEND AMREX_TUTORIALS_SUBDIRS GPU)
endif ()

list(TRANSFORM AMREX_TUTORIALS_SUBDIRS PREPEND "${CMAKE_CURRENT_LIST_DIR}/")

#
# Search for CMakelists.txt in the subdirectories defined in the list above
# and include those that contain one into the build
#
include(SetupTutorials)
foreach (_subdir IN LISTS AMREX_TUTORIALS_SUBDIRS)
   file( GLOB_RECURSE _tests "${_subdir}/*CMakeLists.txt" )
   foreach ( _item  IN LISTS _tests)
      get_filename_component(_dir ${_item} DIRECTORY )
      add_subdirectory(${_dir})
      message("_item :"  ${_item})
   endforeach ()
   message("_subdir :"  ${_subdir})
endforeach ()
