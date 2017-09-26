cmake_minimum_required(VERSION 2.8.4)

project(STACKBLOCK CXX)

set(STACKBLOCK_DIR "${STACKBLOCK_SOURCE_DIR}")

include(CheckCXXSourceCompiles)
include(CheckIncludeFileCXX)
find_package(Boost 1.55 COMPONENTS filesystem program_options system serialization REQUIRED)
find_package(BLAS)
find_package(LAPACK)

set(CMAKE_INSTAL_PREFIX "$CMAKE_CURRENT_SOURCE_DIR")

set(NEWMAT_INCLUDE_DIR ${STACKBLOCK_DIR}/newmat10)
add_subdirectory(newmat10)
include_directories(
    ${NEWMAT_INCLUDE_DIR}
    ${STACKBLOCK_DIR}
    ${STACKBLOCK_DIR}/include 
    ${Boost_INCLUDE_DIR} 
)

set(GENETIC_INCLUDE_DIR ${STACKBLOCK_DIR}/genetic)

add_subdirectory(genetic)
include_directories(${GENETIC_INCLUDE_DIR})

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")

set(SRC_NPDM 
    modules/npdm/npdm.C 
    modules/npdm/npdm_driver.C 
    modules/npdm/npdm_patterns.C 
    modules/npdm/npdm_expectations.C 
    modules/npdm/npdm_expectations_engine.C
    modules/npdm/npdm_permutations.C 
    modules/npdm/npdm_spin_adaptation.C 
    modules/npdm/npdm_operator_selector.C 
    modules/npdm/npdm_spin_ops.C
    modules/npdm/onepdm_container.C 
    modules/npdm/twopdm_container.C 
    modules/npdm/threepdm_container.C 
    modules/npdm/fourpdm_container.C
    modules/two_index_ops/two_index_wrappers.C 
    modules/three_index_ops/three_index_wrappers.C 
    modules/four_index_ops/four_index_wrappers.C
    modules/three_index_ops/three_index_compound_ops.C 
    modules/four_index_ops/four_index_compound_ops.C 
    modules/three_index_ops/three_index_op_components.C
    modules/four_index_ops/four_index_op_components.C
    modules/three_index_ops/three_index_ops.C 
    modules/four_index_ops/four_index_ops.C
    modules/three_index_ops/build_3index_ops.C
    modules/four_index_ops/build_4index_ops.C
    modules/npdm/pairpdm_container.C
    modules/npdm/nevpt2_npdm_driver.C 
    modules/npdm/nevpt2_container.C 
    modules/npdm/externalsort.C 
    modules/three_index_ops/build_3index_singleoperator.C 
    modules/npdm/npdm_driver_2.C
)
set(SRC_SPIN_ADAPTED
    saveBlock.C 
    Stackspinblock.C 
    operatorUtilities.C 
    Stack_op_components.C 
    stackguess_wavefunction.C 
    Stackwavefunction.C 
    Stackdensity.C 
    stackopxop.C 
    StackBaseOperator.C 
    StackOperators.C 
    modules/ResponseTheory/sweepResponse.C 
    modules/ResponseTheory/sweepCompress.C 
    pario.C 
    dmrg.C 
    fiedler.C 
    least_squares.C 
    sweep_mps.C 
    set_stackspinblock_components.C 
    linear.C 
    readinput.C 
    timer.C 
    SpinQuantum.C 
    Symmetry.C 
    input.C 
    hfOccGenerator.C 
    Schedule.C 
    orbstring.C 
    slater.C 
    csf.C 
    StateInfo.C  
    screen.C 
    MatrixBLAS.C 
    operatorfunctions.C 
    solver.C 
    davidson.C 
    sweep_params.C 
    sweep.C 
    partial_sweep.C 
    BlockAndDecimate.C 
    initblocks.C 
    rotationmat.C 
    renormalise.C 
    couplingCoeffs.C 
    distribute.C 
    new_anglib.C 
    fci.C 
    IrrepSpace.C 
    modules/generate_blocks/sweep.C 
    SpinSpace.C 
    include/IntegralMatrix.C 
    modules/onepdm/sweep.C 
    modules/onepdm/onepdm.C 
    #modules/twopdm/sweep.C 
    #modules/twopdm/twopdm.C 
    #modules/twopdm/twopdm_2.C
)

if (${Boost_VERSION} GREATER 105500)
    set_property(SOURCE ${SRC_SPIN_ADAPTED} APPEND PROPERTY COMPILE_DEFINITIONS
        "BOOST_1_56_0")
    set_property(SOURCE ${SRC_NPDM} APPEND PROPERTY COMPILE_DEFINITIONS
        "BOOST_1_56_0")
endif()

if (BLAS_FOUND AND LAPACK_FOUND)
    set(LA_LIBRARIES ${BLAS_LIBRARIES} ${LAPACK_LIBRARIES}) 
    set_property(SOURCE ${SRC_SPIN_ADAPTED} APPEND PROPERTY COMPILE_DEFINITIONS
        "BLAS")
    set_property(SOURCE ${SRC_NPDM} APPEND PROPERTY COMPILE_DEFINITIONS
        "BLAS")
    set_property(SOURCE ${SRC_SPIN_ADAPTED} APPEND PROPERTY COMPILE_DEFINITIONS
        "USELAPACK")
endif()

if (${CMAKE_WITH_MPI})
else (${CMAKE_WITH_MPI})
    set_property(SOURCE ${SRC_SPIN_ADAPTED} APPEND PROPERTY COMPILE_DEFINITIONS
        "SERIAL")
    set_property(SOURCE ${SRC_NPDM} APPEND PROPERTY COMPILE_DEFINITIONS
        "SERIAL")
    set_property(SOURCE main.C APPEND PROPERTY COMPILE_DEFINITIONS
        "SERIAL")
endif (${CMAKE_WITH_MPI})

add_library(libnpdm ${SRC_NPDM})
target_link_libraries(libnpdm ${Boost_LIBRARIES} libnewmat)

add_library(libspin_adapted ${SRC_SPIN_ADAPTED})
target_link_libraries(libspin_adapted ${Boost_LIBRARIES} ${LA_LIBRARIES} libnewmat libgenetic libnpdm rt)

add_executable(block.spin_adapted ${STACKBLOCK_DIR}/main.C)
target_link_libraries(block.spin_adapted libspin_adapted ${Boost_LIBRARIES})
