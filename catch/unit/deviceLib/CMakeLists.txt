# Common Tests - Test independent of all platforms
set(TEST_SRC
    floatMath.cc
    anyAll.cc
    ballot.cc
    clz.cc
    ffs.cc
    funnelshift.cc
    brev.cc
    popc.cc
    ldg.cc
    threadfence_system.cc
    syncthreadsand.cc
    syncthreadscount.cc
    syncthreadsor.cc
)

if(UNIX)
    set(TEST_SRC ${TEST_SRC}
        deviceAllocation.cc)
endif()

# AMD only tests
set(AMD_TEST_SRC
    unsafeAtomicAdd.cc
    vectorTypesDevice.cc
    mbcnt.cc
    bitExtract.cc
    bitInsert.cc
    floatTM.cc
)
set(AMD_ARCH_SPEC_TEST_SRC
    AtomicAdd_Coherent_withunsafeflag.cc
    AtomicAdd_Coherent_withoutflag.cc
    AtomicAdd_Coherent_withnoUnsafeflag.cc
    AtomicAdd_NonCoherent_withoutflag.cc
    AtomicAdd_NonCoherent_withnoUnsafeflag.cc
    AtomicAdd_NonCoherent_withunsafeflag.cc
    BuiltIns_fmax.cc
    BuiltIns_fmin.cc
    BuiltIns_fadd.cc
    unsafeAtomicAdd_RTC.cc
    unsafeAtomicAdd_Coherent_withunsafeflag.cc
    unsafeAtomicAdd_Coherent_withoutflag.cc
    unsafeAtomicAdd_Coherent_withnounsafeflag.cc
    unsafeAtomicAdd_NonCoherent_withoutflag.cc
    unsafeAtomicAdd_NonCoherent_withnounsafeflag.cc
    unsafeAtomicAdd_NonCoherent_withunsafeflag.cc
)

# Note to pass arch use format like -DOFFLOAD_ARCH_STR="--offload-arch=gfx900  --offload-arch=gfx906"
# having space at the start/end of OFFLOAD_ARCH_STR can cause build failures
if(UNIX)
    add_custom_target(kerDevAllocMultCO.code COMMAND ${CMAKE_CXX_COMPILER} --genco ${OFFLOAD_ARCH_STR} ${CMAKE_CURRENT_SOURCE_DIR}/kerDevAllocMultCO.cc -o ${CMAKE_CURRENT_BINARY_DIR}/../../unit/deviceLib/kerDevAllocMultCO.code -I${CMAKE_CURRENT_SOURCE_DIR}/../../../../include/ -I${CMAKE_CURRENT_SOURCE_DIR}/../../include)
    add_custom_target(kerDevWriteMultCO.code COMMAND ${CMAKE_CXX_COMPILER} --genco ${OFFLOAD_ARCH_STR} ${CMAKE_CURRENT_SOURCE_DIR}/kerDevWriteMultCO.cc -o ${CMAKE_CURRENT_BINARY_DIR}/../../unit/deviceLib/kerDevWriteMultCO.code -I${CMAKE_CURRENT_SOURCE_DIR}/../../../../include/ -I${CMAKE_CURRENT_SOURCE_DIR}/../../include)
    add_custom_target(kerDevFreeMultCO.code COMMAND ${CMAKE_CXX_COMPILER} --genco ${OFFLOAD_ARCH_STR} ${CMAKE_CURRENT_SOURCE_DIR}/kerDevFreeMultCO.cc -o ${CMAKE_CURRENT_BINARY_DIR}/../../unit/deviceLib/kerDevFreeMultCO.code -I${CMAKE_CURRENT_SOURCE_DIR}/../../../../include/ -I${CMAKE_CURRENT_SOURCE_DIR}/../../include)
    add_custom_target(kerDevAllocSingleKer.code COMMAND ${CMAKE_CXX_COMPILER} --genco ${OFFLOAD_ARCH_STR} ${CMAKE_CURRENT_SOURCE_DIR}/kerDevAllocSingleKer.cc -o ${CMAKE_CURRENT_BINARY_DIR}/../../unit/deviceLib/kerDevAllocSingleKer.code -I${CMAKE_CURRENT_SOURCE_DIR}/../../../../include/ -I${CMAKE_CURRENT_SOURCE_DIR}/../../include)
