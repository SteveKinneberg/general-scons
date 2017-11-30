General SCons Base Files
========================

This repository provides a set of SCons files that work together to provide a
more useful SCons experience for projects that target multiple platforms while
providing a clean way to specify platform options.

Features
--------

* Support native and cross compilation of primary deliverable.
* Support for building host tools whose source is included with the overall
  project and are used in the generation of primary deliverable.
* Support for building Doxygen based documentation.
* Support for automated unit tests using Google Mock and Google Test.
* Support for code coverage reporting.
* Support for cross-compilation using Yocto generated SDKs.
* Support for building for Android using Android NDKs and SDKs without having to
  maintain a duplicate build specification via Android.mk files used by ndk-build.


Dependencies
------------

* SCons and, by extension, Python are the only hard requirements.
* Doxygen will need to be installed for documentation generation.  (Note: The
  SCons files only check whether Doxygen itself is installed or not.  It does
  not check for tools that Doxygen may depend on such as 'dot' from GraphViz or
  'mscgen'.)
* Google Test will need to be installed to support automated testing.
* Google Mock will need to be installed if you wish to use a C++ code mocking
  framework.
* LCov and GenHtml will need to be installed if you wish to enable code coverage
  reporting when building with GCC.
* An SDK generated from Yocto will need to be installed to enable
  cross-compilation using said SDK.
* The Android NDK and SDK will need to be installed to enable building for Android.


Usage
-----

### Installation

Copy the following files and directories to where you will be developing your
project:

  * SConstruct
  * SConsFiles/
  * site_scons/

Add the following to your version control's ignore file (i.e., .gitignore for
git):

    build/
    dist/
    *.pyc
    .scon*
    config.log

Or, just use the .gitignore file from this project.

### Running

Just run `scons` to get the default build.

The SCons files in this project provide the following command line variables
that can be passed as arguments to scons:

  * `V`: Build output verbosity.  Defaults to False/no/off/0.  Controls whether a
    short readable line is output for each build step or whether the complete
    command line is output.  Typically, developers will prefer the shorter,
    cleaner output unless they are actually modifying the command line options
    themselves.
    
  * `PRODUCTION`: Select between production and non-production builds.  Defaults
    to False/no/off/0.  Non-production builds may have debug symbols enabled,
    code coverage enabled, and paths to locally built libraries embedded into
    the executables.  Production builds always disable debug symbols, code
    coverage hooks, and never embed paths to locally built libraries.
    
  * `NDEBUG`: Suppress debug build information.  Defaults to False/no/off/0.  When
    enabled, it will define NDEBUG on the command line to compilers, not build
    debug symbols, and build with optimization enabled.
    
  * `TP`: The target platform.  Defaults to 'host".  It can be set to any of the
    defined target platforms.  (Use `scons -h` to get a list available targets
    along with other information.)
    
  * `COVERAGE`: Enable code coverage reporting.  Defaults to True/yes/on/1.  This
    controls whether code coverage hooks are added and the reporting of code
    coverage results.
    
  * `GTEST_DIR`: The path to the Google Test source code.  This must be specified
    if Google Test is not installed in a system directory or you wish to
    override the version of Google Test being used.

  * `GMOCK_DIR`: The path to the Google Mock source code.  This must be specified
    if Google Test is not installed in a system directory or you wish to
    override the version of Google Mock being used.

  * `TEST_V`: Unit test output verbosity.  Defaults to `on_failure`.  This
    controls whether the Google Test reports are printed to the screen or not.
    The options are:

      * `always`: Always output the Google Test results.
      * `on_failure`: Only output the Google Test results if there was a
        failure.
      * `never`: Never output the Google Test results.

  * GTEST_FILTER: Specify which Google Test to run.  Default is unset to run all
    tests.  Note, that if there are multiple Google Test programs all will be
    run with the specified filter.  (This is primarily an aid to developers
    working on code that affects a given test case by allowing the developer to
    build and run the specific test cast in one step.)


### Build output

Intermediary build artifacts are built under the build/ directory.  Host and
target build intermediaries are further separated into appropriate
subdirectories under build/.  The directory structure under the
build/<host/target>/ directory follows that of the source directory.

The SCons files in this project uses SCons' default behavior of "copying" the
source files to the build directory before building.  On systems that support
hard links, like Linux and BSD, this is just a hard link rather than a copy
which is both faster and more space efficient than copying.  The reason for
choosing this approach is that it simplifies supporting generated C/C++ files
(from tools like lex/flex and yacc/bison) while keeping those generated files in
the build directory tree.

Final build artifacts are placed under the dist/ directory.  Like the build/
directory, host and target builds are placed into appropriate subdirectories.
Below that, the final build artifacts are separated as follows:

  * bin/ -- Executables
  * include/ -- Externally deliverable library header files
  * lib/ -- Externally deliverable libraries
  * docs/ -- Location of generated documentation
  * coverage/ -- Location of code coverage results



Design
------

The overall design is highly modular to make it easy to add new target and host
platforms.  The common components have been also been modularized to improve
maintainability and extensibility.

### Build environments

