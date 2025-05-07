Mesa plugin to superimpose solid-color borders over the Vulkan drawing surface.  This is useful to support tracking via Sinden light guns and probably other purposes.  Code is derived from the Vulkan 'overlay' module shipped with Mesa.

Building
=======

Since the module relies on some Mesa internal utilities and configuration info, the easiest way to build is to download the Mesa source tree (e.g. from Git, or from another recent source package, copy this directory into the larger Mesa source tree as src/vulkan/screen-border, and apply the included patch to update Mesa's meson build files (`patch -p1 < src/vulkan/screen-border/mesa-build.patch` from the top-level Mesa directory).

After configuring the build system, you can build the screen-border module by including :code:`screen-border` as part of the :code:`vulkan-layers` argument, e.g.

.. code-block:: sh

  meson -Dvulkan-layers=device-select,screen-border build/
  ninja -C build/

If you're on a 64-bit system and want to use this module with 32-bit software (including some Steam games and older versions of Wine/Proton), you'll probably want to build both 32- and 64-bit versions of the module.  On my system (Arch Linux), the Rust toolchain's 32-bit support is not as robust as 64-bit, and struggles with 32-bit versions of the Mesa bindings, so I had to configure my 32-bit build without the Rust components of Gallium:

.. code-block:: sh

  meson setup build32 \
        -Dandroid-libbacktrace=disabled \
        -Dgallium-drivers=softpipe,llvmpipe \
        -Dgallium-extra-hud=true \
        -Dgallium-nine=false \
        -Dgallium-va=disabled \
        -Dgallium-vdpau=disabled \
        -Dgallium-xa=disabled \
        -Dplatforms=x11,wayland \
        -Dvulkan-drivers=amd,intel,intel_hasvk,swrast \
        -Dvulkan-layers=device-select,overlay,screen-border \
        -Dtools=[] \
        -Dbuildtype=plain \
        --cross-file=/usr/share/meson/cross/lib32 \
        -Dsysconfdir=/etc \
        -Dlibdir=lib32 \
        -Dlegacy-x11=dri2

See `docs/install.rst <https://gitlab.freedesktop.org/mesa/mesa/-/blob/master/docs/install.rst>`__ for more information on configuring, building, and installing Mesa.

If you prefer to build this module outside of the Mesa source tree, you simply need to ensure that your build process can find all of the necessary headers (all from `mesa/include`, `mesa/src/util`, and `mesa/src/vulkan/util`), e.g. by symlinks or -I options, and that you set up a few platform-specific configuration options that are normally deduced in Mesa's meson setup process (see the top portion of screen_borders.cpp for a manual list of `#define`s).


Installation
=======

Even if you build screen-borders within the Mesa source tree, you do not need to overwrite or replace your existing Mesa installation (e.g. from a package provided by your distribution) to use the module.  Simply copy `libVkLayer_MESA_screen_border.so` (from the build output directory) to a directory where your runtime linker can find it (e.g. `/usr/lib`, `/usr/lib64`, and/or `/usr/lib32`, or a location in `LD_LIBRARY_PATH`, or a path you specify in `LD_PRELOAD`), and the `VkLayer_MESA_screen_border.json` file from this source directory into your system's `explicit_layer.d/` collection.  Note that 32-bit and 64-bit versions of `libVkLayer_MESA_screen_border.so` will need to be installed into separate directories.  The location of the `explicit_layer.d/` directory will vary from system to system; for Arch Linux it's under `/usr/share`, but other distributions may use `/etc` or `/usr/etc` and a custom installation might use `/usr/local/share` or `/usr/local/etc`.  You can find the right directories for your system by running `locate explicit_layer.d` and `locate libVkLayer_khronos_validation.so` (to find out where your system Mesa installed the default Vulkan plugins).

If you built in-tree and plan to just fully replace your system Mesa (i.e. possibly overwriting the standard files provided by your distribution), then `sudo ninja -C builddir/ install` should put everything (including the screen-border module) in the right place.

If you plan to use the module with Steam games, you'll need to make sure that both `libVkLayer_MESA_screen_border.so` and `VkLayer_MESA_screen_border.json` are in places where the Steam runtime can find them.  In *theory* Steam and Proton should be configured to look for Vulkan modules in the standard system directories, but versions of the Steam runtime have shipped where that isn't necessarily the case.  The `locate` commands above will help you find other directories (within your Steam installation) where you might need to copy the files if your Steam setup doesn't seem to be finding them in the system directories.


Basic Usage
=======

The easiest way to activate the layer is via the `VK_LOADER_LAYERS_ENABLE` or `VK_INSTANCE_LAYERS` environment variable, e.g. by running programs as

.. code-block:: sh

  VK_LOADER_LAYERS_ENABLE=VK_LAYER_MESA_screen_border /path/to/my_vulkan_app


for example, `VK_LOADER_LAYERS_ENABLE=VK_LAYER_MESA_screen_border vkgears -fullscreen`

You can run

.. code-block:: sh

  export VK_LOADER_LAYERS_ENABLE=VK_LAYER_MESA_screen_border


to load the module for all programs subsequently invoked by that shell.

The module has several runtime-adjustable parameters that can be controlled by the `VK_LAYER_MESA_SCREEN_BORDER_CONFIG` environment variable, e.g.

.. code-block:: sh

  VK_LOADER_LAYERS_ENABLE=VK_LAYER_MESA_screen_border VK_LAYER_MESA_SCREEN_BORDER_CONFIG=color=orange,thickness=20 /path/to/my_vulkan_app


  By default, the border will be white with a thickness of 10 pixels.  See `border_params.c` for more details on the options and syntax.

For Steam games (including Windows games run under Proton), you can insert the environment variable assignments above as part of the program's launch options, e.g.

.. code-block:: sh

  VK_LAYER_MESA_SCREEN_BORDER_CONFIG=thickness=16 VK_INSTANCE_LAYERS=VK_LAYER_MESA_screen_border %command%


The screen-borders module can be applied to Windows games using DirectX via Proton's DXVK backend, which translates DirectX calls to Vulkan.  Recent versions of Proton enable DXVK by default on supported systems.  For OpenGL games (whether native or running under Proton/WINE), you can use Mesa's Zink backend to translate OpenGL calls to Vulkan:

.. code-block:: sh

  VK_LAYER_MESA_SCREEN_BORDER_CONFIG=thickness=16 MESA_LOADER_DRIVER_OVERRIDE=zink VK_INSTANCE_LAYERS=VK_LAYER_MESA_screen_border %command%


.. _docs/install.rst: ../../docs/install.rst
