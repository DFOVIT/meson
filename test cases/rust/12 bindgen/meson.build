# SPDX-license-identifer: Apache-2.0
# Copyright © 2021-2022 Intel Corporation

project('rustmod bindgen', ['c', 'rust'], meson_version : '>= 0.63')

prog_bindgen = find_program('bindgen', required : false)
if not prog_bindgen.found()
  error('MESON_SKIP_TEST bindgen not found')
endif

add_project_arguments('-DPROJECT_ARG', language : 'c')
add_global_arguments('-DGLOBAL_ARG', language : 'c')

# This seems to happen on windows when libclang.dll is not in path or is not
# valid. We must try to process a header file for this to work.
#
# See https://github.com/rust-lang/rust-bindgen/issues/1797
result = run_command(prog_bindgen, 'include/other.h', check: false)
if result.returncode() != 0
  error('MESON_SKIP_TEST bindgen does not seem to work')
endif

# This is to test the include_directories argument to bindgen
inc = include_directories('include')

c_lib = static_library('clib', 'src/source.c', include_directories : inc)

rust = import('unstable-rust')

gen = rust.bindgen(
  input : 'src/header.h',
  output : 'header.rs',
  include_directories : 'include',
)

# see: https://github.com/mesonbuild/meson/issues/8160
f = configure_file(
  input : 'src/main.rs',
  output : 'main.rs',
  copy : true,
)

rust_bin = executable(
  'rust_bin',
  [f, gen],
  link_with : c_lib,
)

test('main', rust_bin)

# Test a generated header
gen_h = custom_target(
  'gen.h',
  command : [find_program('src/gen_header.py'), '@INPUT@', '@OUTPUT@'],
  output : 'gen.h',
  input : 'src/header.h'
)

gen2_h = custom_target(
  'other.h',
  command : [find_program('src/gen_header.py'), '@INPUT@', '@OUTPUT@'],
  output : 'other.h',
  input : 'include/other.h'
)

gen_rs = rust.bindgen(
  input : [gen_h, gen2_h],
  output : 'gen.rs',
)

f = configure_file(
  input : 'src/main2.rs',
  output : 'main2.rs',
  copy : true,
)

rust_bin2 = executable(
  'rust_bin2',
  [f, gen_rs],
  link_with : c_lib,
)

test('generated header', rust_bin2)

subdir('sub')
subdir('dependencies')

gp = rust.bindgen(
  input : 'src/global-project.h',
  output : 'global-project.rs',
)

gp_lib = static_library('gp_lib', 'src/global.c')

gp_exe = executable(
  'gp_exe',
  structured_sources(['src/global.rs', gp]),
  link_with : gp_lib,
)

test('global and project arguments', gp_exe)
