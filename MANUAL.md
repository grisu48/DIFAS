# Manual

These scripts ease the environment setup procedure required to build your awesome project.
Each file located within the `packages` directory contains meta data and build instructions for a specific package as well as its dependencies.

The `installer` can be used to install packages, dependencies will be resolved automatically if `--with-depends` is provided.

    $ ./installer gmp mpfr mpc gcc

or

    $ ./installer --with-depends gcc

Note that all dependencies will be installed regardless of whether they are already present on your system, outside of this dependency installer.

The default `DIFAS_PREFIX` is set in `config.sh` and can be overwritten via an environment variable:

    $ DIFAS_PREFIX=/opt/custom-libs ./installer --with-depends gcc

## Package File structure

The package file itself needs to define the following variables:

- `NAME`
- `VERSION`
- `PACKAGE` - typically `$NAME-$VERSION`
- `FILE` - name of the file containing the source code
- `URL` - will be used for downloading the source code
- `SHA256SUM` - (optional) checksum of the file containing the source code

Furthermore, one package can depend on other packages.
These need to be specified in `DEPENDS`, for instance:

    DEPENDS="gmp mpc mpfr"

The build instructions will then follow.
[`internal/default_implementation.sh`](internal/default_implementation.sh) defines the default build steps.
Each step can be overwritten in the corresponding package file.

Have a look at the provided package files, they should give you an idea on how to handle things.

## Third Party Symlinker

All packages installed by DIFAS are separated by their name and version, this allows one to have multiple versions of a package installed side-by-side.
Yet, in order for a project to find its required dependencies, another layer of indirection is added.

A folder `third_party` should be located inside your project's build directory, containing symlinks for each dependency.
The symlinks point to installed packages in `DIFAS_PREFIX`.
The `third_party` folder should not contain different versions of the same package.
Example:

    $ ls -l "$HOME/.difas"
    drwxr-xr-x.  4 alex   dps 4.0K Nov 18 15:14 autoconf-2.68/
    drwxr-xr-x.  4 alex   dps 4.0K Nov 18 15:14 automake-1.15/
    drwxr-xr-x.  7 alex   dps 4.0K Nov 18 15:14 binutils-2.27/
    drwxr-xr-x.  5 alex   dps 4.0K Nov 18 15:14 bison-3.0.4/
    drwxr-xr-x.  4 alex   dps 4.0K Nov 18 15:14 boost-1.50.0/
    drwxr-xr-x.  4 alex   dps 4.0K Nov 18 15:14 boost-1.59.0/
    drwxr-xr-x.  5 alex   dps 4.0K Nov 18 15:14 cmake-3.2.1/
    drwxr-xr-x.  5 alex   dps 4.0K Nov 18 15:14 cmake-3.6.1/
        ...

    $ ls -l "$MY_AWESOME_PROJECT/build/third_party"
    lrwxrwxrwx. 1 alex dps   47 Nov 22 13:17 autoconf -> /home/alex/.difas/autoconf-2.68/
    lrwxrwxrwx. 1 alex dps   47 Nov 22 13:17 automake -> /home/alex/.difas/automake-1.15/
    lrwxrwxrwx. 1 alex dps   47 Nov 22 13:17 binutils -> /home/alex/.difas/binutils-2.27/
    lrwxrwxrwx. 1 alex dps   45 Nov 22 13:17 bison -> /home/alex/.difas/bison-3.0.4/
    lrwxrwxrwx. 1 alex dps   46 Nov 22 13:17 boost -> /home/alex/.difas/boost-1.59.0/
    lrwxrwxrwx. 1 alex dps   45 Nov 22 13:17 cmake -> /home/alex/.difas/cmake-3.2.1/
        ...

The `third_party_linker` creates these symlinks for you automatically.
It uses the same version of a package as defined in the related package file.
The folder `third_party` is created in the current working directory when invoking `third_party_linker`.

## Using an Installed Packages

To use one of the installed packages directly, adjust the necessary environment variables accordingly:

- `PATH`
- `CPATH`
- `LIBRARY_PATH`
- `LD_LIBRARY_PATH`

## CMake Integration

Add this to the top of your main CMake file:

```cmake
set(THIRD_PARTY_DIR ${CMAKE_BINARY_DIR}/third_party CACHE STRING "Third Party Directory")
if(NOT EXISTS ${THIRD_PARTY_DIR})
	message(FATAL_ERROR "`third_party` directory does not exist, did you run `third_party_linker` ?")
endif()
file(GLOB _prefix_paths ${THIRD_PARTY_DIR}/*)
list(APPEND CMAKE_PREFIX_PATH ${_prefix_paths})
```

Invoke the `third_party_linker` in your build directory before invoking CMake.
The aforementioned code snippet will add each folder (and symlink) located in the `third_party` directory to the `CMAKE_PREFIX_PATH` list.
This list is considered for `find_package` calls.
With this you should not have to adjust anything given you are using `find_package` correctly.

## Patches

The default `pkg_prepare` action will apply all patches inside the `patches` directory which are prefixed with the package name.
The order is inferred from the filename which should have the following structure:

    <package name>-<patch number>-<comment>.patch

## Using a newer GCC

First, install GCC as usual:

    $ ./installer --with-depends gcc

Now set the `DIFAS_USE_GCC` environment variable to some value for subsequent installer invocations.
The installer will now use the freshly installed GCC as compiler.
For instance:

    $ DIFAS_USE_GCC=Y ./installer boost
