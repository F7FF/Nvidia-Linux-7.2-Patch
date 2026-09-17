# Nvidia-Linux-7.2-Patch
### A patch that allows pre-610 Nvidia drivers to hopefully work on Linux 7.2, especially NixOS
---
Linux 7.2 removes `strncpy`, which the pre-610 Nvidia driver uses in a couple of places, which causes the driver to fail to build for 7.2 (especially on NixOS, which doesn't seem to support 610 yet). [This patch made some subsitutions](https://github.com/NVIDIA/open-gpu-kernel-modules/issues/1224), but it fails to compile on <610, so I did essentially the same thing except pulling changes from the actual *release* 610 wherever I could.

7.2 also renamed `drm_atomic_state` to `drm_atomic_commit`, and the aforementioned patch made a couple of clever `#DEFINE`s to work around that.

I'm not really "good" with Nix yet, but I configured it like this:

```nix
hardware.nvidia = {
    # package = config.boot.kernelPackages.nvidiaPackages.stable; # Usually, this would work, but not here

    package = config.boot.kernelPackages.nvidiaPackages.stable // {
      open = config.boot.kernelPackages.nvidiaPackages.stable.open.overrideAttrs (oldAttrs: { # have to patch the open module, otherwise nothing happens
        patches = (oldAttrs.patches or []) ++ [
          ./f7ff_7_2_kernel_nvidia.patch
        ];
      });
    };
  };
```

Best of luck, the moment 610 is available I'm jumping to that.