endif()

if(HIP_PLATFORM MATCHES "amd")
    if (DEFINED OFFLOAD_ARCH_STR)
      string(FIND ${OFFLOAD_ARCH_STR} "gfx90a" ARCH_CHECK)
    elseif(DEFINED $ENV{HCC_AMDGPU_TARGET})
      string(FIND $ENV{HCC_AMDGPU_TARGET} "gfx90a" ARCH_CHECK)
    else()
      set(ARCH_CHECK -1)
    endif()
    set(TEST_SRC ${TEST_SRC} ${AMD_TEST_SRC})
    set_source_files_properties(floatTM.cc PROPERTIES COMPILE_FLAGS -std=c++17)
if(${ARCH_CHECK} GREATER_EQUAL 0)
	set(TEST_SRC ${TEST_SRC} ${AMD_ARCH_SPEC_TEST_SRC})
    set_source_files_properties(AtomicAdd_Coherent_withunsafeflag.cc PROPERTIES COMPILE_OPTIONS "-munsafe-fp-atomics")
    set_source_files_properties(AtomicAdd_NonCoherent_withunsafeflag.cc PROPERTIES COMPILE_OPTIONS "-munsafe-fp-atomics")
    set_source_files_properties(AtomicAdd_Coherent_withnoUnsafeflag.cc PROPERTIES COMPILE_OPTIONS "-mno-unsafe-fp-atomics")
    set_source_files_properties(AtomicAdd_NonCoherent_withnoUnsafeflag.cc PROPERTIES COMPILE_OPTIONS "-mno-unsafe-fp-atomics")
    set_source_files_properties(unsafeAtomicAdd_Coherent_withunsafeflag.cc PROPERTIES COMPILE_OPTIONS "-munsafe-fp-atomics")
    set_source_files_properties(unsafeAtomicAdd_NonCoherent_withunsafeflag.cc PROPERTIES COMPILE_OPTIONS "-munsafe-fp-atomics")
    set_source_files_properties(unsafeAtomicAdd_Coherent_withnounsafeflag.cc PROPERTIES COMPILE_OPTIONS "-mno-unsafe-fp-atomics")
    set_source_files_properties(unsafeAtomicAdd_NonCoherent_withnounsafeflag.cc PROPERTIES COMPILE_OPTIONS "-mno-unsafe-fp-atomics")
    file(GLOB AtomicAdd_files *AtomicAdd_*_*.cc)
    set_property(SOURCE ${AtomicAdd_files} PROPERTY COMPILE_FLAGS --save-temps)
    file(GLOB unsafeAtomicAdd_files *unsafeAtomicAdd_*_*.cc)
    set_property(SOURCE ${unsafeAtomicAdd_files} PROPERTY COMPILE_FLAGS --save-temps)
endif()
    hip_add_exe_to_target(NAME UnitDeviceTests
                      TEST_SRC ${TEST_SRC}
                      TEST_TARGET_NAME build_tests
                      LINKER_LIBS hiprtc)
elseif(HIP_PLATFORM MATCHES "nvidia")
    hip_add_exe_to_target(NAME UnitDeviceTests
                      TEST_SRC ${TEST_SRC}
                      TEST_TARGET_NAME build_tests
                      COMPILE_OPTIONS --Wno-deprecated-declarations)
endif()

if(UNIX)
    add_dependencies(build_tests kerDevAllocMultCO.code kerDevWriteMultCO.code kerDevFreeMultCO.code kerDevAllocSingleKer.code)
endif()
