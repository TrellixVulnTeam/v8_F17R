cmake_minimum_required(VERSION 2.8)
project(v8)

SET(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

INCLUDE(ParseArguments)
FUNCTION(JS2C_GEN)

  PARSE_ARGUMENTS(ARG "JS2CROOT;JSFILES;CPP_FILE;OUTPATH;GTYPE" "DEBUG" ${ARGN})
  IF(NOT ARG_JSFILES)
    MESSAGE(SEND_ERROR "Error: JS2C_GEN() called without any js files")
  ENDIF()
  LIST(LENGTH ARG_JS2CROOT JS2CROOT_LENGTH)
  IF(JS2CROOT_LENGTH GREATER 1)
    MESSAGE(SEND_ERROR "Error: JS2C_GEN() called with too many JS2C roots! Only one is allowed")
  ENDIF()
  LIST(LENGTH ARG_OUTPATH OUTPATH_LENGTH)
  IF(OUTPATH_LENGTH GREATER 1)
    MESSAGE(SEND_ERROR "Error: JS2C_GEN() called with too many OUTPATHs! Only one is allowed")
  ENDIF()

  LIST(LENGTH ARG_CPP_FILE CPP_FILE_LENGTH)
  if(NOT CPP_FILE_LENGTH EQUAL 1)
MESSAGE(SEND_ERROR "Error: JS2C_GEN() not given the output CPP_FILE")
  endif()
  set(CPP_FILE ${ARG_CPP_FILE})

  LIST(LENGTH ARG_GTYPE GTYPE_LENGTH)
  if(NOT GTYPE_LENGTH EQUAL 1)
MESSAGE(SEND_ERROR "Error: JS2C_GEN() not given the output GTYPE")
  endif()
  set(GTYPE ${ARG_GTYPE})

  SET(OUTPATH ${CMAKE_CURRENT_BINARY_DIR})
  IF(OUTPATH_LENGTH EQUAL 1)
    SET(OUTPATH ${ARG_OUTPATH})
  ENDIF()

  SET(JS2CROOT ${CMAKE_CURRENT_SOURCE_DIR})
  IF(JS2CROOT_LENGTH EQUAL 1)
      SET(JS2CROOT ${ARG_JS2CROOT})
  ENDIF()

  #The real logic
  SET(SRCS)

  FOREACH(JSFILE ${ARG_JSFILES})
    #Ensure that the file ends with .sh
    STRING(REGEX MATCH "\\.(js|py)$$" JSEND ${JSFILE})
    IF(NOT JSEND)
      MESSAGE(SEND_ERROR "JS file '${JSFILE}' does not end with .js")
    ENDIF()
    GET_FILENAME_COMPONENT(ABS_FILE ${JSFILE} ABSOLUTE)
    LIST(APPEND SRCS "${ABS_FILE}")
  ENDFOREACH()
  string (REPLACE ";" " " SRCS2 "${SRCS}")
#message(WARNING "Sources : ${SRCS}")
  ADD_CUSTOM_COMMAND(
      OUTPUT "${CPP_FILE}"
      COMMAND ${CMAKE_COMMAND} -E make_directory ${OUTPATH}
      COMMAND "python"
      ARGS "${JS2CROOT}/tools/js2c.py"
      ARGS "${CPP_FILE}"
      ARGS "${GTYPE}"
      ARGS "off"
      ARGS ${SRCS}
#      DEPENDS ${SRCS}
      COMMENT "Running js2c, generating: ${CPP_FILE}"
      VERBATIM)

ENDFUNCTION()

set(V8_BASE_SRC "")
LIST(APPEND V8_BASE_SRC
  src/accessors.cc   src/accessors.h   src/allocation.cc   src/allocation.h
  src/api.cc   src/api.h src/apiutils.h   src/arguments.h   src/assembler.cc src/assembler.h
  src/ast.cc src/ast.h   src/atomicops.h   src/atomicops_internals_x86_gcc.cc src/bignum-dtoa.cc
  src/bignum-dtoa.h src/bignum.cc   src/bignum.h   src/bootstrapper.cc src/bootstrapper.h
  src/builtins.cc src/builtins.h   src/bytecodes-irregexp.h   src/cached-powers.cc
  src/cached-powers.h src/char-predicates-inl.h   src/char-predicates.h   src/checks.cc
  src/checks.h src/circular-queue-inl.h src/circular-queue.cc src/circular-queue.h src/code-stubs.cc
  src/code-stubs.h  src/code.h  src/codegen.cc  src/codegen.h  src/compilation-cache.cc
  src/compilation-cache.h  src/compiler.cc  src/compiler.h  src/contexts.cc  src/contexts.h
  src/conversions-inl.h  src/conversions.cc  src/conversions.h  src/counters.cc  src/counters.h
  src/cpu-profiler-inl.h  src/cpu-profiler.cc  src/cpu-profiler.h  src/cpu.h  src/data-flow.cc
  src/data-flow.h  src/date.cc  src/date.h  src/dateparser-inl.h  src/dateparser.cc
  src/dateparser.h  src/debug-agent.cc  src/debug-agent.h  src/debug.cc  src/debug.h
  src/deoptimizer.cc  src/deoptimizer.h  src/disasm.h  src/disassembler.cc  src/disassembler.h
  src/diy-fp.cc  src/diy-fp.h  src/double.h  src/dtoa.cc  src/dtoa.h  src/elements-kind.cc
  src/elements-kind.h  src/elements.cc  src/elements.h  src/execution.cc  src/execution.h
  src/extensions/externalize-string-extension.cc  src/extensions/externalize-string-extension.h
  src/extensions/gc-extension.cc  src/extensions/gc-extension.h
  src/extensions/statistics-extension.cc  src/extensions/statistics-extension.h
  src/factory.cc  src/factory.h  src/fast-dtoa.cc  src/fast-dtoa.h  src/fixed-dtoa.cc
  src/fixed-dtoa.h  src/flag-definitions.h  src/flags.cc
  src/flags.h  src/frames-inl.h  src/frames.cc  src/frames.h  src/full-codegen.cc
  src/full-codegen.h  src/func-name-inferrer.cc  src/func-name-inferrer.h  src/gdb-jit.cc
  src/gdb-jit.h  src/global-handles.cc  src/global-handles.h  src/globals.h
  src/handles-inl.h  src/handles.cc  src/handles.h  src/hashmap.h  src/heap-inl.h
  src/heap-profiler.cc  src/heap-profiler.h  src/heap.cc  src/heap.h  src/hydrogen-instructions.cc
  src/hydrogen-instructions.h  src/hydrogen.cc  src/hydrogen.h  src/ic-inl.h  src/ic.cc  src/ic.h
  src/incremental-marking.cc  src/incremental-marking.h  src/inspector.cc  src/inspector.h
  src/interface.cc  src/interface.h  src/interpreter-irregexp.cc  src/interpreter-irregexp.h
  src/isolate.cc  src/isolate.h  src/json-parser.h  src/json-stringifier.h  src/jsregexp.cc
  src/jsregexp.h  src/lazy-instance.h  src/list-inl.h  src/list.h  src/lithium-allocator-inl.h
  src/lithium-allocator.cc  src/lithium-allocator.h  src/lithium.cc  src/lithium.h  src/liveedit.cc
  src/liveedit.h  src/liveobjectlist-inl.h  src/liveobjectlist.cc  src/liveobjectlist.h
  src/log-inl.h  src/log-utils.cc  src/log-utils.h  src/log.cc  src/log.h
  src/macro-assembler.h  src/mark-compact.cc  src/mark-compact.h  src/messages.cc  src/messages.h
  src/natives.h  src/objects-debug.cc  src/objects-inl.h  src/objects-printer.cc
  src/objects-visiting.cc  src/objects-visiting.h  src/objects.cc  src/objects.h  src/once.cc  src/once.h
  src/optimizing-compiler-thread.h  src/optimizing-compiler-thread.cc  src/parser.cc  src/parser.h
  src/platform-posix.h  src/platform-tls-mac.h  src/platform-tls-win32.h  src/platform-tls.h  src/platform.h
  src/preparse-data-format.h  src/preparse-data.cc  src/preparse-data.h  src/preparser.cc  src/preparser.h
  src/prettyprinter.cc  src/prettyprinter.h  src/profile-generator-inl.h  src/profile-generator.cc
  src/profile-generator.h  src/property-details.h  src/property.cc  src/property.h
  src/regexp-macro-assembler-irregexp-inl.h  src/regexp-macro-assembler-irregexp.cc
  src/regexp-macro-assembler-irregexp.h  src/regexp-macro-assembler-tracer.cc
  src/regexp-macro-assembler-tracer.h  src/regexp-macro-assembler.cc  src/regexp-macro-assembler.h
  src/regexp-stack.cc  src/regexp-stack.h  src/rewriter.cc  src/rewriter.h  src/runtime-profiler.cc
  src/runtime-profiler.h  src/runtime.cc  src/runtime.h  src/safepoint-table.cc  src/safepoint-table.h
  src/scanner-character-streams.cc  src/scanner-character-streams.h  src/scanner.cc  src/scanner.h
  src/scopeinfo.cc  src/scopeinfo.h  src/scopes.cc  src/scopes.h  src/serialize.cc
  src/serialize.h  src/small-pointer-list.h  src/smart-pointers.h  src/snapshot-common.cc
  src/snapshot.h  src/spaces-inl.h  src/spaces.cc  src/spaces.h  src/store-buffer-inl.h
  src/store-buffer.cc  src/store-buffer.h  src/string-search.cc  src/string-search.h
  src/string-stream.cc  src/string-stream.h  src/strtod.cc  src/strtod.h  src/stub-cache.cc
  src/stub-cache.h  src/token.cc  src/token.h  src/transitions-inl.h  src/transitions.cc
  src/transitions.h  src/type-info.cc  src/type-info.h  src/unbound-queue-inl.h  src/unbound-queue.h
  src/unicode-inl.h  src/unicode.cc  src/unicode.h  src/utils-inl.h  src/utils.cc
  src/utils.h  src/v8-counters.cc  src/v8-counters.h  src/v8.cc  src/v8.h
  src/v8checks.h  src/v8conversions.cc  src/v8conversions.h
  src/v8globals.h   src/v8memory.h  src/v8threads.cc  src/v8threads.h   src/v8utils.cc 
  src/v8utils.h  src/variables.cc  src/variables.h  src/version.cc  src/version.h
  src/vm-state-inl.h  src/vm-state.h  src/zone-inl.h  src/zone.cc  src/zone.h)

# appending 64 bit only sources.

LIST(APPEND V8_BASE_SRC 
  src/x64/assembler-x64-inl.h
  src/x64/assembler-x64.cc
  src/x64/assembler-x64.h
  src/x64/builtins-x64.cc
  src/x64/code-stubs-x64.cc
  src/x64/code-stubs-x64.h
  src/x64/codegen-x64.cc
  src/x64/codegen-x64.h
  src/x64/cpu-x64.cc
  src/x64/debug-x64.cc
  src/x64/deoptimizer-x64.cc
  src/x64/disasm-x64.cc
  src/x64/frames-x64.cc
  src/x64/frames-x64.h
  src/x64/full-codegen-x64.cc
  src/x64/ic-x64.cc
  src/x64/lithium-codegen-x64.cc
  src/x64/lithium-codegen-x64.h
  src/x64/lithium-gap-resolver-x64.cc
  src/x64/lithium-gap-resolver-x64.h
  src/x64/lithium-x64.cc
  src/x64/lithium-x64.h
  src/x64/macro-assembler-x64.cc
  src/x64/macro-assembler-x64.h
  src/x64/regexp-macro-assembler-x64.cc
  src/x64/regexp-macro-assembler-x64.h
  src/x64/stub-cache-x64.cc
)

if(APPLE)
LIST(APPEND V8_BASE_SRC 
  src/platform-macos.cc
  src/platform-posix.cc
)
endif(APPLE)

#if shared
#LIST(APPEND V8_BASE_SRC src/v8dll-main.cc)
#add_definitions(-DV8_SHARED -DBUILDING_V8_SHARED)
#end if shared

include_directories(src)

add_library(v8_base STATIC ${V8_BASE_SRC})

JS2C_GEN(
  JSFILES src/runtime.js src/v8natives.js src/array.js src/string.js src/uri.js src/math.js src/messages.js src/apinatives.js src/debug-debugger.js src/mirror-debugger.js src/liveedit-debugger.js src/date.js src/json.js src/regexp.js src/macros.py
  GTYPE CORE
  CPP_FILE libraries.cc
)
JS2C_GEN(
  JSFILES src/proxy.js src/collection.js src/object-observe.js src/macros.py
  GTYPE EXPERIMENTAL
  CPP_FILE experimental-libraries.cc
)


add_library(v8_nosnapshot
  src/snapshot-empty.cc
  ${CMAKE_CURRENT_BINARY_DIR}/libraries.cc
  ${CMAKE_CURRENT_BINARY_DIR}/experimental-libraries.cc
)


add_executable(mksnapshot src/mksnapshot.cc)
target_link_libraries(mksnapshot v8_nosnapshot v8_base )

get_target_property(MK_SNAPSHOT_EXEC mksnapshot LOCATION)

ADD_CUSTOM_COMMAND(
      OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/snapshot.cc"
      COMMAND "${MK_SNAPSHOT_EXEC}"
      ARGS "--log-snapshot-positions"
      ARGS "--logfile"
      ARGS "${CMAKE_CURRENT_BINARY_DIR}/snapshot.log"
      ARGS "${CMAKE_CURRENT_BINARY_DIR}/snapshot.cc"
#TODO:Extra stuff if ARM
      DEPENDS mksnapshot
      COMMENT "Running mksnapshot, generating: snapshot.cc with ${MK_SNAPSHOT_EXEC}"
      VERBATIM)


add_library(v8_snapshot
  ${CMAKE_CURRENT_BINARY_DIR}/snapshot.cc
  ${CMAKE_CURRENT_BINARY_DIR}/libraries.cc
  ${CMAKE_CURRENT_BINARY_DIR}/experimental-libraries.cc
)

set(PREPARSER_SRC)
list(APPEND PREPARSER_SRC
  include/v8-preparser.h  include/v8stdint.h  src/allocation.cc
  src/allocation.h  src/atomicops.h  src/atomicops_internals_x86_gcc.cc
  src/bignum.cc  src/bignum.h  src/bignum-dtoa.cc  src/bignum-dtoa.h
  src/cached-powers.cc  src/cached-powers.h  src/char-predicates-inl.h
  src/char-predicates.h  src/checks.h  src/conversions-inl.h
  src/conversions.cc  src/conversions.h  src/diy-fp.cc  src/diy-fp.h
  src/double.h  src/dtoa.cc  src/dtoa.h  src/fast-dtoa.cc
  src/fast-dtoa.h  src/fixed-dtoa.cc  src/fixed-dtoa.h  src/globals.h
  src/hashmap.h  src/list-inl.h  src/list.h  src/once.cc  src/once.h
  src/preparse-data-format.h  src/preparse-data.cc  src/preparse-data.h
  src/preparser.cc  src/preparser.h  src/preparser-api.cc
  src/scanner.cc  src/scanner.h  src/strtod.cc  src/strtod.h
  src/token.cc  src/token.h  src/unicode-inl.h  src/unicode.cc
  src/unicode.h  src/utils-inl.h  src/utils.cc  src/utils.h)

if(WIN32)
  list(APPEND PREPARSER_SRC src/win32-math.cc src/win32-math.h)
endif(WIN32)

add_library(preparser_lib ${PREPARSER_SRC})


add_library(v8 src/v8dll-main.cc)
target_link_libraries(v8 v8_base v8_snapshot)


#TODO: Test windows here to specify -D_CRT_SECURE_NO_WARNINGS
add_executable(v8_shell samples/shell.cc)
target_link_libraries(v8_shell v8)
