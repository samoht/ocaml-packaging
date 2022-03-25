# OCaml packaging

# Tier-1 Platforms

## macOS

This is the most popular developer platform, and there are two widely used package managers:

- [Homebrew](https://brew.sh) 
- [MacPorts](https://www.macports.org)

Both of them accept PRs for new packages, and ocaml and opam are both packaged up there and reasonably regularly updated.

To create a binary mpkg installer for osx, which lets the binaries be signed and distributable through the App Store.  This is an attractive option if we have a teaching environment (GUI) in the future. The [Packages](http://s.sudre.free.fr/Software/Packages/about.html) tool is a good way to create a Mac GUI.

Continuous integration is unfortunately very difficult on macOS. `ocaml-ci` can be extended with macOS builders.

## Linux Distributions

While source-based installation on Linux is largely a similar experience, binary distribution mechanism wary wildly.  The [opam depext](https://github.com/ocaml/opam-depext/tree/2.0) covers many distributions, but here we focus on the ones we plan to test thoroughly.

We have excellent Continuous Integration support for all these Linux distributions via `ocaml-ci` and `opam-repo-ci`. We do not believe it necessary to test Linux kernel variants as nothing depends on this yet (this may change as we investigate Meltdown and Spectre mitigation technologies).

### Debian

Apt-based distribution. Need to set up a server to [regularly build binary packages](https://www.debian.org/doc/manuals/maint-guide/build.en.html) for a development remote. Due to the long release cycles in [Debian Stable](https://wiki.debian.org/DebianStable) we do need to publish a binary remote for users to subscribe to the latest packages. 

### Ubuntu

Apt-based distribution with a fast six month release cycles, but Ubuntu also has a [PPA](https://launchpad.net/ubuntu/+ppas) mechanism for publishing custom binary remotes that they autobuild. We need to support Ubuntu LTS (currently 16.04) as well as the latest release (currently Ubuntu 21.10). The latest Ubuntu releases tend to incorporate interesting new technologies like a new gcc or binutils, so they highlight problems that will show up elsewhere quite fast.

### Fedora

RPM based distribution.  opam1 historically has not worked well there due to a lack of aspcud, but opam2 should work out of the box with its builtin solver.  Release of Fedora are fairly frequent, and [Richard Jones](http://people.redhat.com/rjones/) is a good contact there who is a long-time contributor to OCaml packaging.

More recently, opam2-beta6 has been [pushed to the Fedora27 testing repository](https://bugzilla.redhat.com/show_bug.cgi?id=1501992) and so opam2 should be available in future Fedora releases.

### CentOS

CentOS is the long term release of Fedora, and the free version of RedHat Enterprise Linux.  CentOS 6 is still used in some places, but is now far too old to support (in particular, the version of `git` is ancient and doesnt work well with opam). 

Therefore CentOS 7.x is likely the only release to support.

### Alpine Linux

This is the most popular container distribution of Linux, and generates really minimal images for use with Docker.  It is also a sensibly secure distribution and has PIE binaries by default (which makes it show up some bugs that don't occur on other Linux distros).

## Windows

Windows presents a specific challenge as Microsoft officially support concurrent major releases for a considerable period of time. Additionally, there are 4 modes of operation for OCaml: running within Cygwin (which is a BSD-like environment); running within Windows Services for Linux (WSL, which provides binary compatibility for 64-bit versions of several Linux distributions); running natively based on the Mingw-w64 toolchain (which, despite the name, provides both 32 and 64-bit compilers); and running natively based on Microsoft's C compiler (MSVC, which provides additional challenges, as it is also supported in multiple concurrent versions).

Tier-1 support will consist of the latest version of Windows 10 in the Semi-Annual release channel (current Windows 10 1709) for amd64, using both the Mingw-w64 and MSVC compiler toolchains at their latest released versions, and with Cygwin as the underlying Unix toolchain.

@dra27 Decision on whether to include Windows 10 LTSB (and how many versions) as Tier-1?

# Tier-2 Platforms

These are platforms for which CI will likely lag unless we get active contributors to maintain them.

## *BSD

The best way to get into the BSDs is to contribute a package to their respective ports trees. Each of them already have an ocaml and opam package.

### FreeBSD

FreeBSD has a [source-based ports](https://www.freebsd.org/ports/) tree as well a binary [pkg](https://www.freebsd.org/doc/handbook/pkgng-intro.html) installer for snapshots compiled from ports.  @hannes is an OCaml+FreeBSD contributor who may be able to offer advice.
 
### OpenBSD

OpenBSD has a small set of OCaml and opam ports ports, and some active users who develop Coq using it. 

### NetBSD

TODO investigate status.

## Windows (other versions)

@dra27 to expand further. Windows 7 (Server 2008 R2), 8 (Server 2012) and 8.1 (Server 2012 R2) supported and also x86 versions of the compiler. Also support for WSL as a Unix backend.

## Other CPU Architectures 

As we now have more active architectures, the OCaml Docker images for opam2 are going multiarch, initially with arm64 and x86.  Not all of the distributions above support non-x86, so here are some notes on where we are.

### x86

The 64-bit x86_64 will be supported for all distributions.  There have been some requests for 32-bit builds, but this seems like a low priority.

### ARM

There have been a number of requests for both ARM32 (rPi 1/2) and ARM64 (rPi3) distributions of OCaml, particularly since it has an excellent native-code generator for both.  Thanks to support from [Packet.net workonarm](https://github.com/WorksOnArm/cluster/issues/5) we have two machines that can do continuous builds.  This is currently 64-bit only, but we are working with them to get access to ones that can do 32-bit as well.

### PPC64

IBM is loaning two POWER8 machines to Tarides to use as build machines.  These will be used to generate best-effort multiarch images for this platform but it will remain Tier 2 until it stabilises.

# Mechanising This Information

There are a few libraries that code up this information for use by tools:

- [ocaml-version](http://anil-code.recoil.org/ocaml-version/ocaml-version/Ocaml_version/index.html) keeps track of OCaml releases, and also which distributions are supported on which architecture.
- [ocaml-dockerfile](http://anil-code.recoil.org/ocaml-dockerfile/) has code to generate logic for all the Linux distributions, and in particular the [Dockerfile_distro](http://anil-code.recoil.org/ocaml-dockerfile/dockerfile-opam/Dockerfile_distro/index.html) module lists all the information above in an OCaml-accessible format.

