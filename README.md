# tensorflow-amd-git
This repository contains the PKGBUILD, patches, and test scripts for building the `tensorflow-amd-git` and `python-tensorflow-amd-git` [AUR packages](https://aur.archlinux.org/packages/tensorflow-amd-git) for Arch Linux.
Unlike the `tensorflow-rocm` series of packages, which pull from [Google's tensorflow/tensorflow repo](https://github.com/tensorflow/tensorflow/),
this package pulls directly from [AMD's upstream fork.](https://github.com/ROCmSoftwarePlatform/tensorflow-upstream/)
Specifically, as of 2023-08-18 it draws on the "r2.13-rocm-enhanced" branch on the commit of [the last successful build.](http://ml-ci.amd.com:21096/job/tensorflow/job/release-rocmfork-r213-rocm-enhanced/job/release-build-whl/lastSuccessfulBuild/)

## Build instructions
```sh
sudo pacman -S base-devel git
git clone https://aur.archlinux.org/tensorflow-amd-git
cd tensorflow-amd-git
makepkg --syncdeps
sudo pacman -U tensorflow*.zst python-tensorflow*.zst
```

## Building for other machines
This package uses `-march=native` to enable CPU specific optimizations. If you are building for other people, do:
```sh
sed "s/-march=native/-march=x86-64/" -i PKGBUILD
```

## Building for other graphics cards
I have read that this repo needs no configuration to detect which supported cards are installed and build for them.
If you are building for a graphics card not installed on your machine, such as if you need to build for gfx1030 but you have a gfx1031 card installed,
you must create the file `/opt/rocm/bin/target.lst` and add each gfx architecture you want to build for.
Something like:
```sh
echo -e "gfx900\ngfx904\ngfx906\ngfx908\ngfx90a\ngfx1030" | sudo tee /opt/rocm/bin/target.lst
```
As of 2023-07-05, I have confirmed that RX 580 (gfx803) is NOT supported.
I have confirmed that you can use RX 6750 XT (gfx1031) by setting `target.lst` before building:
```sh
echo -e "gfx1030" | sudo tee /opt/rocm/bin/target.lst
```
and by setting this environment variable before running:
```sh
export HSA_OVERRIDE_GFX_VERSION=10.3.0
```
You can find your gfx by running `/opt/rocm/bin/rocminfo | grep gfx` after installing `rocminfo`.

## Fixing this PKGBUILD when it inevitably breaks
To learn what the build environment is supposed to look like, see the official Dockerfile for building tensorflow: [tensorflow/tools/ci_build/Dockerfile.rocm in ROCmSoftwarePlatform/tensorflow-upstream](https://github.com/ROCmSoftwarePlatform/tensorflow-upstream/blob/develop-upstream/tensorflow/tools/ci_build/Dockerfile.rocm). That's where I found out about `target.lst` among other things; hopefully it helps.

If you are completely unable to build the repository, you may have luck with installing AMD's pre-built python wheels. pip seems to think tensorflow-rocm is only supported up to python 3.10, so you will have to download the wheel [directly from AMD's website](http://ml-ci.amd.com:21096/job/tensorflow/job/release-rocmfork-r213-rocm-enhanced/job/release-build-whl/lastSuccessfulBuild/) and install it: `pip install tensorflow_rocm-2.13*cp311-cp311-manylinux2014_x86_64.whl` When I tried it, it did seem to run on GPU and not CPU.

If AMD's website no longer hosts the python wheels, you may find an equivalent by following these breadcrumbs:
* [tensorflow/tensorflow](https://github.com/tensorflow/tensorflow/) ->
* [Community Supported TensorFlow Builds](https://github.com/tensorflow/build#community-supported-tensorflow-builds) ->
* [Linux AMD ROCm GPU Stable : TF 2.x 	Build Status 	Release 2.12](http://ml-ci.amd.com:21096/job/tensorflow/job/release-rocmfork-r212-rocm-enhanced/job/release-build-whl/lastSuccessfulBuild/)

Lastly, it appears the official way to use TensorFlow on ROCm is by a docker image. You may get random permission errors while downloading because it takes forever to download, and your system clock may desynchronize with the server. To fix the permissions errors, `sudo systemctl start systemd-timesyncd`. Otherwise, follow [the official instructions here](https://hub.docker.com/r/rocm/tensorflow) or [my longer tutorial here.](https://github.com/mpeschel10/test-tensorflow-rocm)

## Tensorflow 2.14
This PKGBUILD is not able to build Tensorflow 2.14 as a drop-in replacement. I suggest three changes to get you started; after that, you're on your own.

 1. Update the "last good commit" hash.
You can find a good commit hash from [AMD's Jenkins CI server.](http://ml-ci.amd.com:21096/job/tensorflow/job/nightly-rocmfork-develop-upstream/job/nightly-build-whl/lastSuccessfulBuild/)
Then update the PKBUILD like:

```sh
...
#_known_good_commit=c19cfffe476ec3338fb24ed3ce2baabfc558076e
 _known_good_commit=81b90075b3309e3c538915d212f0149daf9cd2a6  
...
```

 2. Change the branch of the git source link.
```sh
...
#source=('tensorflow-upstream-rocm::git+https://github.com/ROCmSoftwarePlatform/tensorflow-upstream#branch=r2.13-rocm-enhanced'
 source=('tensorflow-upstream-rocm::git+https://github.com/ROCmSoftwarePlatform/tensorflow-upstream#branch=develop-upstream'
...
```

 3. Add the `libxcrypt-compat` dependency.
```sh
...
#makedepends=('python-numpy' 'git' 'python-wheel' \
 makedepends=('python-numpy' 'git' 'python-wheel' 'libxcrypt-compat'\
...
```
Then
```sh
pacman -S libxcrypt-compat
```

Good luck.

### Pull requests welcome.
