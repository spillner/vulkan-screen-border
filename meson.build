# Copyright © 2019 Intel Corporation
# SPDX-License-Identifier: MIT

screen_border_shaders = [
  'screen_border.frag',
  'screen_border.vert',
]
screen_border_spv = []
foreach s : ['screen_border.frag', 'screen_border.vert']
  _name = f'@s@.spv.h'
  screen_border_spv += custom_target(
    _name,
    input : s,
    output : _name,
    command : [
      prog_glslang, '-V', '-x', '-o', '@OUTPUT@', '@INPUT@', glslang_quiet,
      glslang_depfile,
    ],
    depfile : f'@_name@.d',
  )
endforeach

vklayer_files = files(
  'screen_border.cpp',
  'border_params.c',
)

vklayer_mesa_screen_border = shared_library(
  'VkLayer_MESA_screen_border',
  vklayer_files, screen_border_spv, sha1_h,
  c_args : [no_override_init_args],
  gnu_symbol_visibility : 'hidden',
  dependencies : [idep_vulkan_util, idep_mesautil, vulkan_wsi_deps, libimgui_core_dep, dep_dl],
  include_directories : [inc_include, inc_src],
  link_args : cc.get_supported_link_arguments(['-Wl,-Bsymbolic-functions', '-Wl,-z,relro']),
  install : true
)

install_data(
  files('VkLayer_MESA_screen_border.json'),
  install_dir : join_paths(get_option('datadir'), 'vulkan', 'explicit_layer.d'),
  install_tag : 'runtime',
)

install_data(
  'mesa-screen-border-control.py',
  install_dir : get_option('bindir'),
  install_mode : 'r-xr-xr-x',
)