There are 4 primary build environments defined, each with their own specific
purpose:

  * project: This sets up a few base build variables and is the environment
    where the documentation will be built from.
  * host: This provides the build environment for host-side tools that may need
    to be built as part of the project.  It inherits all the settings from the
    project environment.  It will set up compilers and compiler flags for the
    build host.  (Don't use this environment for the project's real deliverable.)
  * target: This provides the build environment for the project's real
    deliverable.  If the target platform the real deliverable is being built for
    happens to be the host platform then the target environment will actually be
    the host build environment, just with a different name.  This allow for code
    to be built for the real deliverable to be built in the "target" environment
    without the need for sprinkling build type checks throughout all the SCons
    files.  If the target is different than the host, then the compiler and
    compiler options will be setup for the target environment.
  * target_test: This environment adds Google Mock and Google Test to the build
    and sets up for code coverage reporting.  If the target environment is the
    same as the host environment, then the automated tests will be run as part
    of the build.



### `SConstruct`

All SCons projects must have a SConstruct file.  For the most part, this file
calls into the other SConscript files to create the various build environments.

The order in which the SConscript files for the project environment, host
environment, target environment, and target test environment are called is
important because each environment builds upon the previous environments.  If
desired, would be fairly easy to add a host test environment.  It would need to
be created after the host environment.  It would also need a block to actually
run the tests near where the target tests are run.


### `site_scons/site_tools/*`

This directory contains additional builders created by other developers.  They
are copyrighted by their respective developers and licensed independently from
this rest of this project.  Note that the versions included in this repository
have been modified.  The modifications mostly center around generating clean
build reporting.  See [3rd Party
Readme](site_scons/site_tools/3rdPartyReadme.md) for information on copyright
ownership, licensing, and original project URL.


### `SConsFiles/SConscript.custom_tests`

This defines a number of functions that can be use for build-time checks for
things like support for compiler options, C++ symbol definition, pkg-config
results.


### `SConsFiles/SConscript.gtool`

This integrates Google Test and Google Mock into the build.  It will check for
pre-built and/or source versions installed by Ubuntu or Fedora packages, or
optionally, build from source specified on the scons command line.


### `SConsFiles/SConscript.host_env`

This creates the host build environment for building host-side tools that will
then be available for building the rest of the project in the target build
environment.


### `SConsFiles/SConscript.project_env`

This creates the base-line environment that defines project-wide build variables and
builds the documentation.


### `SConsFiles/SConscript.target_env`

This creates a target build environment for building the final deliverable.  It
can make use of tools built in the host environment.


### `SConsFiles/SConscript.test_env`

This creates a test environment for building automated unit test programs and
running them.


### `SConsFiles/coverage/SConscript.coverage`

This primarily serves as a platform selector for code coverage hooks build
options since these will by necessity be platform specific.


### `SConsFiles/coverage/SConscript.coverage.posix`

This checks for tool availability and code coverage compiler option support
using GCC options.


### `SConsFiles/platforms/SConscript.platform.android`

This sets up the build variables necessary for building code that targets
Android.  It will actually run a ndk-build on a special dummy Android.mk file to
get the compiler and its options.  It adds the following SCons command line
variables:

  * `ANDROID_CPU`: Android target processor.
  * `ANDROID_NDK_PATH`: Path to the base directory of an Android NDK installation.
  * `ANDROID_SDK_PATH`: Path to the base directory of an Android SDK installation.
  * `ANDROID_STL`: Which Android C++ STL implementation to use.


### `SConsFiles/platforms/SConscript.platform.host`

This is mostly just a "no-op" file that exists to present "host" as a build
target platform.  It avoids awkward conditional statements sprinkled throughout
the SCons files.


### `SConsFiles/platforms/SConscript.platform.yocto`

This sets up an environment to use tools and libraries from an SDK generated by
the Yocto project.  It add the following SCons command line variable:

  * `YOCTO_SDK_PATH`: Path to an installed Yocto SDK.
  
When building for a Yocto platform, `YOCTO_SDK_PATH` must be specified and it
must be able to run the environment-setup script.


### `SConsFiles/platforms/support/Android.mk`

This is a dummy Android.mk file used by the Android platform to extract the
various build variables necessary for building for an Android target.


### `SConsFiles/tool_options/SConscript.tool_options`

This primarily serves as a platform selector for tool build options since these
will by necessity be platform specific.


### `SConsFiles/tool_options/SConscript.tool_options.posix`

This sets up the options passed to C/C++ compilers for POSIX platforms.  The
options currently specified will (mostly) work with both GCC and LLVM/Clang.
Build-time checks are done to ensure that the compiler does support each option.

This set of options can be altered to suite the requirements of a given
project.  The set of options provided by default were chosen to encourage the
development of clean and portable code.  Some options, like '-Wshadow', may be
annoying to some developers but can be quite helpful in preventing difficult to
see bugs.



Notes
-----

1. With the exception of the code found under the `site_scons/` directory, this
   project is licensed under the terms of the [Unlicense](http://unlicense.org).

2. The master branch will always remain stable.  Development branches, however,
   may experience changes in history (i.e., `git rebase -i`).

3. The default options in
   `SConsFiles/tool_options/SConscript.tool_options.posix` enable a fair number
   of additional compiler checks including pedantic checks.  These can be
   modified to suit a given project, but are recommended to help ensure quality,
   portable code.  It is probably best to think of this as "free" static code
   analysis since there is no need to install and use complicated tools.  This,
   however, should *not* be considered a replacement for more advanced static
   code analysis tools.


To-Do
----
* Add Microsoft compiler options file.
* Add code coverage for Microsoft builds (if available).
* Add support for LLVM/Clang.
