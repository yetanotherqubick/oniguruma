set(ONIG_SAMPLE_PROGRAMS
  encode
  listcap
  names
  posix
  simple
  sql
  syntax
)

foreach(program IN LISTS ONIG_SAMPLE_PROGRAMS)
  add_executable(${program} "${program}.c")
  target_link_libraries(${program} PRIVATE onig)
endforeach()

add_custom_target(onig-samples-test
  COMMAND $<TARGET_FILE:encode>
  COMMAND $<TARGET_FILE:listcap>
  COMMAND $<TARGET_FILE:names>
  COMMAND $<TARGET_FILE:posix>
  COMMAND $<TARGET_FILE:simple>
  COMMAND $<TARGET_FILE:sql>
  COMMAND $<TARGET_FILE:syntax>
  DEPENDS ${ONIG_SAMPLE_PROGRAMS}
  WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}"
  COMMENT "Run Oniguruma sample programs"
)
