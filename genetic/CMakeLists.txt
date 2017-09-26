set(SRC_GENETIC 
    CrossOver.C
    Evaluate.C
    GAInput.C
    GAOptimize.C
    Generation.C
    Mutation.C 
    RandomGenerator.C 
    ReadIntegral.C
)

if (${CMAKE_WITH_MPI})
else (${CMAKE_WITH_MPI})
    set_property(SOURCE ${SRC_GENETIC} APPEND PROPERTY COMPILE_DEFINITIONS
        "SERIAL")
endif (${CMAKE_WITH_MPI})

add_library(libgenetic ${SRC_GENETIC})

