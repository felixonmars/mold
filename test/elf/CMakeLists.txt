option(MOLD_ENABLE_QEMU_TESTS "Enable tests for other targets" OFF)

file(GLOB TESTS
  RELATIVE ${CMAKE_CURRENT_LIST_DIR}
  CONFIGURE_DEPENDS
  "*.sh")

function(add_qemu_test TEST TRIPLE MACHINE)
  if(NOT ${CMAKE_HOST_SYSTEM_PROCESSOR} STREQUAL ${MACHINE})
    add_test(NAME "${MACHINE}-${TEST}"
      COMMAND bash -x ${CMAKE_CURRENT_LIST_DIR}/${TEST}
      WORKING_DIRECTORY ${mold_BINARY_DIR})

    set(ENV
      TEST_CC=${TRIPLE}-gcc
      TEST_CXX=${TRIPLE}-g++
      TEST_GCC=${TRIPLE}-gcc
      TEST_GXX=${TRIPLE}-g++
      OBJDUMP=${TRIPLE}-objdump
      MACHINE=${MACHINE}
      "QEMU=qemu-${MACHINE} -L /usr/${TRIPLE}")
    set_tests_properties("${MACHINE}-${TEST}" PROPERTIES ENVIRONMENT "${ENV}")
  endif()
endfunction()

foreach(TEST IN LISTS TESTS)
  add_test(NAME "${CMAKE_HOST_SYSTEM_PROCESSOR}-${TEST}"
    COMMAND bash -x ${CMAKE_CURRENT_LIST_DIR}/${TEST}
    WORKING_DIRECTORY ${mold_BINARY_DIR})

  if(MOLD_ENABLE_QEMU_TESTS)
    add_qemu_test(${TEST} x86_64-linux-gnu x86_64)
    add_qemu_test(${TEST} i686-linux-gnu i386)
    add_qemu_test(${TEST} aarch64-linux-gnu aarch64)
    add_qemu_test(${TEST} arm-linux-gnueabihf arm)
    add_qemu_test(${TEST} riscv64-linux-gnu riscv64)
  endif()
endforeach()
