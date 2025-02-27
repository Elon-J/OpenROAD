############################################################################
##
## BSD 3-Clause License
##
## Copyright (c) 2021, The Regents of the University of California
## All rights reserved.
##
## Redistribution and use in source and binary forms, with or without
## modification, are permitted provided that the following conditions are met:
##
## * Redistributions of source code must retain the above copyright notice, this
##   list of conditions and the following disclaimer.
##
## * Redistributions in binary form must reproduce the above copyright notice,
##   this list of conditions and the following disclaimer in the documentation
##   and/or other materials provided with the distribution.
##
## * Neither the name of the copyright holder nor the names of its
##   contributors may be used to endorse or promote products derived from
##   this software without specific prior written permission.
##
## THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
## AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
## IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
## ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
## LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
## CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
## SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
## INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
## CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
## ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
## POSSIBILITY OF SUCH DAMAGE.
##
############################################################################

include("openroad")

option(LOAD_PARTITIONERS "Load GPMetis and Chaco" OFF)

if (LOAD_PARTITIONERS)
    set(PATH_PARTITIONERS ${CMAKE_CURRENT_SOURCE_DIR} CACHE PATH "Path to Partitioners Folder")
    message(STATUS "Using partitioners")
endif (LOAD_PARTITIONERS)

swig_lib(NAME      par
         NAMESPACE par
         I_FILE    src/partitionmgr.i
         SCRIPTS   src/partitionmgr.tcl
)

target_sources(par
  PRIVATE
    src/Hypergraph.cpp
    src/MakePartitionMgr.cpp
    src/PartitionMgr.cpp
    src/HypergraphDecomposition.cpp
    src/PartitionNetwork.cpp
    src/PartOptions.cpp
    src/autocluster.cpp
)

messages(
  TARGET par
)

add_subdirectory(src/MLPart)

target_link_libraries(par
    PRIVATE
      MLPart
)

if (LOAD_PARTITIONERS)
  target_compile_definitions(par
    PRIVATE
      PARTITIONERS
  )
  target_include_directories(par
    PRIVATE
      ${PATH_PARTITIONERS}/Partitioners/Chaco/code
      ${PATH_PARTITIONERS}/Partitioners/Metis/include
  )

  find_library(CHACO NAMES Chaco
          PATHS ${PATH_PARTITIONERS}/Partitioners
          PATH_SUFFIXES build/Chaco/code)

  if (NOT CHACO)
          add_subdirectory(${PATH_PARTITIONERS}/Partitioners/Chaco/code Chaco/code)
          set(CHACO Chaco)
  endif()

  find_library(METIS NAMES metis
          PATHS ${PATH_PARTITIONERS}/Partitioners
          PATH_SUFFIXES build/Metis build/Metis/libmetis)

  if (NOT METIS)
          add_subdirectory(${PATH_PARTITIONERS}/Partitioners/Metis Metis)
          set(METIS metis)
  endif()

  target_link_libraries(par
    PUBLIC
      ${CHACO}
      ${METIS}
  )
endif (LOAD_PARTITIONERS)

target_include_directories(par
  PUBLIC
    include
)

target_link_libraries(par
  PUBLIC
    odb
    OpenSTA
    utl
    dbSta
)

unset(LOAD_PARTITIONERS)
